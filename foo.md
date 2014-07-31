## Descrizione

API per l'integrazione con Agenda effedesign.

Sono previsti due tipologie di API:

* **Permessi**: dovrà restituire le logiche di effedesign per la visualizzazione o il blocco di una certa funzionalità
* **Funzioni**: dovrà restituire il contenuto HTML dei _box_ o _widget_

N.B.: la lingua non è necessaria come parametro di input nel body della richiesta poiché è già previsto dallo standard HTTP come header `Accept-Language` con la codifica standard internazionale.
Nelle risposte prenderemo come valida la lingua specificata nell'header `Content-Language`.


## /permessi [GET]

I permessi di tutte le funzionalità per utente.

### Richiesta

Nel body della richiesta viene passata un utente.

Le tipologie di _contesto_ rappresentano le varie categorie di attributi disponibili per l'utente connesso all'Agenda2.0. Partiamo dal passaggio della `carriera_studente` (praticamente è il valore di `stu_id`.)

```
Accept:          application/vnd.portal+json; version=1
Content-Type:    application/json
Accept-Language: en-US
```

```json
{
  "id" : "m.rossi",
  "contesto": {
    "id": "df62f04...",
    "tipo": "carriera_studente|profilo_azienda|ecc."
  }
}
```

### Risposta

Rappresenta i permessi di tutte le funzionalità.

```
Request-Id:       e819a6e2-b170-44a3-9f19-094da6bf2221
Content-Type:     application/json; charset=utf-8
Content-Language: it
```

```json
[
  {
    "funzione": {
      "id": "funz_uno"
    },
    "visibile": true,
    "bloccato": false
  },
  {
    "funzione": {
      "id": "funz_due"
    },
    "visibile": true,
    "bloccato": true
  },
  {
    "funzione": {
      "id": "funz_tre"
    },
    "visibile": false,
    "bloccato": true
  }
]
```

> N.B.: i flag `visibile`, `bloccato` potremmo anche unirli in un unico `stato`.


## /funzioni [GET]

I contenuti di tutte le funzionalità per utente.

### Richiesta

Nel body della richiesta viene passata un utente.

Le tipologie di _contesto_ rappresentano le varie categorie di attributi disponibili per l'utente connesso all'Agenda2.0. Partiamo dal passaggio della `carriera_studente` (praticamente è il valore di `stu_id`.)

```
Accept:          application/vnd.portal+json; version=1
Content-Type:    application/json
Accept-Language: en-US
```

```json
{
  "id" : "m.rossi",
  "contesto": {
    "id": "df62f04...",
    "tipo": "carriera_studente|profilo_azienda|ecc."
  }
}
```

### Risposta

Rappresenta il contenuto HTML di più _box_/_widget_.

> N.B.:
>
> * non prevediamo che ci vengano restituiti i tag `<html>`, `<head>`, `<body>`, ecc.
> * il testo contenuto dell'HTML sarà nella lingua specificata da `Content-Language` nella risposta, quanto richiesto con `Accept-Language`.

```
Request-Id:       e819a6e2-b170-44a3-9f19-094da6bf2221
Content-Type:     application/json; charset=utf-8
Content-Language: it
```

```json
[
  {
    "id": "funz_uno",
    "contenuto_html": "<p>Content of the document......</p>"
  },
  {
    "id": "funz_due",
    "contenuto_html": "<p>Content of the document......</p>"
  },
  {
    "id": "funz_tre",
    "contenuto_html": "<p>Content of the document......</p>"
  }
]
```


## /funzioni/{nome_funzione} [GET]

Il contenuto di una singola funzionalità per utente.

### Richiesta

Nel body della richiesta viene passata un utente.

Le tipologie di _contesto_ rappresentano le varie categorie di attributi disponibili per l'utente connesso all'Agenda2.0. Partiamo dal passaggio della `carriera_studente` (praticamente è il valore di `stu_id`.)

```
Accept:          application/vnd.portal+json; version=1
Content-Type:    application/json
Accept-Language: en-US
```

```json
{
  "id" : "m.rossi",
  "contesto": {
    "id": "df62f04...",
    "tipo": "carriera_studente|profilo_azienda|ecc."
  }
}
```

### Risposta

Rappresenta il contenuto HTML di uno _box_/_widget_.

> N.B.:
>
> * non prevediamo che ci vengano restituiti i tag `<html>`, `<head>`, `<body>`, ecc.
> * il testo contenuto dell'HTML sarà nella lingua specificata da `Content-Language` nella risposta, quanto richiesto con `Accept-Language`.

```
Request-Id:       e819a6e2-b170-44a3-9f19-094da6bf2221
Content-Type:     application/json; charset=utf-8
Content-Language: it
```

```json
{
  "id": "funz_uno",
  "contenuto_html": "<p>Content of the document......</p>"
}
```


## /funzioni/{nome_funzione}/permessi [GET]

I permessi di una singola funzionalità per utente.

### Richiesta

Nel body della richiesta viene passata un utente.

Le tipologie di _contesto_ rappresentano le varie categorie di attributi disponibili per l'utente connesso all'Agenda2.0. Partiamo dal passaggio della `carriera_studente` (praticamente è il valore di `stu_id`.)

```
Accept:          application/vnd.portal+json; version=1
Content-Type:    application/json
Accept-Language: en-US
```

```json
{
  "id" : "m.rossi",
  "contesto": {
    "id": "df62f04...",
    "tipo": "carriera_studente|profilo_azienda|ecc."
  }
}
```

### Risposta

Rappresenta i permessi di una certa funzionalità.

```
Request-Id:       e819a6e2-b170-44a3-9f19-094da6bf2221
Content-Type:     application/json; charset=utf-8
Content-Language: it
```

```json
[
  {
    "funzione": {
      "id": "qualche_funzione"
    },
    "visibile": false,
    "bloccato": true
  }
]
```

> N.B.:
>
> * i flag `visibile`, `bloccato` potremmo anche unirli in un unico `stato`.
> * Anche se sappiamo a priori che non ci dovrebbe essere _mai_ più di un permesso per funzionalità, [è preferibile](https://github.com/Apex-net/AgendaBocconi/wiki/HTTP-API:-Guida-alla-Progettazione#minimizzare-nidificazione-dei-percorsi) esporre le risorse appartenenti in un ambito comunque come una collezione.
