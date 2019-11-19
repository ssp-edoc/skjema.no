# viewer.js

Pakken inneholder koden for viewer.js.

## Installasjon

Legg inn viewer.min.js nederst på HTML siden før body slutt tag.
```html
    <body>
        ...
        <script src="viewer.min.js"></script>
    </body>
```

Dersom du ønsker å benytte samme stilsett som på skjema.no, kan du legge inn styles.css i head taggen.

```html
    <head>
        ...
        <link rel='stylesheet' href='styles.css' type='text/css'  />
    </head>
```

Opprett et tomt element der skjemaet skal rendres. Du bestemmer elementets id og oppgir denne i kall til `viewer.init()` (beskrevet nedenfor).

```html
<body>
    <div id="skjema"></div>
</body>
```

For å vise skjema, kalles `.init()` og deretter `.form()`. Koden nedenfor viser strukturen for oppsett av viewer.js for visning av et skjema.

```javascript
document.addEventListener("DOMContentLoaded", function(){
    viewer
        .init({
            apiUrl: "<api url>",
            customerId: "<customer id>",
        })
        .form({
            divId: "<id til div der skjema skal rendres>",
            formId: "<id til skjema som skal vises>"
        });
});
```

## API

### Uthenting av versjonsnummer

For å finne versjonsnummer på viewer.js, kan du i konsollet skrive:
```javascript
viewer.version
```

### `init()`
`.init()` initialiserer viewer.js og må være det første kallet til viewer.js.

```javascript
viewer.init({...})
```

#### init object: 

```javascript
viewer.init({
    apiUrl : "<api url>",
    customerId: "<customer id>",
    hide : {
        title: true, 
        buttons: true, 
    }
})
```

|Egenskap|Eksempel|Obligatorisk|Beskrivelse|
|--------|--------|------------|-----------|
|apiUrl  |https://api.skjema.no|Ja|Url til API-et|
|customerId|9998|Ja|Kunde-id|
|hide.title|true|Nei|Oppgi true dersom det er ønskelig å skjule tittel|
|hide.buttons|true|Nei|Oppgi true dersom det er ønskelig å skjule knappene for "avbryt", "lagre" og til "kontroll"|

### `form()`

`.form()` forteller viewer.js hvilket skjema som skal vises. `.init()` må kalles før `.form()`.

```javascript
viewer
    .init({...})
    .form({...})
```

#### form object:

```javascript
viewer
    .init({...})
    .form({
        divId: "<id til div>",
        formId: "<id til skjema som skal vises eller function() som returnerer denne>",
        refId: "<id til mellomlagret utkast som skal vises eller function() som returnerer denne>",
        previewId: "<id til forhåndsvisning eller function() som returnerer denne>",
        onAborting: function(proceed) {},
        onAborted: function() {},
        onSaving: function(proceed) {},
        onSaved: function() {},
        onSubmitted: function() {}        
    });
```

`divId` er obligatorisk og må oppgis.

##### Angi skjema
viewer.js trenger én av følgende for å vise et skjema: `formId`, `refId` eller `previewId`. 

* `formId` benyttes når et nytt, tomt skjema skal vises. For å se hvilke `formId`-er som er tilgjengelig, kan du gjøre et kall til `viewer.getForms()` (beskrevet nedenfor).
* `refId` benyttes når et mellomlagret utkast skal gjenopptas. For å se hvilke `refId`-er som er tilgjengelig, kan du gjøre et kall til `viewer.getCases()` (beskrevet nedenfor).
* `previewId` vil bli sendt med når edoc-designer åpner URL for forhåndsvisning av skjema.

Det er vanlig at `formId`, `refId` eller `previewId` sendes med som querystring-parametre til HTML-siden. viewer.js har en utility-metode for uthenting av parameter fra querystring: `viewer.util.getQuery(paramName)`. Eksempelet nedenfor henter parametre med navn "formId", "refId" og "previewId" fra querystring. Dersom parameteren ikke finnes, returnerer funksjonen `null`. 

```javascript
viewer
    .init({...})
    .form({
        ...
        formId: function() {
            return viewer.util.getQuery("formId");
        },
        refId: function() {
            return viewer.util.getQuery("refId");
        },        
        previewId: function() {
            return viewer.util.getQuery("previewId");
        }					
    });
```

Verdien for `formId`, `refId` eller `previewId` kan alternativt "hardkodes" slik:

