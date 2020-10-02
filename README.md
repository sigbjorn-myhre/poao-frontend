# pto-frontend
Node.js Express app som håndterer diverse funksjoner som er påkrevd av de fleste frontend applikasjoner.

## Hvordan ta i bruk
Kopier filene som skal serveres til /app/public.

```dockerfile
FROM docker.pkg.github.com/navikt/pto-frontend/pto-frontend:IMAGE_VERSION
COPY build /app/public
```

## Konfigurering
All konfigurering av pto-frontend gjøres gjennom miljø variabler.


### PORT
Setter hvilken port pto-frontend skal kjøre på. Default er 8080.

### SERVE_FROM_PATH
Setter hvor pto-frontend skal lete etter statiske filer. Default er /app/public.

### CONTEXT_PATH
Setter context path for alle paths i pto-frontend. Default er ingen context path.

### REDIRECT_ON_NOT_FOUND
Hvis satt til **true** så vil forespørsler som gir 404 bli redirectet til root path.
Hvis satt til noe annet, så vil 404 forespørsler defaulte til å returnere index.html.
Default er **true**.

REDIRECT_ON_NOT_FOUND=true
```
https://my-app.dev.nav.no/not/a/real/path -> Redirected to https://my-app.dev.nav.no
```

REDIRECT_ON_NOT_FOUND=false
```
https://my-app.dev.nav.no/not/a/real/path -> Serve index.html on this url
```

### NAV_DEKORATOR_URL
Hvis satt så vil pto-frontend sette opp et endepunkt på {CONTEXT_PATH}/dekorator som vil redirecte til URLen som er spesifisert.
Default er **ingen url**.

NAV_DEKORATOR_URL=https://dekoratoren.dev.nav.no
```
/dekorator/client.js -> redirected to https://dekoratoren.dev.nav.no/client.js
```

### ENABLE_FRONTEND_ENV
Hvis satt til **true** så vil pto-frontend lage en **env.js** som blir plassert i SERVE_FROM_PATH.
Denne filen vil inneholde alle miljø variabler som starter med PUBLIC og sette det på window.
Default er **false**.

F.eks hvis man har en variabel som heter PUBLIC_MY_APP_URL så vil dette produsere følgende **env.js** fil.

```js
window.env = {MY_APP_URL: 'https://my-app.dev.nav.no'};
```

Dette kan brukes med en script tag for å laste inn miljøvariabler før appen starter.

```html
<script src="{CONTEXT_PATH}/env.js"></script>
```

### ENFORCE_LOGIN
Hvis satt til **true** så vil pto-frontend validere tokenet til bruker for alle forespørsler. 
Default er **false**.

Hvis ENFORCE_LOGIN er på så må også variblene LOGIN_REDIRECT_URL, OIDC_DISCOVERY_URL, OIDC_CLIENT_ID, TOKEN_COOKIE_NAME settes.

#### LOGIN_REDIRECT_URL
Hvor skal brukeren sendes hvis de ikke har et gyldig token.
Denne URLen burde inneholde **{RETURN_TO_URL}** som vil bli byttet ut med URLen som brukeren var på før de ble sendt videre for innlogging.

Eks:
LOGIN_REDIRECT_URL=https://loginservice.dev.nav.no/login?redirect={RETURN_TO_URL}&level=Level4

#### OIDC_DISCOVERY_URL
URL som peker til discovery endepunktet for en OIDC provider. Brukes for å hente JWKS URI og issuer.

#### OIDC_CLIENT_ID
En klient id som brukes for å verifisere at tokenet kan brukes i appen din.

#### TOKEN_COOKIE_NAME
Navnet på cookien som inneholder tokenet til bruker.


## Eksempel på konfigurasjon

Her er noen eksempler på konfigurasjoner som kan bli brukt.

Konfigurasjon med NAV dekoratør proxy, miljø variabler i frontend og login med redirect for eksterne brukere i testmiljøet.
```
NAV_DEKORATOR_URL=https://dekoratoren.dev.nav.no
ENABLE_FRONTEND_ENV=true
ENFORCE_LOGIN=true
LOGIN_REDIRECT_URL=https://loginservice.dev.nav.no/login?redirect={RETURN_TO_URL}&level=Level4
OIDC_DISCOVERY_URL=https://login.microsoftonline.com/NAVtestB2C.onmicrosoft.com/v2.0/.well-known/openid-configuration?p=B2C_1A_idporten_ver1
OIDC_CLIENT_ID=0090b6e1-ffcc-4c37-bc21-049f7d1f0fe5
TOKEN_COOKIE_NAME=selvbetjening-idtoken
```

Konfigurasjon med NAV dekoratør proxy, miljø variabler i frontend og login med redirect for eksterne brukere i produkjson.
```
NAV_DEKORATOR_URL=https://www.nav.no/dekoratoren
ENABLE_FRONTEND_ENV=true
ENFORCE_LOGIN=true
LOGIN_REDIRECT_URL=https://loginservice.nav.no/login?redirect={RETURN_TO_URL}&level=Level4
OIDC_DISCOVERY_URL=https://login.microsoftonline.com/navnob2c.onmicrosoft.com/v2.0/.well-known/openid-configuration?p=B2C_1A_idporten
OIDC_CLIENT_ID=45104d6a-f5bc-4e8c-b352-4bbfc9381f25
TOKEN_COOKIE_NAME=selvbetjening-idtoken
```

Konfigurasjon med context path for micro frontends.
```
CONTEXT_PATH=/my-app
```