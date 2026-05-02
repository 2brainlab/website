---
layout: post
title: “Anonimizzazione reversibile del PII: un’alternativa ai modelli on-premise”
date: 2026-05-01
author: “Federico Cerruto”
tags:
 - “Privacy”
 - “PII”
 - “AI Engineering”
 - “Compliance”
 - “Agenti AI”
excerpt: “Se non puoi far girare modelli in locale ma devi proteggere i dati dei tuoi utenti, l’anonimizzazione reversibile ti permette di usare LLM cloud senza esporre il PII. Un PoC concreto, con codice.”
---

## TL;DR

Se non puoi far girare modelli on-premise ma hai bisogno di proteggere i dati dei tuoi clienti, l’anonimizzazione reversibile ti permette di usare LLM cloud senza mai trasmettere nomi, email o numeri di telefono reali. Il modello lavora su placeholder tipizzati. Il risultato viene ripristinato lato client. Funziona con qualsiasi API. Codice: [github.com/iron87/anonymizer-poc](http://github.com/iron87/anonymizer-poc)

## Il problema

Eseguire modelli in locale risolve il problema della privacy, ma ne apre altri: costi infrastrutturali, manutenzione dei modelli, latenza, scelta limitata dei modelli disponibili. Per molti team non è un trade-off accettabile.

Ma inviare dati utente a provider LLM cloud è un problema di compliance. Ogni query che contiene nomi, email, numeri di telefono finisce nei log del provider, nelle audit trail, potenzialmente nei dataset di addestramento.

La risposta classica è “esegui il modello in locale.” E se non puoi?

## Quando ha senso

Questo pattern funziona quando:

- stai elaborando dati di clienti o utenti finali (ticket di supporto, documenti medici, contratti);
- non puoi eseguire modelli on-premise per costi, complessità o capacità del team;
- hai bisogno di anonimizzazione reversibile, non di redazione statica.

Alcuni casi d’uso concreti:

- **analisi documentale** — riassumere contratti che contengono nomi e indirizzi di clienti;
- **gestione del calendario** — parsare richieste di meeting con email dei partecipanti;
- **automazione del supporto** — generare risposte a ticket che contengono numeri di telefono;
- **elaborazione di dati medici** — lavorare su record clinici senza esporli al provider.

Non ha senso invece per:

- dati interni all’azienda (codice, documenti strategici, email interne) — usa direttamente un modello locale;
- anonimizzazione unidirezionale (log di audit, report di compliance) — la redazione statica è sufficiente;
- scenari real-time con requisiti di latenza stretti — il lookup sul vault aggiunge overhead.

## Come funziona

Il flusso è lineare:

```
Prompt utente (con PII)
        │
        ▼
  [ Anonimizza ]   ──→  mario.rossi@example.com → [REDACTED_EMAIL_ADDRESS_1]
        │
        ▼
  [ Cloud LLM ]   ──→  lavora solo su placeholder
        │
        ▼
  [ Deanonimizza ] ──→  [REDACTED_EMAIL_ADDRESS_1] → mario.rossi@example.com
        │
        ▼
Risposta finale (PII ripristinato)
```

Ogni sessione mantiene un vault:

```python
@dataclass
class SessionState:
    vault: dict[str, str]    # placeholder → valore originale
    reverse: dict[str, str]  # valore originale → placeholder
    counters: dict[str, int] # entity_type → contatore
```

Presidio individua gli span contenenti PII. Per ogni span, il sistema riutilizza un placeholder esistente se quel valore è già stato incontrato nella sessione, oppure ne crea uno nuovo tipizzato: `[REDACTED_PERSON_1]`, `[REDACTED_EMAIL_ADDRESS_1]`.

Il prompt anonimizzato viene inviato a qualsiasi API LLM — OpenAI, Anthropic, Azure. La risposta torna con i placeholder intatti. Il vault li ripristina con i valori originali.

## Provarlo in locale

```bash
git clone https://github.com/iron87/anonymizer-poc
cd anonymizer-poc
docker compose up --build -d
```

Una richiesta di esempio:

```bash
curl -s http://localhost:9000/v1/agent/chat \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "demo-1",
    "model": "local/llama3.2",
    "message": "Riscrivi questo promemoria: contatta mario.rossi@example.com o il 3331234567 per la riunione."
  }' | jq .
```

Risposta:

```json
{
    "request_id": "demo-1",
    "model": "llama3.2",
    "content": "Ricordati di contattare mario.rossi@example.com o chiamare il 3331234567 per la riunione in programma."
}
```

Il modello non ha mai visto l’email reale né il numero di telefono. Ha lavorato su `[REDACTED_EMAIL_ADDRESS_1]` e `[REDACTED_PHONE_NUMBER_1]`. La risposta è stata ripristinata tramite il vault di sessione.

## Cosa ho imparato

### I placeholder tipizzati non sono un dettaglio

Senza informazione di tipo — `[REDACTED_1]` invece di `[REDACTED_EMAIL_ADDRESS_1]` — il modello perde contesto. Non sa se sta gestendo un nome, un’email o un identificativo arbitrario.

Con i tipi, il modello può ragionare correttamente: “questo è un placeholder email, va usato dove un’email è attesa.” La qualità della risposta cambia in modo misurabile.

### Il vault diventa la nuova superficie d’attacco

Hai spostato il problema, non eliminato. Il vault contiene la mappatura PII ed è il dato sensibile da proteggere.

In produzione servono almeno:

- storage persistente e cifrato (Redis con encryption at rest, o un DB dedicato);
- policy TTL per la scadenza delle sessioni;
- controlli di accesso espliciti su chi può interrogare il vault.

### Il gateway semplifica l’orchestrazione

Il PoC usa Plano come gateway con filtri di input e output. Un approccio equivalente è implementabile con LiteLLM, che offre un modulo dedicato per il PII masking con Presidio. In questo caso ho scelto Plano per sperimentare uno stack diverso da quello che uso di solito.

```yaml
listeners:
  - type: model
    port: 12000
    input_filters:
      - pii_anonymizer
    output_filters:
      - pii_deanonymizer
```

Plano gestisce l’orchestrazione dei filtri. L’applicazione invia richieste al gateway senza codice di routing custom. Ma il pattern funziona anche senza gateway: basta chiamare anonymize → LLM → deanonymize in sequenza.

## Conclusioni

L’anonimizzazione reversibile non è la soluzione per tutti i contesti, ma riempie uno spazio preciso: quando i modelli locali sono fuori portata ma i dati degli utenti non possono uscire non protetti.

Il punto più interessante non è tecnico. È organizzativo: questo pattern rende possibile usare i migliori modelli disponibili sul mercato senza rinunciare alla compliance, a patto di accettare che la complessità non sparisce ma si sposta — dal modello al vault, dall’infrastruttura GPU alla gestione del ciclo di vita delle sessioni.

Per un’agenzia tecnica che lavora con clienti enterprise, è esattamente il tipo di trade-off che vale la pena documentare, prototipare e avere pronto come risposta concreta quando arriva la domanda: “ma i nostri dati dove vanno?”

*PoC completo: [github.com/iron87/anonymizer-poc](http://github.com/iron87/anonymizer-poc)*