```javascript
viewer
    .init({...})
    .form({
        ...
        formId: "701660",
    });
```

Merk at viewer.js vil hive en exception dersom mer enn én av `formId`, `refId` eller `previewId` er satt. 

##### Event-handling
Når et skjema er startet, kan utfyller avbryte, lagre eller sende inn skjemaet. viewer.js tilbyr event-handlere som gjør det mulig å hooke seg på disse hendelsene.

Koden nedenfor viser et eksempel på hvordan de viktigste event-handlerne kan implementeres.

```javascript
viewer
    .init({...})
    .form({
        ...
        onAborting: function(proceed) {
            proceed();
        },
        onAborted: function() {
            window.location = "index.html";
        },
        onSaved: function(saveInfo) {
            if(saveInfo.isAuthenticated) {
                window.location = "cases.html";
            } else {
                window.location = "saveReceipt.html?refId=" + saveInfo.externalRefId;
            }
        },
        onSubmitted: function(receipt) {
            window.location = "submitReceipt.html?refId=" + receipt.refId;
        }
    });
```

###### `onAborting`
`onAborting` blir kalt når utfyller trykker avbryt og før selve avbrytelsen blir utført. Det er valgfritt å registrere en eventhandler til dette eventet. Du kan bruke eventet til å vise en "vil du virkelig avbryte"-dialog. Dersom utfyller bekrefter avbryting, kaller du `proceed()` eller `proceed(true)`. Dersom utfyller ikke ønsker å avbryte, kaller du `proceed(false)`. 

Eksempelet nedenfor tar utgangspunkt i at du har laget en funksjon `showAbortDialog()` som viser et bekreftelses-vindu. Avhengig av om utfyller klikker ja eller nei, kalles `proceed()` med true eller false.

```javascript
viewer
    .init({...})
    .form({
        ...
        onAborting: function(proceed) {
            showAbortConfirmationDialog() //må implementeres av deg
                .on("click", "#yes",
                    function() {
                        proceed(true);
                    })
                .on("click", "#no",
                    function() {
                        proceed(false);
                    });            
        },        
    });
```

###### `onAborted`
`onAborted` blir kalt etter at avbrytelsen er fullført. Her har du eksempelvis mulighet for å laste en ny URL i nettleseren.  

```javascript
viewer
    .init({...})
    .form({
        ...
        onAborted: function() {
            window.location = "index.html";
        },        
    });
```

###### `onSaved`
`onSaved` blir kalt etter at en mellomlagrring er fullført. Her har du eksempelvis mulighet for å laste en ny URL i nettleseren. 

```javascript
viewer
    .init({...})
    .form({
        ...
        onSaved: function() {
            window.location = "cases.html";
        }
    });
```

###### `onSubmitted`
`onSubmitted` blir kalt etter at en innsending er fullført. Her har du eksempelvis mulighet for å laste en ny URL i nettleseren. Funksjonen får et `receipt`-objekt. Dette objektet inneholder referanse id for innsendingen i `receipt.refId`. Dette kan du eksempelvis bruke for å laste en side som viser info om innsendelsen. Se avsnittet for `getCaseInfo()` nedenfor.

Eksempelet nedenfor laster en ny side med referanse id i querystring. 

```javascript
viewer
    .init({...})
    .form({
        ...
        onSubmitted: function(receipt) {
            window.location = "submitReceipt.html?refId=" + receipt.refId;
        }
    });
```

###### `onStarting`
`onStarting` blir kalt _før_ viewer starter opp et skjema, enten det er et nytt skjema eller gjenopptaking av eksisterende skjema. Eventet kan benyttes til å eksempelvis å sjekke at brukeren har akseptert personvernerklæring. Det er ikke obligatorisk å registere event-handler på eventet. 

```javascript
viewer
    .init({...})
    .form({
        ...
        onStarting: function() {
		    let readAndAcceptedPrivacy = sessionStorage.getItem("readAndAcceptedPrivacy"); //sjekk om bruker har godkjent personvernerklæring
			
			if(!readAndAcceptedPrivacy) {
				showPrivacyConfirmation() //må implementeres av deg
			}				
        },        
    });
```


