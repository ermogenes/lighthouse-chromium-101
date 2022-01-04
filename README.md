# lighthouse-chromium-101
Experiment with Lighthouse running on Chromium using GitHub Actions

## Auditing

Start a production server, and then run Lighthouse CLI with Chrome headless option.

It can be done using `wait-on` (to wait for building and serving) and `npm-run-all` (to run building and auditing in parallel).

In `package.json`, using a Vue 3 project:

- `serve:prod` build production code and serves in HTTPS mode at `https://localhost:8080`;
- `lh` waits 30 seconds to run Lighthouse headless, ignoring HTTPS, and outputing reports to `./audit`;
- `audit` run both scripts in parallel.

```json
  "scripts": {
    "serve:prod": "vue-cli-service serve --mode production --https",
    "lh": "wait-on https://localhost:8080 --httpTimeout 30000 && lighthouse https://localhost:8080 --chrome-flags='--ignore-certificate-errors' --chrome-flags='--headless' --output-path=./audit/lighthouse --output json --output html --output csv",
    "audit": "npm-run-all -p -n -r serve:prod lh"
  },
```

## With GitHub Actions

Create a workflow that runs the `audit` npm script. Use `actions/upload-artifact@v2` to upload the resulting reports to the action artifacts, and retains 90 days.

In `.github\workflows\lighthouse-audit.yml`:

```yml
name: Audit with Lighthouse
on:
  push:
    branches:
      - main
jobs:
  lighthouse-audit:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout 🛎️
        uses: actions/checkout@v2.3.1

      - name: Audit 🩺
        run: |
          cd lc101
          npm install
          npm run audit

      - name: Archive reports 🗃
        uses: actions/upload-artifact@v2
        with:
          name: lighthouse-reports
          retention-days: 90
          path: |
            lc101/audit
```

## Using environments without Chrome or Chromium

You can install `chromium` as dev dependency, and use `cross-env` to set `CHROME_PATH` to something like `node_modules/chromium/lib/chromium/chrome-win` (or the correct path for your env) before run Lighthouse.