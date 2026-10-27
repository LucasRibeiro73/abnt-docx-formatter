# ABNT Formatter

A Flask web app that formats academic papers (`.docx`) according to **ABNT**, the Brazilian standard for academic documents. Users sign up, buy credit packs through **Stripe Checkout**, and spend one credit per formatted document.

## Features

- **Document formatting** with `python-docx`: page margins, fonts, heading levels, long quotations, abstract and references sections.
- **Authentication** with Flask-Login and hashed passwords (PBKDF2).
- **Credit-based billing**: Stripe Checkout sessions (card and boleto) and a **signed webhook** (`checkout.session.completed`) that adds credits to the user's account.
- **Safe file handling**: uploaded file names are sanitized and only `.docx` files are accepted.
- **Config via environment variables**: no secrets in the code.

## How it works

1. The user uploads a `.docx` file from the dashboard.
2. If they have credits, `formatador_abnt.py` walks through the document paragraphs, detects sections (abstract, numbered headings, quotations marked with `[CITAÇÃO]`, references) and applies the ABNT styles.
3. One credit is charged and the formatted file is offered for download.
4. Credits are bought on Stripe. Stripe calls `/stripe-webhook`, the signature is verified and the credits are added.

## Tech stack

Python 3.12 · Flask · Flask-Login · Flask-SQLAlchemy (SQLite) · python-docx · Stripe API

## Running locally

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt

cp .env.example .env        # then fill in your Stripe test keys
python app.py
```

The app runs at `http://localhost:5000`. To test the webhook locally, use the [Stripe CLI](https://docs.stripe.com/stripe-cli):

```bash
stripe listen --forward-to localhost:5000/stripe-webhook
```

## Project structure

```
app.py               # Routes: auth, dashboard, upload, Stripe checkout and webhook
formatador_abnt.py   # Document formatting rules
models.py            # User model (email, password hash, credits)
Templates/           # Jinja templates
static/              # CSS, images and the .docx template users can download
```

## Next steps

- Automated tests for the formatting rules and the webhook.
- Idempotent webhook handling (store processed Stripe event IDs).
- Move formatting to a background job and store files in object storage.
- Docker setup and CI.
