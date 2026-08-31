# hugo-demo

A minimal demo project: a [Hugo](https://gohugo.io/) site, hand-written theme,
locally vendored Bootstrap for a responsive layout, deployed automatically to
**GitHub Pages** via **GitHub Actions**.

## Structure

- `content/` — Markdown content: Home, About, News (list + posts), Signup.
- `layouts/` — Hand-written templates. No external theme.
- `static/css`, `static/js` — Vendored Bootstrap 5 (CSS + JS bundle), copied
  from the npm package, no CDN dependency.
- `.github/workflows/hugo.yaml` — Builds the site with Hugo and publishes it
  to GitHub Pages on every push to `main`.

## Local development

```sh
hugo server -D
```

Then open <http://localhost:1313/>. `-D` includes draft content
(`draft: true` in front matter), which is useful while writing new posts.

To produce a production build locally (same as what CI does):

```sh
hugo --minify
```

Output goes to `public/` (git-ignored).

## First-time GitHub setup

1. Create the repo on GitHub (e.g. `tonvanbart/hugo-demo`), then push this
   project:

   ```sh
   cd hugo-demo
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin git@github.com:tonvanbart/hugo-demo.git
   git push -u origin main
   ```

2. In the GitHub repo, go to **Settings → Pages** and under "Build and
   deployment", set **Source** to **GitHub Actions**. (This is a one-time
   manual step — Pages doesn't build with Actions until you tell it to.)

3. Push to `main` (or re-run the workflow manually from the **Actions** tab)
   and the site will build and deploy. The URL will be
   `https://tonvanbart.github.io/hugo-demo/`.

The workflow asks GitHub Pages for the correct base URL at build time
(via `actions/configure-pages`), so it doesn't matter if you rename the repo
or fork it — you don't need to hand-edit `baseURL` in `hugo.toml` for it to
work in CI. That value in `hugo.toml` is only used for local reference /
`hugo server`.

## Signup page / serverless form backend

The **Signup** page (`content/signup.md`, `layouts/_default/signup.html`)
posts JSON to whatever URL is set in `params.formEndpoint` in `hugo.toml`.
By default that's empty, so the form runs in a harmless "no backend
configured" mode — it validates input but shows a warning instead of
actually submitting, which is fine for exploring the site without any setup.

To wire it up to a real backend using **Google Apps Script + Google Sheets**:

1. Create a new Google Sheet. Add a header row, e.g. `Timestamp | Name | Email`.
2. In the Sheet, go to **Extensions → Apps Script**.
3. Replace the default code with something like:

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

4. Click **Deploy → New deployment**, type **Web app**, set "Who has access"
   to **Anyone**, and deploy. Copy the generated `/exec` URL.
5. Put that URL in `hugo.toml`:

   ```toml
   [params]
     formEndpoint = "https://script.google.com/macros/s/XXXXXXXX/exec"
   ```

6. Rebuild / push. Submissions from the Signup page will now land as new
   rows in the Sheet.

If venue wifi is unreliable during a live demo, it's safe to leave
`formEndpoint` empty and instead walk through the Apps Script code and a
Sheet with some rows already in it from an earlier test run.
