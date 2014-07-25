HTTP API: Guida alla Progettazione
==================================


## Descrizione

Questa guida descrive un insieme di pratiche di progettazione di HTTP+JSON API, basato su [HTTP API Design Guide](https://github.com/interagent/http-api-design).

Le idee presentate qui sono valide per qualsiasi linguaggio di programmazione e framework, quindi gli esempi di codice presentati sono in [linguaggio C#](http://en.wikipedia.org/wiki/C_Sharp_%28programming_language%29) per [Microsoft ASP.NET Web API](http://www.asp.net/web-api) per comodità nostra.


## Fondamenti


### Richiedi TLS

API _pubblica_ deve richiedere [TLS](http://en.wikipedia.org/wiki/Transport_Layer_Security) per l'accesso, senza eccezioni. Non vale la pena cercare di capire o spiegare quando è bene usare TLS e quando non lo è: richiedere TLS per tutto.


### Versione con Header "Accept"

Imposta la versione dell'API fin dall'inizio. Utilizzare i header `Accept` per comunicare la versione, insieme a un tipo di contenuto personalizzato, ad esempio: `Accept: application/vnd.foobar+json; version=3`.

> Una buona pratica è il [Versionamento Semantico](http://semver.org/lang/it/), ma ogni progetto potrebbe avere uno schema diverso.

#### Chiamate Incompatibili

Rifiuta le chiamate che non specificano la versione o le chiamate _incompatibili_.

Per abilitare automaticamente questa funzionalità si potrebbe utilizzare il filtro `ApiVersionCheckFilter` definito in `Common.Filters`, quindi registrare il filtro globalmente in `Application_Start`:

```cs
// Global.asax.cs
using Common.Filters;

public class WebApiApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        //// Sostituire `foobar` con il nome del progetto.
        GlobalConfiguration.Configuration.Filters.Add(
            new ApiVersionCheckFilter("foobar", "json", "3"));
    }
}
```


### Traccia le richieste con Request-Id

Includere un header `Request-Id` in ogni risposta, popolato con un valore [UUID](http://en.wikipedia.org/wiki/Universally_unique_identifier). Sia il server che il client potrebbero voler loggare questo valore per tenere traccia ed eventualmente fare il debugging delle richieste.

Per abilitare automaticamente questa funzionalità si potrebbe utilizzare il filtro `ApiRequestIdHeaderFilter` definito in `Common.Filters`, quindi registrare il filtro globalmente in `Application_Start`:

```cs
// Global.asax.cs
using Common.Filters;

public class WebApiApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        GlobalConfiguration.Configuration.Filters.Add(new ApiRequestIdHeaderFilter());
    }
}
```


## Richieste


### Restituire Status Code Appropriati

Restituire status code appropriato con un risposta HTTP. Le risposte con successo devono essere codificati in base a questa guida:

* `200`: Richiesta con successo per la chiamata `GET`, e la chiamate sincrona `DELETE`
* `201`: Richiesta con successo per la chiamata sincrona `POST`
* `202`: Richieste accettate per le chiamate asincrone `POST` e `DELETE`

Prestare attenzione all'uso di codici di errore di autenticazione e di autorizzazione:

* `401 Unauthorized`: Richiesta fallita perché l'utente non è autenticato
* `403 Forbidden`: Richiesta fallita perché l'utente non ha il permesso per accedere alla specifica risorsa

Per maggiori informazioni ed indicazioni su status code per i casi di errore dell'utente e di errore del server, continua a leggere "[Return appropriate status codes](https://github.com/interagent/http-api-design#return-appropriate-status-codes)" e fare riferimento a [HTTP response code spec](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).


### Accetta JSON nel Body della Richiesta

Per coerenza con body delle risposte, accetta JSON serializzato nel body delle ricchieste per `PUT` o `POST`. Ad esempio:

```
$ curl -X POST https://service.com/facolta \
    -H "Content-Type: application/json" \
    -d '{"name": "Facoltà di Economia", "dipartimento": {"id": "5d8201b0..."}}'

{
  "id": "7f54a2j1...",
  "nome": "Facoltà di Economia",
  "dipartimento": {
    "id": "5d8201b0...",
    "nome": "Dipartimento di Economia",
    "indirizzo": "Via Cras Vestibulum Consectetur, 12"
  },
  ...
}
```


### Utilizzare Formato Coerente per Percorsi

#### Nomi Risorse

Utilizzare la versione plurale di un nome risorsa a meno che la risorsa in questione è singolo all'interno del sistema (per esempio, nella maggior parte dei sistemi, un utente ha spesso un unico account):

```
/docenti/{docente_id}
/prenotazioni/{prenotazione_id}
/aule/{aula_id}
```

#### Azioni

Preferire endpoint che non necessitano particolari layout di azioni per singole risorse. Nei casi in cui sono necessarie azioni speciali, metterli sotto un prefisso `azioni` standard, per chiarezza:

```
/docenti/{docente_id}/azioni/prenota
```

Anche se l'ideale sarebbe (passando aula e docente nel body della richiesta):

```
POST /prenotazioni
```

#### Percorsi ed Attributi Minuscoli

Utilizzare i nomi in caratteri minuscoli e separati con `-`, per coerenza con i nomi di host:

```
servizio-api.com/utenti
servizio-api.com/impostazioni-account
```

Utilizzare i nomi in caratteri minuscoli anche nei nomi degli attributi, ma utilizzare `_` per separare le parole, così che in JavaScript, si possono scrivere senza le doppio apici:

```json
nome: "Mario"
numero_civico: 12
```

#### Minimizzare Nidificazione dei Percorsi

Nei modelli di dati con relazioni padre/figlio, i percorsi possono diventare nidificati, ad esempio:

```
/dipartimenti/{dipartimento_id}/facolta/{facolta_id}/studenti/{studente_id}
```

Minimizzare la profondità di nidificazione preferendo di localizzare le risorse a livello di root. Utilizzare la nidificazione per indicare le collezioni appartenenti ad un'ambito. Ad esempio, riprendendo l'esempio di prima, dove uno studente appartiene ad una facoltà che appartiene ad un dipartimento:

```
/dipartimenti/{dipartimento_id}
/dipartimenti/{dipartimento_id}/facolta
/facolta/{facolta_id}
/facolta/{facolta_id}/studenti
/studenti/{studente_id}
```


## Risposte


### Formato Data & Ora

Per gli attributi di data & ora accetta e rispondi soltanto in [UTC](http://en.wikipedia.org/wiki/Coordinated_Universal_Time). Formatta i valori di data & ora in [ISO8601](http://en.wikipedia.org/wiki/ISO_8601):

```json
"prenotato_in": "2012-01-01T12:00:00Z"
```


### Nidificare le relazioni dirette

Serializzare i riferimenti alle relazioni dirette con un oggetto nidificato:

```json
{
  "nome": "Facoltà di Economia",
  "dipartimento": {
    "id": "5d8201b0..."
  }
}
```

Anziché, ad esempio:

```json
{
  "nome": "Facoltà di Economia",
  "dipartimento_id": "5d8201b0..."
}
```

Questo approccio da la possibilità di aggiungere più informazioni per la risorsa relativa senza dover cambiare la struttura della risposta od altrimenti introdurre più campi di risposte a livello più alto. Ad esempio:

```json
{
  "nome": "Facoltà di Economia",
  "dipartimento": {
    "id": "5d8201b0...",
    "nome": "Dipartimento di Economia",
    "indirizzo": "Via Cras Vestibulum Consectetur, 12"
  }
}
```


### Genera Errori Strutturati

Nel caso di errori generare body di risposta coerenti. Includere un id per le macchine, un id per gli umani insieme ad un messaggio di error, ed opzionalmente un url che punta il client per richiedere maggiori informazioni sull'errore ed eventualmente come risolvere. Ad esempio:

```
HTTP/1.1 429 Too Many Requests
```

```json
{
  "id":      "limite_accesso",
  "message": "Account superato il limite di accesso API.",
  "url":     "https://example.com/limiti-accesso-api"
}
```

Per abilitare automaticamente questa funzionalità si potrebbe utilizzare il filtro `ApiErrorResponseFilter` definito in `Common.Filters`, quindi registrare il filtro globalmente in `Application_Start`:

```cs
// Global.asax.cs
using Common.Filters;

public class WebApiApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        GlobalConfiguration.Configuration.Filters.Add(new ApiErrorResponseFilter());
    }
}
```

Sarà sufficiente generare un'eccezione, quindi l'API risponderà:

```
HTTP/1.1 500 Internal Server Error
```

```json
{
  "id":      "api_unknown",
  "message": "Il messaggio specificato nell'eccezzione.",
  "url":     null
}
```

Se si vuole specificare lo status code ed informazioni diversi, allora si può creare un'eccezione custom ereditando l'eccezione di base `BaseApiException` definita in `Common.Exceptions`. Ad esempio:

```cs
public class ApiIncompatibileException : BaseApiException
{
    public ApiIncompatibileException(Exception innerException)
        : base(
            HttpStatusCode.NotAcceptable,
            "api_version",
            "Questo API non è compatibile la versione specificata dal client.",
            innerException: innerException)
    {
    }
}
```
