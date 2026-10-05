# hugo-demo

Een minimaal demoproject: een [Hugo](https://gohugo.io/)-site met een
zelfgeschreven thema en een lokaal meegeleverde kopie van Bootstrap voor een
responsieve opmaak. De site wordt via **GitHub Actions** automatisch
gepubliceerd op **GitHub Pages**.

## Structuur

- `content/` — inhoud in Markdown: Home, Over, Nieuws (overzicht en berichten)
  en Aanmelden.
- `layouts/` — zelfgeschreven templates, zonder extern thema.
- `static/css`, `static/js` — meegeleverde Bootstrap 5 (CSS en JS-bundel),
  gekopieerd uit het npm-pakket, zonder afhankelijkheid van een CDN.
- `.github/workflows/hugo.yaml` — bouwt de site met Hugo en publiceert die bij
  elke push naar `main` op GitHub Pages.

## Lokaal ontwikkelen

```sh
hugo server -D
```

Open daarna <http://localhost:1313/>. Met `-D` worden ook concepten
(`draft: true` in de front matter) getoond. Deze verschijnen niet op de produktie site.

Zo maak je lokaal een produktiebuild (hetzelfde als wat CI doet):

```sh
hugo --minify
```

De uitvoer komt in `public/` terecht (en wordt door git genegeerd).

## Eenmalige installatie op GitHub

1. Maak de repository aan op GitHub (bijvoorbeeld `tonvanbart/hugo-demo`) en
   push dit project:

   ```sh
   cd hugo-demo
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin git@github.com:tonvanbart/hugo-demo.git
   git push -u origin main
   ```

2. Ga in de repository op GitHub naar **Settings → Pages** en zet onder "Build
   and deployment" de **Source** op **GitHub Actions**. (Dit is een eenmalige
   handmatige stap: Pages bouwt pas met Actions als je dat hebt ingesteld.)

3. Push naar `main` (of start de workflow handmatig opnieuw vanaf het tabblad
   **Actions**). De site wordt dan gebouwd en gepubliceerd op
   `https://tonvanbart.github.io/hugo-demo/`.

De workflow vraagt tijdens de build de juiste basis-URL op bij GitHub Pages
(via `actions/configure-pages`). Het maakt dus niet uit of je de repository
hernoemt of forkt: je hoeft `baseURL` in `hugo.toml` niet handmatig aan te
passen om het in CI te laten werken. Die waarde in `hugo.toml` dient alleen als
lokale referentie en voor `hugo server`.

## Aanmeldpagina en serverless formulierbackend

De pagina **Aanmelden** (`content/signup.md`, `layouts/_default/signup.html`)
stuurt JSON naar de URL die in `params.formEndpoint` in `hugo.toml` staat.
Standaard is die leeg. Het formulier werkt dan in een "demo" modus zonder
backend: de invoer wordt gecontroleerd, maar in plaats van te verzenden toont
het formulier een waarschuwing. Zo kun je de site zonder enige configuratie
verkennen.

Zo koppel je het formulier aan een echte backend met **Google Apps Script en
Google Sheets**:

1. Maak een nieuwe Google Sheet aan. Voeg een kopregel toe, bijvoorbeeld
   `Tijdstip | Naam | E-mail`.
2. Ga in de Sheet naar **Extensies → Apps Script**.
3. Vervang de standaardcode door iets als:

   ```javascript
   function doPost(e) {
     var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
     var data = JSON.parse(e.postData.contents);
     sheet.appendRow([new Date(), data.name, data.email]);
     return ContentService
       .createTextOutput(JSON.stringify({ status: "ok" }))
       .setMimeType(ContentService.MimeType.JSON);
   }
   ```

4. Klik op **Implementeren → Nieuwe implementatie**, kies het type
   **Web-app**, zet "Wie heeft toegang" op **Iedereen** en implementeer.
   Kopieer de gegenereerde `/exec`-URL.
5. Zet die URL in `hugo.toml`:

   ```toml
   [params]
     formEndpoint = "https://script.google.com/macros/s/XXXXXXXX/exec"
   ```

6. Bouw de site opnieuw of push je wijzigingen. Inzendingen vanaf de pagina
   Aanmelden komen nu als nieuwe rijen in de Sheet terecht.

