## Descrizione

API per l'integrazione con Agenda effedesign.

Sono previsti due tipologie di API:

* **Permessi**: dovrà restituire le logiche di effedesign per il blocco di una certa funzionalità visibile per l'utente
* **Funzioni**: dovrà restituire il contenuto HTML dei _box_ o _widget_ visibili per l'utente

N.B.: la lingua non è necessaria come parametro di input nel body della richiesta poiché è già previsto dallo standard HTTP come header `Accept-Language` con la codifica standard internazionale.
Nelle risposte prenderemo come valida la lingua specificata nell'header `Content-Language`.


## /utenti/{id_utente}/permessi?contesto={cod_contesto} [GET]

I permessi di tutte le funzionalità per utente, con la possibilità di filtrare per contesto (carriera).

### Richiesta

* `{id_utente}` è lo user-id dell'utente richiesto.
* `{cod_contesto}` serve attualmente per gestire la carriera degli studenti. Va bene la concatenzaione di `Carriera-STU_ID`

Le tipologie di _contesto_ rappresentano le varie categorie di attributi disponibili per l'utente connesso all'Agenda2.0. Partiamo dal passaggio della `carriera_studente` (praticamente è il valore di `stu_id`.)

```
Accept:          application/vnd.portal+json; version=1
Content-Type:    application/json
Accept-Language: en-US
```

### Risposta

Rappresenta i permessi di tutte le funzionalità visibili per l'utente.

```
Request-Id:       e819a6e2-b170-44a3-9f19-094da6bf2221
Content-Type:     application/json; charset=utf-8
Content-Language: it
```

```json
[
  {
    "funzione": {
      "cod": "funz_uno"
    },
    "evidenziato": true,
    "bloccato": false
  },
  {
    "funzione": {
      "cod": "funz_due"
    },
    "evidenziato": true,
    "bloccato": true
  },
  {
    "funzione": {
      "cod": "funz_tre"
    },
    "evidenziato": false,
    "bloccato": true
  }
]
```


## /utenti/{id_utente}/funzioni?contesto={cod_contesto} [GET]

I contenuti di tutte le funzionalità per utente, con la possibilità di filtrare per contesto (carriera).

### Richiesta

* `{id_utente}` è lo user-id dell'utente richiesto.
* `{cod_contesto}` serve attualmente per gestire la carriera degli studenti. Va bene la concatenzaione di `Carriera-STU_ID`

```
Accept:          application/vnd.portal+json; version=1
Content-Type:    application/json
Accept-Language: en-US
```


### Risposta

Rappresenta il contenuto HTML di uno o più _box_/_widget_.

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
    "cod": "funz_uno",
    "contenuto_html": "<p>Content of the document......</p>",
    "permesso": {
        "evidenziato": true,
        "bloccato": true
    }   
  },
  {
    "cod": "funz_due",
    "contenuto_html": "<p>Content of the document......</p>",
    "permesso": {
        "evidenziato": true,
        "bloccato": true
    }   
  },
  {
    "cod": "funz_tre",
    "contenuto_html": "<p>Content of the document......</p>",
    "permesso": {
        "evidenziato": true,
        "bloccato": true
    }   
  }
]
```

## /utenti/{id_utente}/funzioni/{cod_funzione}?contesto={cod_contesto} [GET]

Il contenuto di una singola funzionalità per utente, con possibilità di filtro per contesto.

### Richiesta

* `{id_utente}` è lo user-id dell'utente richiesto.
* `{cod_funzione}` codice univoco che individua una funzionalità.
* `{cod_contesto}` serve attualmente per gestire la carriera degli studenti. Va bene la concatenzaione di `Carriera-STU_ID`

```
Accept:          application/vnd.portal+json; version=1
Content-Type:    application/json
Accept-Language: en-US
```


### Risposta

Rappresenta il contenuto HTML di un _box_/_widget_.

> N.B.:
>
> * in caso di passaggio di un {cod_funzione} non visibile per l'utente va restituito uno **STATUS_CODE = 204** (No Content)
> * non prevediamo che ci vengano restituiti i tag `<html>`, `<head>`, `<body>`, ecc.
> * il testo contenuto dell'HTML sarà nella lingua specificata da `Content-Language` nella risposta, quanto richiesto con `Accept-Language`.

```
Request-Id:       e819a6e2-b170-44a3-9f19-094da6bf2221
Content-Type:     application/json; charset=utf-8
Content-Language: it
```

```json
{
  "cod": "funz_uno",
  "contenuto_html": "<p>Content of the document......</p>",
  "permesso": {
     "evidenziato": true,
     "bloccato": true
   }
}
```


## /utenti/{id_utente}/funzioni/{cod_funzione}/permesso?contesto={cod_contesto} [GET]

I permessi di una singola funzionalità per utente, con possibilità di filtro per contesto.

### Richiesta

* `{id_utente}` è lo user-id dell'utente richiesto.
* `{cod_funzione}` codice univoco che individua una funzionalità.
* `{cod_contesto}` serve attualmente per gestire la carriera degli studenti. Va bene la concatenzaione di `Carriera-STU_ID`

```
Accept:          application/vnd.portal+json; version=1
Content-Type:    application/json
Accept-Language: en-US
```

### Risposta

Rappresenta il permesso di una certa funzionalità dell'utente.

```
Request-Id:       e819a6e2-b170-44a3-9f19-094da6bf2221
Content-Type:     application/json; charset=utf-8
Content-Language: it
```

```json
{
    "funzione": {
      "cod": "funz_uno"
    },
    "evidenziato": true,
    "bloccato": false
}
```

> N.B.:
>
> * in caso di passaggio di un `{cod_funzione}` non visibile per l'utente va restituito uno **STATUS_CODE = 204** (No Content)
