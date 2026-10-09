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

1. Maak de repository aan op GitHub (bijvoorbeeld `gebruikernaam/hugo-demo`) en
   push dit project:

   ```sh
   cd hugo-demo
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin git@github.com:gebruikernaam/hugo-demo.git
   git push -u origin main
   ```

2. Ga in de repository op GitHub naar **Settings → Pages** en zet onder "Build
   and deployment" de **Source** op **GitHub Actions**. (Dit is een eenmalige
   handmatige stap: Pages bouwt pas met Actions als je dat hebt ingesteld.)

3. Push naar `main` (of start de workflow handmatig opnieuw vanaf het tabblad
   **Actions**). De site wordt dan gebouwd en gepubliceerd op
   `https://gebruikernaam.github.io/hugo-demo/`.

De workflow vraagt tijdens de build de juiste basis-URL op bij GitHub Pages
(via `actions/configure-pages`). Het maakt dus niet uit of je de repository
hernoemt of forkt: je hoeft `baseURL` in `hugo.toml` niet handmatig aan te
passen om het in CI te laten werken. Die waarde in `hugo.toml` dient alleen als
lokale referentie en voor `hugo server`.

## Aanmeldpagina en formulierverwerking met Formspree

De pagina **Aanmelden** (`content/signup-formspree.md`,
`layouts/_default/signup-formspree.html`) stuurt de formuliergegevens naar
[Formspree](https://formspree.io/), een externe dienst die formulierinzendingen
ontvangt en doorstuurt. Er is dus geen eigen server of backendcode nodig. De
URL waarnaar het formulier verstuurt staat in `params.formspreeEndpoint` in
`hugo.toml`. Als die leeg is, wordt de invoer wel gecontroleerd, maar in plaats
van te verzenden toont het formulier een waarschuwing.

Zo koppel je het formulier aan je eigen Formspree-account:

1. Maak een (gratis) account aan op <https://formspree.io/>.
2. Maak een nieuw formulier aan. Formspree geeft je dan een endpoint in de vorm
   `https://formspree.io/f/xxxxxxxx`.
3. Zet die URL in `hugo.toml`:

   ```toml
   [params]
     formspreeEndpoint = "https://formspree.io/f/xxxxxxxx"
   ```

4. Bouw de site opnieuw of push je wijzigingen. Inzendingen vanaf de pagina
   Aanmelden (naam, e-mailadres en de keuze voor vegetarisch) verschijnen nu in
   het Formspree-dashboard.

Het formulier verstuurt de gegevens met `fetch` en de header
`Accept: application/json`. Formspree antwoordt dan met JSON in plaats van door
te sturen naar een eigen bedankpagina, zodat de bevestiging of een eventuele
foutmelding gewoon op de pagina zelf verschijnt.

De eerdere variant met Google Apps Script en een Google Sheet
(`content/signup.md`, `layouts/_default/signup.html`, `params.formEndpoint`)
staat nog in het project, maar is niet meer opgenomen in het menu.
Dit is een voorbeeld van een serverless functie en gebruikt eigen geschreven code; 
deze heeft niet de limiet van Formspree (50 submits per maand voor een gratis account).
De code in het voorbeeld werkt, maar voor echt gebruik zou er wel wat hardening aan
toegevoegd moeten worden.
