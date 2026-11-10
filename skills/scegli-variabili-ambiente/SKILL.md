---
name: scegli-variabili-ambiente
description: "Decides whether an environment variable belongs on Vercel, on Convex or on both by where the code reads it, and sets it without exposing its value. Use when adding, moving or missing a variable."
---

# Scegli dove mettere le variabili d'ambiente

## Principio

In un progetto Next.js su Vercel con backend Convex convivono due runtime, ciascuno con le sue variabili d'ambiente. Convex legge solo le proprie e lo stesso vale per Vercel. Metti una chiave solo dove il codice la legge con `process.env`, senza duplicarla per prudenza.

## Decisione

1. Cerca `process.env.NOME` nel codice Next, di solito in `app/` o `src/`, e nella cartella `convex/`.
2. Se compare solo nel codice Next, la chiave va solo su Vercel con `vercel env add NOME production`, più `preview` e `development` quando servono.
3. Se compare solo in `convex/`, la chiave va solo su Convex con `bunx --bun convex env set NOME --prod`, passando il valore dal prompt o da stdin e mai in chiaro nel comando.
4. Se compare in entrambi, la chiave va su entrambi con lo stesso valore.
5. I segreti che fanno da ponte tra Next e Convex, per esempio quello con cui Next firma le chiamate alle funzioni Convex, vanno sempre su entrambi con lo stesso valore. Allinea anche le coppie che hanno nomi diversi sui due host, come `ADMIN_EMAIL` su Convex e `NEXT_PUBLIC_ADMIN_EMAIL` su Vercel.

## Dopo una modifica

- Dichiara dove hai messo la chiave, cioè Vercel, Convex o entrambi.
- Su Vercel una variabile nuova o cambiata arriva al codice solo con un nuovo deploy. Su Convex è attiva subito.
- Le variabili `NEXT_PUBLIC_*` vengono incorporate nel bundle durante la build, quindi ogni cambio richiede un rebuild.
- Per il deployment di produzione Convex usa sempre `--prod`. Senza il flag il comando scrive sul deployment di sviluppo.

## Quando una chiave manca

- Se un pannello di stato, un health check o un errore segnala una chiave assente su un host, trattala come un'assenza reale e verificala su quell'host prima di concludere che basta l'altro.
- Se il progetto tiene un catalogo delle chiavi attese per host, aggiornalo quando sposti una chiave. Per far sparire un avviso correggi il catalogo, invece di aggiungere la chiave a un host che non la legge.

## Cosa evitare

- Dare per scontato che Convex veda le variabili di Vercel, o il contrario.
- Stampare i valori in chat, nei log o nei commit. Per controllare la presenza bastano i nomi. `vercel env ls` mostra solo quelli, mentre `convex env list` stampa anche i valori, quindi filtrane l'output con `bunx --bun convex env list --prod | cut -d= -f1`.
