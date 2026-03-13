---
layout: post
title: "Implementare un agente con GitHub Copilot: dall'idea al commit"
date: 2026-03-10
author: 2brainlab
tags: ["GitHub Copilot", "AI Engineering", "Agenti AI", Automazione, "Developer Experience"]
excerpt: "GitHub Copilot non è solo autocompletamento. Configurato come agente, diventa uno strumento per automatizzare task ripetitivi, generare contenuti strutturati e mantenere la knowledge base aggiornata senza effort manuale."
---

## TL;DR
GitHub Copilot non è solo autocompletamento. Configurato come agente, diventa uno strumento per automatizzare task ripetitivi, generare contenuti strutturati e mantenere la knowledge base aggiornata senza effort manuale.

## Il problema
Lavorare in un team piccolo significa ogni ora spesa su task ripetitivi è un'ora tolta a lavoro ad alto valore. La documentazione si accumula in ritardo, i contenuti vengono rimandati e le decisioni rimangono nella memoria delle persone anziché nel repository.

Il rischio concreto per un'agenzia di due persone:

- knowledge base che invecchia senza aggiornamenti;
- articoli mai pubblicati perché "manca tempo per scriverli";
- decisioni prese oralmente e non documentate;
- overhead elevato per ogni nuovo cliente o progetto.

## La soluzione
Invece di usare Copilot come strumento di completamento passivo, lo configuriamo come agente attivo con:

1. **Istruzioni contestuali** (`copilot-instructions.md`) che definiscono template, convenzioni e comandi operativi;
2. **Workflow predefiniti** per i task più comuni (articoli, tool evaluation, ADR, prototipi);
3. **Integrazione con il repository** come unica fonte di verità per tutto il lavoro dell'agenzia.

Il risultato: un ciclo di lavoro dove l'agente conosce la struttura del progetto, segue le convenzioni stabilite e produce output immediatamente utilizzabili.

## Walkthrough pratico

### 1) Definisci le istruzioni base

Il file `.github/copilot-instructions.md` è il punto di partenza. Struttura le istruzioni in tre blocchi:

- **Contesto dell'organizzazione**: chi siete, cosa fate, quali sono le directory chiave;
- **Comandi operativi**: mapping tra richiesta in linguaggio naturale e azione concreta;
- **Principi trasversali**: lingua, tono, convenzioni di naming.

Regola chiave: sii specifico sui template, non sui risultati. L'agente è bravo a rispettare strutture, non a indovinare intenzioni vaghe.

### 2) Configura i template per ogni tipo di contenuto

Per ogni tipo di contenuto nel repository definisci:

- percorso file (`content/articles/YYYY-MM-DD-slug.md`);
- struttura del front matter (per blog, valutazioni tool, ADR);
- sezioni obbligatorie e sezioni facoltative;
- esempi di output atteso.

Questo riduce drasticamente il tempo di review dell'output generato.

### 3) Integra la generazione nel flusso quotidiano

L'agente è più efficace quando lo usi in modo routinario, non episodico. Alcuni pattern che funzionano:

- **Fine sprint**: chiedi all'agente una sintesi delle decisioni prese e crea ADR automaticamente;
- **Dopo una demo**: usa il template walkthrough per documentare lo step-by-step mentre è ancora fresco;
- **Prima di una call cliente**: chiedi lo stato dell'agenzia e usa il report come briefing.

### 4) Misura la riduzione del friction

Dopo 2-3 settimane misura:

- tempo medio per produrre un articolo (prima vs. dopo);
- numero di ADR create nel periodo;
- percentuale di template rispettati senza correzioni manuali.

Se il friction non scende, rivedere le istruzioni è più efficace che cambiare strumento.

### 5) Itera le istruzioni come codice

Le `copilot-instructions.md` vanno trattate come codice di produzione:

- versionamento con commit message descrittivi;
- review dopo ogni output non soddisfacente;
- test rapido: fai la stessa richiesta prima e dopo la modifica e confronta.

## Conclusioni e prossimi passi
Un agente ben configurato non sostituisce il giudizio umano — lo amplifica. Il valore non è nell'automazione cieca, ma nel ridurre il costo operativo delle task note per liberare capacità cognitiva per quelle nuove.

Prossimi passi per implementare questo setup:

1. creare o revisionare il file `copilot-instructions.md` con i comandi operativi core;
2. definire i template per i 3-4 contenuti più frequenti;
3. testare con un ciclo di lavoro reale (una settimana);
4. misurare friction prima e dopo;
5. documentare i pattern che funzionano in una pratica ingegneristica.