### `getCaseInfo()` - uthenting av info for kvittering
Etter innsending av skjema er det vanlig å vise en side som viser skjemaets navn, tidspunkt for innsendelse og referanse id. Denne informasjonen hentes via kall til `viewer.getCaseInfo(refId)`. `refId` får du fra `onSubmitted()`-eventet når skjemaet sendes inn. 

Eksempelet nedenfor viser hvordan referanse id, skjemaets navn og tidspunkt for innsendelse hentes.

```javascript
<script>
    document.addEventListener("DOMContentLoaded", () => {
        const refId = viewer.util.getQuery("refId");

        viewer
            .init(...)
            .getCaseInfo(refId)
                .then(info => {
                    document.getElementById("receipt").innerHTML = 
                    `
                        refId: ${info.refId} <br/>
                        name: ${info.name} <br/>
                        updatedOn: ${info.updatedOn} <br/>
                    `;
                }).catch((error) => {
                    console.log(error);
                });
        });
</script>

<div id="receipt"></div>
```

### `session()`
viewer.js benytter HTML 5 sessionStorage for lagring av skjemadata under utfylling. `.session()` gjør det mulig å fortelle viewer.js etter hvor lang inaktivitet disse skal fjernes fra sessionStorage. `.init()` må kalles før `.session()`.

```javascript
viewer
    .init({...})
    .session({...})
```

#### session object:
Eksempelet nedenfor setter opp 20 minutter som inaktiv periode, med varsling 5 minutter før disse 20 minuttene nåes. `onIdleTimeoutWarning()` blir da kalt et gitt antall minutter før sesjonen utgår. Eventet gjør det mulig å vise en dialog der bruker kan forlenge eller avslutte sesjonen sin. Kall `extendSession()` dersom sesjonen skal forlenges, eller `endSession()` dersom sesjonen skal avsluttes. Det er valgfritt å registrere en handler på `onIdleTimeoutWarning`. Dersom 20 minutter med inaktivitet passerer, eller utfyller i `onIdleTimeoutWarning` velger å avslutte sesjonen, blir `onIdleTimeout` kalt. I `onIdleTimeout` må du kalle `viewer.logOut()` og sende med _absolutt_ URL til siden som brukeren skal sendes til _etter_ at utlogging er fullført. Det er viktig at URL-en er absolutt (ikke relativ). Det er vanlig å sende brukeren til en side som bekrefter at vedkommende er utlogget. 

```javascript
viewer
    .init({...})
    .session({
        idleTimeoutMinutes: 20,
        warningTimeoutMinutes: 5,
        onIdleTimeoutWarning: function(extendSession, endSession, sessionOptions) {
            showTimeoutWarning(sessionOptions.warningTimeoutMinutes) //må implementeres av deg
                .on("click", "#yes", function () { 
                    extendSession();
                })
                .on("click", "#no", function () {
                    endSession();
                });
        },
        onIdleTimeout: function() {
            viewer.logOut('https://yourhost/logoutConfirmation.html'); //replace with URL of you choice, the URL must be ABSOLUTE!
        }
    })
```

`sessionOptions.warningTimeoutMinutes` viser hvor mange minutter det er igjen til sesjonen avsluttes på det tidspunktet da funksjonen ble kalt. Dersom du skal lage en nedtellingsfunksjon, kan du ta utgangspunkt i denne verdien.


|Egenskap|Eksempel|Obligatorisk|Beskrivelse|
|--------|--------|------------|-----------|
|idleTimeoutMinutes  |60|Nei, default er 20|Antall minutter med inaktivitet før skjemadata fjernes.|
|warningTimeoutMinutes|5|Nei|Utfyller blir gitt en advarsel om at sesjonen vil tømmes dette antall minutter i forveien.|

### Brukerhåndtering
Dersom utfyller ikke er pålogget, men gjør en handling i viewer.js som krever pålogging, vil edoc-api sende brukeren til en identity-provider for pålogging. Handlinger i viewer.js som krever pålogging er mellomlagring av utkast og gjenopptaking av utkast, samt oppstart av skjema der det er satt krav til sikkerhetsnivå. 

Eksempelet nedenfor viser hvordan du kan få tak i informasjon om bruker som blir logget på av viewer.js:

