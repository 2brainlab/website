---
layout: post
title: "Spec-kit e agenti AI: la lezione vera è la disciplina di delivery"
date: 2026-03-27
author: "Federico Cerruto"
tags:
  - "Spec-driven development"
  - "Agenti AI"
  - "AI Engineering"
  - "Delivery"
  - "Platform Engineering"
excerpt: "Spec-kit è utile non tanto perché accelera il primo giorno, ma perché rende più leggibile il decimo: decisioni tracciate, task più piccoli e un feedback loop che resta gestibile anche quando il sistema inizia a complicarsi."
---

## TL;DR

Spec-kit è utile non tanto perché accelera il primo giorno, ma perché rende più leggibile il decimo. Quando lavori con agenti AI su un sistema nuovo, il valore reale non sta nel volume di codice prodotto, ma nella qualità del feedback loop: decisioni esplicite, task piccoli, validazione continua.

## Il problema

Questo non è un articolo sul contrasto tra vibe coding e sviluppo strutturato. È un articolo sulla disciplina di delivery.

L’obiettivo era costruire una piattaforma backend per agenti AI abbastanza concreta da generare apprendimento reale, non una demo buona solo per una presentazione veloce. Il perimetro includeva:

- bootstrap con un solo comando;
- invocazione API multi-tenant;
- sessioni conversazionali con memoria;
- tracing delle esecuzioni;
- policy controls;
- stack con FastAPI, LangGraph, LiteLLM, Redis, Postgres, Langfuse e Docker Compose.

Su carta è una lista gestibile. In pratica, è il tipo di architettura che diventa fragile molto in fretta se mancano ordine, tracciabilità delle decisioni e una sequenza di implementazione disciplinata.

Il punto non era solo “far funzionare i componenti”. Il punto era capire se un workflow spec-driven regge davvero quando il progetto smette di essere banale e iniziano a crescere integrazioni, vincoli operativi e superficie di cambiamento.

## La soluzione

Per questo esperimento ho scelto spec-kit, il toolkit open source di GitHub per la spec-driven development.

L’idea è semplice: chiarire l’intento prima dell’implementazione e mantenere vivi gli artefatti che guidano il delivery. Nel concreto, il workflow si articola in cinque fasi:

1. **Constitution**: principi e vincoli non negoziabili.
2. **Specify**: comportamento atteso, valore e acceptance criteria.
3. **Plan**: architettura, stack, tradeoff e vincoli tecnici.
4. **Tasks**: task ordinati e dipendenti, pronti per l’esecuzione.
5. **Implement**: sviluppo incrementale con validazione continua.

La parte interessante non è il numero di file generati. È il fatto che ogni decisione trovi un posto preciso in cui vivere.

Un vincolo non resta nel prompt del momento. Un tradeoff non si perde nella chat history. Un criterio di accettazione non rimane implicito nella testa di chi guida l’agente.

## Walkthrough pratico

La lezione più utile non è arrivata dai tool scelti, ma da come il lavoro ha avanzato davvero.

### 1. Prima la specifica, poi il codice

Prima di toccare il codice applicativo sono stati definiti gli artefatti di specifica: constitution, spec, checklist, plan e task list.

Questa sequenza sembra formale, ma in realtà cambia il comportamento del progetto. Costringe a chiarire subito quali vincoli contano davvero e impedisce di demandare tutto all’improvvisazione del momento.

### 2. I vincoli infrastrutturali vanno dichiarati presto

Il requisito di bootstrap con un solo comando, per esempio, è stato scritto nel piano prima ancora che esistesse il Compose file.

Questo ha avuto un effetto concreto: l’infrastruttura è stata costruita per rispettare il vincolo fin dall’inizio, invece di retrofittarlo più tardi con workaround, health check tardivi e startup order impliciti.

Quando un vincolo arriva tardi, di solito si paga due volte: una in implementazione e una in debugging.

### 3. Il vero valore dei task è mantenere chiara la causalità

Nel modo corretto di usare spec-kit, il file dei task non è un backlog generico. È un contratto di esecuzione:

1. prendi un task;
2. implementa solo quel task;
3. valida il risultato;
4. marca il task come completato;
5. passa al successivo.

Il vantaggio non è burocratico. È diagnostico.

Se il cambiamento è piccolo, capisci rapidamente cosa ha rotto cosa. Se unisci troppi task in un’unica ondata di lavoro, perdi proprio il beneficio principale del metodo: mantenere leggibile il rapporto tra decisione, modifica e risultato.

### 4. Il feedback loop si rompe molto prima di quanto sembri

A un certo punto ho rotto io stesso questa disciplina: 64 file modificati in un solo burst, con circa 2300 linee aggiunte.

![Esperimento spec-kit: il punto in cui il change set diventa troppo grande]({{ '/assets/img/blog/spec-kit.png' | relative_url }})

Da lì il feedback loop ha iniziato a degradarsi:

- troppa superficie modificata in una volta sola;
- catena causale poco chiara quando qualcosa falliva;
- volume di codice alto, ma confidenza reale bassa;
- rollback costoso, perché eliminava insieme parti corrette e parti sbagliate.

Questa è probabilmente la lezione più importante dell’esperimento: spec-kit è una struttura, non un guardrail. Il metodo non ti impedisce di sbagliare. Funziona solo se mantieni davvero il lavoro in slice piccole e verificabili.

### 5. Gli errori utili vanno promossi a regole di delivery

In un sistema nuovo, i requisiti che contano emergono spesso mentre costruisci e fai girare il software. Li scopri negli edge case, negli attriti tra servizi, nei test fragili, nelle configurazioni ambigue.

Per questo la domanda utile non è “quanto codice ha scritto l’agente?”, ma “quale vincolo mancava nella specifica per rendere quel comportamento prevedibile?”.

Quando il processo funziona, anche i failure diventano materiale di progetto:

- un problema di startup ordering diventa un vincolo esplicito nel plan;
- un’ambiguità sul model routing diventa una regola chiara di configurazione;
- un test instabile tra locale e CI diventa un checkpoint di validazione nel task.

Questo è il punto in cui la costruzione del software smette di produrre solo output e inizia a produrre conoscenza riusabile.

## Conclusioni e prossimi passi

Il takeaway più interessante non riguarda la velocità iniziale. Riguarda la qualità del feedback loop.

Spec-kit mi è stato utile perché ha reso espliciti i vincoli prima del codice, ha dato una casa alle decisioni architetturali e ha reso più naturale lavorare in task piccoli e verificabili. Ma la lezione vera è un’altra: il metodo da solo non basta. Se torni a batch troppo grandi, test tardivi e validazione opaca, anche un workflow specification-driven perde gran parte del suo valore.

È anche il motivo per cui faccio fatica a vedere utile un agente lasciato in autonomia per ore su qualcosa di nuovo e ancora poco compreso. Su ticket circoscritti il perimetro è noto. Su un sistema nuovo, invece, il lavoro serve anche a scoprire i vincoli. E quel loop di scoperta richiede presenza umana continua, non solo review finale.

Per un’agenzia tecnica, il punto pratico è chiaro: il vantaggio competitivo non sta nel “far scrivere più codice all’agente”, ma nel costruire un processo che renda il delivery più leggibile, più revisionabile e più riusabile nel tempo.

I prossimi passi naturali, dopo un esperimento di questo tipo, sono tre:

1. formalizzare una pratica di task slicing per i progetti agentici;
2. rendere espliciti i vincoli minimi di delivery nei prototipi multi-servizio;
3. trasformare i failure ricorrenti in regole operative e checklist di validazione.