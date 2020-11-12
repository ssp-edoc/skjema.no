# edoc-api

Edoc-api er et stateless REST API. 

## Autentisering

### Utstedelse av JWT-token
edpc-api bruker et JWT token for å identifisere brukeren. Dette tokenet kan utstedet av edoc-api via pålogging med ID-porten eller Feide. Tokenet kan også utstedes av en annen applikasjon. Tokenet må legges i en cookie som sendes med ved kall til edoc-api. Nedenfor beskrives nødvendige trinn for å generere tokenet.

#### Installer pakke 

```
Install-Package eDocument.Core.Web.Token 
```

#### Genererer SHA256-nøkkel

For å generere et token som aksepteres av edoc-api, må du få oppgitt den symmetriske SHA256-nøkkelen som edoc-api installasjonen er konfigurert med. Denne nøkkelen er SVÆRT PRIVAT og SENSITIV og skal ikke deles med uvedkommende. 


Hvis du ønsker å generere en SHA256-nøkkel til test-formål, kan du benytte powershell:
```
PS > $hmac = New-Object System.Security.Cryptography.HMACSHA256
PS > [Convert]::ToBase64String($hmac.Key)
AQihRq+hH4IYRYclfDNOQ80isVYHhs5AUFQic7VWHsQmvUVfyJnvQLNkRoS3e+FeiAVFyR36rh8xXf3zs2apBA==
```

#### Opprett token

Gitt at du har en nøkkel og info om en bruker, kan du nå generere et JWT token slik:
```
var secretKey = "HEktn2swBv02pLmZEEL1cMtnie2QCReebAWwXmWFa8vPXkeD2vIRXwqXY7brJ6gz/PCXnPR2Pg48Co0L+pSBIw==";

var tokenString = eDocument.Core.Web.JwtToken.CreateUserToken(
    friendlyUsername: "Ole Olsen",
    actualUniqueUsername: "DA0BFF42-3EAD-41F2-BBF9-97F53BB2B09E",
    idleTimeoutMinutes: 20,
    securityLevel: 3,
    key: secretKey
);

Console.WriteLine(tokenString); //eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1bmlxdWVfbmFtZSI6IkRBMEJGRjQyLTNF...
```

Merk at edoc-api kun vil akseptere dette tokenet dersom riktig SHA256-nøkkel er benyttet til å generere tokenet.

#### Opprett token med custom claims

Dersom du har egne "claims" eller info du ønsker å legge inn i tokenet for senere uthenting, kan du legge disse inn i tokenet på følgende måte:

```

var customClaims = new Dictionary<string, string>();

customClaims["foo"] = "bar";
customClaims["bah"] = "baz";

var tokenString = eDocument.Core.Web.JwtToken.CreateUserToken(
    friendlyUsername: "Ole Olsen",
    actualUniqueUsername: "DA0BFF42-3EAD-41F2-BBF9-97F53BB2B09E",
    idleTimeoutMinutes: 20,
    securityLevel: 3,
    key: secretKey,
    claims: customClaims
);

```

Når du senere får tak i `ApiUser`, eksempelvis i en preutfylling, vil du kunne hente ut informasjonen igjen slik:

```

var apiUserPrincipal = HttpContext.Current.User as ApiUser.ApiUserPrincipal;
var fooValue = apiUserPrincipal.ApiUser.PropertyBag["foo"]; //bar
var bahValue = apiUserPrincipal.ApiUser.PropertyBag["bah"]; //baz

```

#### Pakk tokenet inn i en cookie

Tokenet må leveres som en cookie av applikasjonen din. Hvordan dette gjøres, avhenger av applikasjonen din (e.g. .NET standard/framework/WebForms/Web API). Nedenfor er et eksempel. Viktige punkter:

1. Cookie-navn må være "token"
2. IKKE sett cookie-expiration. Cookien skal være en såkalt session/transient cookie. Hvis du setter expiration, vil cookien ikke bli borte selv om brukeren lukker nettleseren. 
3. Sett riktig domene. Edoc-api og applikasjonen som utsteder cookien må ha samme topp-nivå-domene, og det er dette som skal settes her. Dersom din applikasjon kjører på www.mysite.com og edoc-api kjører på www.forms.mysite.com, må altså ".mysite.com" benyttes som domene på cookien.
4. HttpOnly må settes til true. Dette er for å unngå at JavaScript som er lastet på siden kan lese og stjele cookie-informasjonen.
5. Secure må settes til true slik at cookie-informasjonen ikke kan stjeles underveis.

```
var cookieName = "token";
var jwtToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1bmlxdWVfbmFtZSI6IkRBMEJGRjQyLTNF...";

var cookie = new System.Web.HttpCookie(cookieName, jwtToken)
{
    Domain = domain,
    Path = "/",
    HttpOnly = true,
    Secure = true,
};

HttpContext.Current.Response.SetCookie(cookie);
```

For å logge brukeren ut, må du fjerne token-cookien, eller overskrive den med en cookie som har en blank verdi (string.Empty). 

For å passe på at tokenet er gyldig så lenge brukeren er aktiv i løsningen, vil edoc-api "refreshe" tokenet hver gang det gjøres en operasjon mot edoc-api. Det vil si at edoc-api vil lese informasjonen som er satt i tokenet, og generere et nytt token med den samme informasjonen som er gyldig i en ny periode (default 20 minutter, men kan konfigureres i edoc-api). Dersom brukeren din forventes å oppholde seg i lengre tid i applikasjonen din, uten å være innom edoc-api, kan det være relevant at applikasjonen din passer på å refreshe tokenet før brukeren gjør en skjema-operasjon mot edoc-api, slik at brukeren har et gyldig token når vedkommende gjør en forespørsel mot edoc-api. 