```javascript
viewer
    .init({...})
    .session({
        ...
        onAuthenticated: (getUser) => {
            getUser() //if you need to get the user's id, invoke getUser.
                .then((user) => {
                    alert(`You are logged in as ${user.id}!`); //do something with user.id
                });
        }        
    })
```
Du kan også gjøre pålogging eksplisitt via viewer.js. viewer.js benytter edoc-api som backend. Edoc-api støtter pålogging med [ID-porten](http://eid.difi.no/nb/id-porten), [Feide](https://www.feide.no/) og AD. ID-porten og Feide er single-sign-on løsninger. Det betyr at dersom du allerede har en backend integrert mot eksempelvis ID-porten, vil en pålogging i din backend-løsning automatisk sørge for at brukeren er pålogget i edoc-api. Du kan benytte funksjonaliteten i viewer.js for å utføre operasjoner som innlogging, utlogging og uthenting av brukerinformasjon uavhengig av om du har en eksisterende backend integrert mot noen identity-providere.

Å være innlogget i edoc-api via en single-sign-on løsning innebærer to ting:
1. Brukeren har fått utstedt et [JWT-token](https://jwt.io/) av edoc-api. Tokenet er levert som en session-cookie. Cookien er httpOnly og kan ikke leses av JavaScript. Cookien er secure og sendes bare over HTTPS. 
2. Brukeren har fått utstedt en session-cookie av single-sign-on provideren (e.g. ID-porten). 

Edoc-api er stateless. Det vedlikeholder ingen informasjon om brukerens pålogging. All nødvendig informasjon om brukerens identitet er lagret i JWT-tokenet. Tokenet inneholder en hash slik at informasjonen i tokenet ikke kan modifiseres. 

JWT-tokenet inneholder følgende informasjon om den påloggede brukeren, her representert som et JSON objekt:

```javascript
{
    "id": "<fødselsnummer>",
    "securityLevel": 3,
    "displayName": "Per Ove Olsen",
    "firstName": "Per Ove",
    "lastName": "Olsen",
    "eMail": "per.ove@olsen.no",
    "mobilePhone": "12312312",
    "culture": "nb",
    "idleTimeoutMinutes": 20,
    "identityProvider": "MinID",
}
```

JWT-tokenet har begrenset varighet og fornyes hver gang det gjøres et kall til edoc-api. Varigheten bestemmes av en innstilling i edoc-apiet og er per default 20 minutter. JWT tokenet sendes kun med til edoc-api-domenet den ble utstedet av. Edoc-api sender CORS-headere slik at kun kjente og godkjente domener får innhold fra edoc-api. 

Merk at identity-provideren ID-porten returnerer brukerens fødselsnummer som id. Edoc-api benytter _ikke_ brukerens fødselsnummer som identifikator internt, men en enveis-hash (SHA-512) av et saltet fødselsnummer. Løsningen unngår på den måten å lagre fødselsnummeret. 

Her kommer noen eksempler på hvordan du kan benytte single-sign-on funksjonaliteten i viewer.js og edoc-api:

#### Innlogging
Etter å ha gjort `viewer.init()` med apiUrl og customerId, kan du gjøre innlogging med kallet `viewer.logIn()`. SecurityLevel settes til 3 eller 4, avhengig av hvilket [sikkerhetsnivå](http://eid.difi.no/nb/sikkerhet-og-informasjonskapsler/ulike-sikkerhetsniva) du krever at brukeren logger seg inn med. Kallet til `viewer.logIn()` vil resultere i at brukeren blir videresendt til en annen URL via `window.location.href`.

```javascript
viewer.init({
    apiUrl: "https://api.preprod.skjema.no",
    customerId: "ummo"
});

viewer.logIn({
    securityLevel: 3,
    returnUrl: "https://yoursite.com/yourLandingPage"
});
```

Dersom brukeren ikke er innlogget i edoc-api, vil edoc-api sende brukeren til single-sign-on provider (e.g. ID-porten). Dersom brukeren er innlogget hos single-sign-on-provider, vil brukeren bli sendt tilbake til edoc-api med informasjon om pålogget bruker. Edoc-api vil utstede JWT-token i session cookie og sende brukeren tilbake til den URL-en `viewer.logIn()`-kallet ble gjort fra. Dersom du ønsker at bruker skal bli sendt til en annen URL etter pålogging, kan dette spesifiseres slik:

```javascript
viewer.logIn({
    securityLevel: 3,
    returnUrl: "https://yoursite.com/yourPage"
});
```

Det er viktig at returnUrl er absolutt (ikke relativ). 

Dersom du ønsker å gjøre pålogging med Feide, må du oppgi provider slik:

```javascript
viewer.logIn({
    provider: "feide"
});
```

#### Sjekke om bruker er pålogget, hente info om pålogget bruker 
For å sjekke om bruker er pålogget og/eller hente info om pålogget bruker, kan du kalle `viewer.getAuthenticatedUser()`. Dette kallet sender JWT-tokenet til edoc-api. Edoc-api vil validere tokenet og returnere informasjonen i tokenet som et JSON-objekt. 

Merk at `viewer.getAuthenticatedUser()` er asynkron og returnerer et [Promise-objekt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises). Dersom brukeren ikke er logget inn resolver promiset null. Dersom brukeren er pålogget resolver promiset et JSON-objekt.

Merk også at `viewer.getAuthenticatedUser()` kun indikerer om brukeren har en lokal pålogging hos edoc-api. Det vil si: det kan være at brukeren har en single-sign-on-session hos identity provider (e.g ID-porten) uten at brukeren har en lokal sesjon hos edoc-api. 

```javascript
viewer.init({
    apiUrl: "https://api.preprod.skjema.no",
    customerId: "ummo"
});

const user = await viewer.getAuthenticatedUser();
```

Dersom brukeren er logget inn (i.e. har et gyldig JWT-token i en session cookie), vil promise-objektet resolve et JSON-objekt som ser slik ut:
```javascript
{
    "id": "<fødselsnummer>",
    "securityLevel": 3,
    "displayName": "Per Ove Olsen",
    "firstName": "Per Ove",
    "lastName": "Olsen",
    "eMail": "per.ove@olsen.no",
    "mobilePhone": "12312312",
    "culture": "nb",
    "idleTimeoutMinutes": 20,
    "identityProvider": "MinID",
}
```

#### Logge ut bruker
Du kan logge brukeren ut ved å kalle `viewer.logOut()`. 

Edoc-api er som nevnt stateless. Det ligger ingen informasjon om brukerens pålogging på serveren. 

En utlogging fra edoc-api innebærer følgende:
1. Brukerens session cookie med JWT-token utstedt av edoc-api slettes. 
2. Eventuell informasjon lagret av viewer.js i [sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage) i nettleseren slettes. viewer.js bruker prefikset "___edoc:" på verdier den lagrer i sessionStorage.
3. Brukerens session cookie utstedt av single-sign-on-provideren slettes. 
4. Brukerens session cookie hos alle andre aktører brukeren har vært innom som en del av single-sign-on sesjonen slettes. 

Argumentet til `viewer.logOut()` angir hvilken URL brukeren skal sendes til _etter_ at utlogging er fullført. Det er vanlig å vise en side som bekrefter at vedkommende er utlogget. Det er viktig at URL-en er absolutt (ikke relativ). Hvis du ikke oppgir noen URL, vil brukeren bli sendt til samme URL som kallet til `viewer.logOut()` ble gjort fra. Kallet til `viewer.logOut()` vil resultere i at brukeren blir videresendt til en annen URL via `window.location.href`. 

```javascript
viewer.init({
    apiUrl: "https://api.preprod.skjema.no",
    customerId: "ummo"
});

viewer.logOut("https://yoursite.com/logoutConfirmation");
```

Du kan gjøre kallet til `viewer.logOut()` selv om brukeren ikke er innlogget. 


### `getForms()`
`viewer.getForms()` returnerer tilgjengelige skjema for kunden. Eksempelet nedenfor viser hvordan skjema kan hentes og listes ut på en HTML-side.

```html
<!DOCTYPE html>
<html>
<body>
    <script>
        document.addEventListener("DOMContentLoaded", () => {
            setLoaderVisibility(true);

            viewer
                .init(...)
                .getForms()
                    .then(forms => {
                        const createFormLink = (form) => {
                            var fragment = document.createDocumentFragment();
                            const formLink = document.createElement("a");
                            formLink.setAttribute("href", `form.html?formId=${form.formId}`);
                            formLink.text = `${form.name}`;

                            fragment.appendChild(formLink);
                            const br = document.createElement("br");
                            fragment.appendChild(br);
                            return fragment;
                        }

                        const formsElement = document.getElementById("forms");

                        if(forms) {
                            forms.forEach(function(form) {
                                formsElement.appendChild(createFormLink(form));
                            });
                        } else {
                            formsElement.appendChild(document.createTextNode("No forms"));
                        }
                    });
        });
    </script>    

    <div id="forms"></div>

    <script src="viewer.min.js"></script>
</body>
</html>
```

### `getCases()` og `deleteCase()`

`viewer.getCases()` returnerer mellomlagrede utkast for en utfyller. Eksempelet nedenfor viser hvordan utkast kan hentes og listes ut på en HTML-side. Eksempelet viser også hvordan utkastene kan slettes via `viewer.deleteCase()`.

```html
<!DOCTYPE html>
<html>
<body>
    <script>
        document.addEventListener("DOMContentLoaded", () => {
            viewer.init(...);

            const renderCases = () => 
                viewer.getCases().then(cases => {
                    document.getElementById("content").style.display = "block";

                    const createCaseLink = (c, isSaved) => {
                        var fragment = document.createElement("div");
                        fragment.setAttribute("id", c.refId);

                        const caseLink = document.createElement("a");
                        if(isSaved) {
                            caseLink.setAttribute("href", `form.html?refId=${c.refId}`);
                        }
                        caseLink.text = `${c.name}`;
                        fragment.appendChild(caseLink);

                        const deleteLink = document.createElement("a");
                        deleteLink.setAttribute("href", "#");
                        deleteLink.text = " delete";
                        deleteLink.onclick = function(event) {
                            viewer.deleteCase(c.refId)
                            .then(() => {
                                document.getElementById(c.refId).remove();
                            }).catch((error) => {
                                alert(JSON.stringify(error));
                            });
                        };
                        fragment.appendChild(deleteLink);

                        const br = document.createElement("br");
                        fragment.appendChild(br);

                        return fragment;
                    }

                    const savedCasesElement = document.getElementById("savedCases");
                    savedCasesElement.innerHTML = "";

                    if(cases.saved) {
                        cases.saved.forEach(function(c) {
                            savedCasesElement.appendChild(createCaseLink(c, true));
                        });
                    } else {
                        savedCasesElement.appendChild(document.createTextNode("No saved cases"));
                    }

                    const submittedCasesElement = document.getElementById("submittedCases");
                    submittedCasesElement.innerHTML = "";

                    if(cases.submitted) {
                        cases.submitted.forEach( (c) => {
                            submittedCasesElement.appendChild(createCaseLink(c, false));
                        });
                    } else {
                        submittedCasesElement.appendChild(document.createTextNode("No submitted cases"));
                    }
                });

            renderCases();
        });
    </script>    

    <div id="content">
        <h2>Saved cases</h1>
        <div id="savedCases"></div>

        <h2>Submitted cases</h1>
        <div id="submittedCases"></div>
    </div>

    <script src="viewer.min.js"></script>
</body>
</html>
```

### Progresjonsvisning og feilhåndtering
Alle viewer metoder som gjør et kall til edoc-api returnerer et `Promise`-objekt. Fra det tidspunktet metoden blir kalt, og til operasjonen er fullført, kan det være lurt å vise en progresjons-indikator. Eksempelet nedenfor viser hvordan det kan gjøres.

```html
<!DOCTYPE html>
<html>
<body>
    <script>
        function setLoaderVisibility (isVisible) {
            const loader = document.getElementById("loader");

            if(loader) {
                loader.style.display = isVisible ? "block" : "none";
            }
        };

        document.addEventListener("DOMContentLoaded", () => {
            setLoaderVisibility(true);
            
            viewer
                .init(...)
                .form(...)
                .then(() => {
                    setLoaderVisibility(false);
                }).catch((error) => {
                    alert(error);
                }).finally(() => {
                    setLoaderVisibility(false);
                });
        });
    </script>

    <style>
        .loader {
            border: 16px solid #f3f3f3; /* Light grey */
            border-top: 16px solid #3498db; /* Blue */
            border-radius: 50%;
            width: 120px;
            height: 120px;
            animation: spin 2s linear infinite;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>

    <div id="viewer"></div>
    <div id="loader" class="loader"></div>

    <script src="viewer.min.js"></script>
</body>
</html>
```

## Innhold

### Viewer

- Babel Polyfill
- Handlebars
- jQuery
- jQuery UI
- DropZone
- i18next
- mathjs
- moment.js
- opn
- urijs 

### Styles

- Bootstrap 3.3.7
- Budicons