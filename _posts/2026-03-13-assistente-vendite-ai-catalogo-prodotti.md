---
layout: post
title: "Dal catalogo sparso a un assistente vendite AI per rispondere ai clienti"
date: 2026-03-13
author: 2brainlab
tags:
	- "AI sales enablement"
	- "catalog intelligence"
	- n8n
	- "chatbot aziendale"
	- "product data"
	- "automazione commerciale"
excerpt: "Quando le informazioni di prodotto sono distribuite tra PDF, sito web e materiali non strutturati, il team vendite perde tempo e rischia risposte incomplete. In questo use case abbiamo centralizzato il catalogo, costruito un agente in n8n e reso disponibili risposte tecniche immediate tramite una chat usata direttamente dai venditori."
---

## TL;DR

Quando le informazioni di prodotto sono distribuite tra PDF, sito web e
materiali non strutturati, il team vendite perde tempo e rischia risposte
incomplete. In questo use case abbiamo centralizzato il catalogo, costruito un
agente in n8n e reso disponibili risposte tecniche immediate tramite una chat
usata direttamente dai venditori.

## Il problema

Il cliente aveva un catalogo prodotti ricco di informazioni, ma estremamente
frammentato:

- schede tecniche in PDF;
- contenuti pubblicati sul sito;
- dettagli distribuiti in fonti diverse, con formati e aggiornamenti non uniformi.

Questo creava un problema operativo molto concreto. I venditori dovevano
rispondere rapidamente ai clienti, spesso su aspetti tecnici dei prodotti, ma
non avevano sempre il livello di competenza necessario per recuperare e
interpretare in autonomia tutte le informazioni disponibili.

Il risultato era prevedibile:

- tempi più lunghi per rispondere;
- dipendenza da figure tecniche interne;
- rischio di dare informazioni parziali o incoerenti;
- bassa scalabilità del supporto commerciale.

In pratica, il problema non era solo “trovare i dati”, ma renderli
interrogabili in modo semplice, affidabile e utile nel flusso quotidiano del
team sales.

## La soluzione

Abbiamo progettato uno strumento interno pensato per un obiettivo preciso:
permettere ai venditori di fare una domanda in linguaggio naturale e ottenere
subito una risposta tecnica utilizzabile con il cliente.

L’architettura è stata costruita in tre passaggi:

1. **Raccolta delle fonti**: abbiamo implementato parser e scraper per
	recuperare le informazioni da PDF, sito web e altre sorgenti rilevanti.
2. **Razionalizzazione del catalogo**: abbiamo normalizzato i dati, eliminato
	ridondanze e trasformato contenuti dispersi in una base informativa
	coerente.
3. **Accesso conversazionale**: abbiamo costruito un agente con n8n,
	interrogabile tramite il nodo chat di n8n, così che i venditori potessero
	usarlo online come un assistente operativo.

Per la persistenza abbiamo inserito il catalogo consolidato in un database
relazionale. Questa scelta ci ha dato una struttura chiara per prodotti,
attributi, relazioni e varianti, semplificando sia la manutenzione sia il
recupero delle informazioni da parte dell’agente.

L’agente non è quindi un semplice chatbot “generico”, ma un’interfaccia
conversazionale sopra una base dati razionalizzata e mantenibile.

## Walkthrough pratico

Di seguito il flusso di lavoro che abbiamo seguito.

### 1. Mappatura delle fonti informative

Il primo step è stato identificare dove vivevano davvero le informazioni utili:

- PDF commerciali e tecnici;
- pagine prodotto del sito;
- eventuali contenuti descrittivi o schede accessorie.

Questa fase è fondamentale perché, in molti casi, il problema non è l’assenza
di dati ma la loro dispersione.

### 2. Parser e scraper per estrarre il contenuto

Abbiamo costruito componenti dedicati per:

- leggere i PDF e recuperarne il testo utile;
- estrarre le informazioni strutturate e semi-strutturate dal sito;
- uniformare campi, nomenclature e unità di misura dove necessario.

L’obiettivo non era solo “ingestire documenti”, ma trasformare materiali
eterogenei in dati realmente utilizzabili in produzione.

### 3. Modello dati relazionale

Una volta raccolte le informazioni, le abbiamo organizzate in un database
relazionale.

Questo passaggio ha permesso di:

- centralizzare il catalogo;
- definire relazioni chiare tra prodotti e caratteristiche;
- gestire meglio aggiornamenti e manutenzione;
- dare all’agente una fonte più affidabile rispetto a documenti isolati.

### 4. Agente conversazionale in n8n

Sopra questa base abbiamo costruito un agente in n8n, esposto tramite il nodo
chat.

Dal punto di vista del venditore, l’esperienza è semplice:

- scrive una domanda;
- l’agente recupera le informazioni rilevanti;
- restituisce una risposta sintetica e operativa.

Dal punto di vista del business, il valore è ancora più chiaro:

- meno interruzioni verso il team tecnico;
- risposte più rapide ai clienti;
- maggiore autonomia commerciale;
- migliore utilizzo del patrimonio informativo già esistente.


![Workflow n8n assistente vendite]({{ '/assets/img/blog/sales-assistant.png' | relative_url }})

## Conclusioni e prossimi passi

Questo use case mostra un pattern molto ricorrente nei progetti AI applicati
alle operations commerciali: il valore non nasce dal chatbot in sé, ma dalla
qualità del lavoro fatto prima sul dato.

La parte decisiva è stata trasformare un catalogo disperso in una base
informativa interrogabile. Solo a quel punto l’agente in n8n è diventato
davvero utile per i venditori.

I prossimi passi naturali, in un’evoluzione di questo progetto, sarebbero:

- introdurre workflow di aggiornamento automatico delle fonti;
- aggiungere tracciamento delle domande più frequenti;
- misurare tempi di risposta e impatto sul tasso di conversione;
- collegare il sistema a CRM o strumenti di customer support.

Per un’agenzia come la nostra, è anche un caso interessante da riutilizzare nel
framework prototipi: stessa logica, fonti diverse, stesso outcome operativo.
