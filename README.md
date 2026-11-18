# EduChain Nexus

A GitHub-and-LinkedIn-style innovation hub for high school students, teachers, alumni and partners. Django 5, server-rendered pages, Chart.js.

## Run it
```
pip install -r requirements.txt
python manage.py makemigrations accounts hub
python manage.py migrate   # if you ran this before, run makemigrations again: registration and apps add new fields and tables
python manage.py seed_demo      # synthetic people and projects; sign in as admin or any username, password nexus-demo
python manage.py runserver
```
Change the demo password, or skip `seed_demo`, before real use. All pages need sign-in because they show student work.

## What is in it
- **Explore**: search projects by category, language and sort; cards or list. Project pages with README, commits, issues, pull requests, forks, stars, contributors.
- **Portfolios**: innovation score ring, collaboration, leadership and code-quality scores, a 26-week contribution calendar, awards, alumni endorsements, and four generated resume versions (CV, university, internship, scholarship) that print to PDF.
- **Ledger**: awards, project ownership (with first-publication date) and endorsements are sealed in a hash-linked chain. Anyone signed in can check a record code, and the page re-verifies the whole chain.
- **Talent**: students ranked per field (software, AI, cybersecurity, data, robotics, UI/UX, cloud, blockchain).
- **Schools**: rankings (table, cards, bars) and a page per school with rule-based suggestions.
- **Alumni**: directory, mentoring requests, endorsements.
- **Calendar**: month grid or list, category filter, event registration, `.ics` export for Google Calendar and Outlook.
- **Marketplace**: listings with licence and interest forms.
- **Insights**: growth in languages and categories, with plain-English sentences such as "Python projects increased by 42% this academic year."
- **For you**: projects, mentors and events matched to your interests, each with the reason.
- **Registration**: anyone can register at `/register/` (student, teacher, alumni, recruiter or university). The account stays switched off until an administrator approves it on the **Registrations** page (or in the Django admin). People waiting or refused are told so when they try to sign in, and get an email when decided (printed to the console until you set `DJANGO_EMAIL_BACKEND`). There is a honeypot, strong-password checks and a 5-per-hour limit per connection.
- **Apps**: approved members upload a zip of an application to their profile. Other signed-in members can browse its files, read text files, edit a text file (saved as a new version; the old one is kept and the owner can restore it) and download any version. Owners can switch edits off for their app ("view and download only"), remove it, or upload new versions. Edits to the same file at the same time are refused rather than overwritten.
- **Data hub** (administrators): upload and download CSV for every dataset, with preview and confirm.

## The innovation score
The nine inputs and weights are exactly those in the brief. Raw counts cannot be added fairly, so each input is first scaled to 0-100 against a "full marks" value (`engine/scoring.py`, `FULL_MARKS`), then weighted. The result is 0-100. Collaboration, leadership and code-quality scores are transparent formulas in the same file. Code quality is a proxy (README, tests, steady commits, issue hygiene), not a static analyser.

## Scope: what this is and is not
Built and working in Django: everything listed above.
Deliberately simplified (the brief names these, and they need external services or accounts):
- **Blockchain**: a permissioned hash-chained ledger in the database. It gives tamper evidence and ordering, not a public network. `hub.services.anchor` is the one place to add a Polygon/Ethereum/Fabric transaction.
- **Git**: commits, issues and pull requests are records in the database, not a real git server. No file hosting or cloning yet.
- **AI**: talent ranking and recommendations use fixed rules and cosine similarity on tags. No XGBoost/CatBoost/LightGBM/neural models, and no AI code review, because those need training data and API keys.
- **Not included**: Spark/Hadoop/Kafka/Elasticsearch, JWT/OAuth/Microsoft Entra/Google sign-in/MFA, REST API, messaging, forums, live rooms, video, Teams/Zoom/Meet, Azure/Kubernetes manifests, GDPR/COPPA tooling beyond sign-in-only access and an audit log.

## Uploaded zips: how they are kept safe
Nothing from a zip is ever extracted to disk or executed on the server. Uploads are checked for: zip-slip paths (`../`, absolute, drive letters, backslashes), symbolic links, passwords, duplicate names, more than 2,000 files, more than 25 MB compressed, 200 MB expanded, 20 MB per file, and extreme compression ratios (zip bombs). Files are stored under generated names outside any public folder and served only to signed-in members as downloads (`Content-Disposition: attachment`, `nosniff`). Text previews are escaped, never rendered. Each owner has 20 apps, 40 versions per app and 500 MB.
**Not done:** the contents are not virus-scanned. The pages say so, and you should add a scanner (for example ClamAV) at `hub.appstore._store` before opening this to the public. Uploaded files live in `NEXUS_MEDIA_ROOT` (default `./media`): put it on a persistent, backed-up disk in production.

## CSV import
Order: schools, students, projects, activity, awards, events, alumni, listings. Each file is fully validated and previewed with line-by-line errors; nothing is written until you confirm, and the write is all-or-nothing. Re-uploading updates existing rows. Downloads are UTF-8 with a BOM for Excel and neutralise spreadsheet formulas. Imported accounts have no password: set one in the Django admin. Imported and admin-created accounts are approved automatically; only self-registration needs approval.

## Tests
`python -m pytest tests` runs the engine and CSV tests with no Django needed, plus end-to-end tests when Django is installed.

