# Workshop dashboard: instructions for Claude

This repo is a participant's copy of the Exagrow workshop starter, "Building a Live Data
Quality Dashboard with Claude Code." The person you are working with may never have written
code. They steer; you build. Follow every rule below.

## How to work with this person

- Explain what you are about to do in one or two plain sentences before you do it, and what
  you did afterwards. No jargon without a short definition the first time it appears.
- Keep changes small: one visible step at a time, so they can see the dashboard grow and so
  any mistake is easy to undo.
- When something fails, say what failed in plain words and what you will try next.
- Never do anything that costs money or changes an account setting without asking first.

## Writing style: no em dashes

Never use em dashes. Not in chat, code comments, commit messages, page text, chart labels or
documentation. An en dash or a doubled hyphen is not a substitute. Rewrite the sentence with
a full stop, colon, semicolon, comma or parentheses instead. If you find an em dash in a file
you touch, fix it in the same edit.

## Branches: `dev` and `prod`

This project has two branches, and each one is a live website:

- **`dev`** is where all work happens. Its site is the test copy.
- **`prod`** is the real, shared site. It only ever receives work that already ran on `dev`.

**First-time setup.** If the repo has no `prod` branch yet, create it from `dev` and push it:

```bash
git checkout dev
git branch prod dev
git push -u origin dev
git push -u origin prod
```

**Every change:**

1. Work on `dev`. No pull requests and no feature branches: commit straight to `dev`.
2. Commit after every change that works, with a short message saying what changed.
3. **Push after every commit**, immediately: `git push origin dev`. The push is the backup.
   Work that exists only on this laptop can be lost; work on GitHub cannot.
4. Read the push output. If it did not say the branch moved, it did not push; fix it before
   doing anything else. Never use `git push -q`, which hides failures.
5. Never force push, and never rewrite history that has been pushed.

**Promoting to `prod`** happens only when the person says so, in so many words ("ship it to
prod", "promote", "publish the real site"). Then:

```bash
git checkout prod
git merge --ff-only dev
git push origin prod
git checkout dev
```

If the fast-forward fails, stop and explain; never merge or reset `prod` to force it. Never
commit to `prod` directly.

## Secrets: the API key

The data source needs an API key (an app token). Treat it like a password.

- **Locally** it lives in `.env`, which git ignores. Copy `.env.example` to `.env` and have
  the person paste the token in themselves. Do not ask them to paste it into chat, and never
  print it, log it, or echo it back.
- **On Netlify** it lives in the site's environment variables (Site configuration, then
  Environment variables), set by the person in the Netlify dashboard.
- **Never in code, never in git, never in the browser.** Anything shipped to the browser can
  be read by anyone who opens the page, so the key is used only inside a Netlify Function
  (below), which runs on Netlify's servers.
- Before every commit, check `git status` and the diff for anything that looks like a key.
  If a key is ever committed or pushed, stop, tell the person plainly, and help them create a
  new token and delete the old one. Removing it from the latest commit is not enough.

## The stack

Keep it small and readable. The person should be able to open a file and roughly follow it.

- **Vite** with plain JavaScript (no React, no TypeScript) for the page.
- **Observable Plot** for charts.
- **A Netlify Function** in `netlify/functions/` that holds the API key, calls the data API,
  and returns only the JSON the page needs. The page calls `/.netlify/functions/<name>`.
- **`netlify.toml`** with the build command (`npm run build`), the publish folder (`dist`) and
  the functions folder.
- Run locally with `npx netlify dev`, which serves the page and the functions together and
  reads `.env`.
- Node.js LTS must be installed. If `node --version` fails, help the person install it from
  nodejs.org before anything else.
- Add a dependency only when it earns its place, and say why.

## Hosting: Netlify

- The person signs up at netlify.com with their GitHub account and imports this repo (Add new
  project, then Import an existing project).
- Set the **production branch to `prod`**, and turn on **branch deploys for `dev`**. Then the
  `dev` branch has its own test URL and `prod` is the real site.
- After each push, check that the Netlify deploy succeeded and that the change actually works
  on the deployed URL. A green deploy is not a working site.

## The data: NYC Open Data

- Source: NYC Open Data (data.cityofnewyork.us), which serves NYC Taxi and Limousine
  Commission (TLC) datasets through the Socrata SODA API. The app token goes in the
  `X-App-Token` request header.
- Find datasets with the catalog API, for example
  `https://api.us.socrata.com/api/catalog/v1?domains=data.cityofnewyork.us&q=taxi`, and
  confirm with the person which one to use.
- **Let the API do the heavy lifting.** Use SoQL (`$select`, `$where`, `$group`, `$order`)
  to ask for counts and summaries, and always set a `$limit`. Never pull a whole dataset into
  the browser or into a function.
- Cache results in the function response (a `Cache-Control` header) so a page refresh does
  not re-query the API.
- **Do not download TLC's raw trip files** (the parquet files on TLC's CloudFront site). A
  room of people pulling them at once from one network gets that network blocked. If the
  API is unavailable, use the mirror link the instructor gives out.

## What the dashboard is for

A data quality dashboard shows the data and how far to trust it, side by side:

- A few headline numbers (KPI tiles) and a chart or two of the trend.
- **Quality checks**, each named by what it tests: completeness (missing values), validity
  (values out of range), timeliness (how old the newest record is), consistency and
  uniqueness. Show each as a count and a rate, not just pass or fail.
- A table the person can drill into to see example records behind a number.
- A line at the top of the page that says where the data comes from and how current it is.
  Keep that line true.
