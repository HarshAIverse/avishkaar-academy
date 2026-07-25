# Avishkaar Academy — Website

A premium, conversion-focused website for Avishkaar Academy (Vikhroli West, Mumbai), covering Class 5th–12th, JEE, NEET, MHT-CET, NDA and Merchant Navy (IMU-CET) coaching.

```
avishkaar-academy/
├── frontend/          Static site (HTML/CSS/JS) — no build step needed
│   ├── index.html
│   ├── css/style.css
│   ├── js/script.js
│   └── assets/images/ (your uploaded logo, toppers, banners)
└── backend/           Express API for the inquiry form
    ├── server.js
    ├── package.json
    ├── .env.example
    └── data/inquiries.json   (leads get stored here)
```

## How it fits together

The Express backend serves the `frontend` folder as static files **and** exposes the
`/api/inquiry` endpoint, so the whole site runs as a single app on one port. You don't
need a separate frontend server.

## Run it locally

```bash
cd backend
npm install
cp .env.example .env      # adjust PORT / ADMIN_TOKEN if you like
npm start
```

Visit **http://localhost:4000** — that's the live site, form included.

## The inquiry form, end to end

1. A visitor submits the "Take the First Step Towards Success" form.
2. The frontend POSTs JSON to `POST /api/inquiry`.
3. The backend validates the fields (name length, 10-digit mobile, valid email if
   provided, course must be one of the dropdown options) and rejects bad submissions
   with a clear message.
4. Valid leads are appended to `backend/data/inquiries.json` with a timestamp and the
   submitter's IP.
5. The form shows a success message and resets.

Submissions are rate-limited to 20 per IP per 15 minutes to deter spam/bot abuse.

### Viewing leads

```
GET /api/inquiries
Authorization: Bearer <ADMIN_TOKEN from .env>
```

Returns all stored inquiries, most recent first. Treat this token like a password —
change it in `.env` before going live, and keep `.env` out of version control.

### Wiring up real notifications

Right now leads are written to a JSON file only. To get an email/SMS/CRM ping the
moment someone submits, add your integration (e.g. Nodemailer, Twilio, a CRM webhook)
inside the `POST /api/inquiry` handler in `backend/server.js`, right after the lead is
saved — there's a comment marking the hook point.

For anything beyond light traffic, swap the JSON file for a real database (Postgres,
MongoDB, etc.) — the `readInquiries` / `writeInquiries` functions are the only place
that needs to change.

## Editing content

- **Copy & sections** — edit `frontend/index.html` directly; every section from the
  brief (Hero, Wall of Fame, Programs, Why Us, Faculty, Testimonials, Gallery, Inquiry,
  Contact, Footer) is in there with HTML comments marking each block.
- **Colors / type / spacing** — all design tokens live at the top of
  `frontend/css/style.css` under `:root`.
- **Images** — drop new files into `frontend/assets/images/` and update the `src`
  attributes. The current images were optimized from your uploads (resized + compressed
  for fast loading); do the same for anything new before adding it.
- **Faculty photos** — the Faculty section currently uses initials in place of photos
  (none were provided). Replace the `.faculty-card__avatar` divs in `index.html` with
  `<img>` tags once you have real photographs.
- **WhatsApp / phone numbers** — search for `918434879737` and `9702928223` in
  `index.html` and swap in your numbers.
- **Google Map** — the embed in the Contact section currently geocodes the Vikhroli
  address by search query. For pixel-perfect placement, generate an embed URL from
  Google Maps' own "Share → Embed a map" for your exact location and swap the `src`.

## Deploying

Any Node host works (Render, Railway, a VPS, etc.):

1. Push this repo (or just the `backend/` and `frontend/` folders) to your host.
2. Set environment variables `PORT` (if required by the host) and `ADMIN_TOKEN`.
3. Build command: `npm install --prefix backend`
4. Start command: `node backend/server.js`

If you'd rather host the frontend separately (e.g. on a CDN) and the backend elsewhere,
set `window.AVISHKAAR_API_BASE = "https://your-api-domain.com"` in a small inline
`<script>` before `js/script.js` loads in `index.html`, and enable CORS for your
frontend's domain in `backend/server.js` (currently open to all origins for convenience
— restrict this before going live).

## Pre-launch checklist

- [ ] Replace placeholder email `admissions@avishkaaracademy.com` with your real inbox
- [ ] Replace social links (Instagram/Facebook/YouTube `#` placeholders) in the footer
      and Contact section
- [ ] Add real faculty photographs
- [ ] Set a strong `ADMIN_TOKEN` in production
- [ ] Restrict CORS to your real domain
- [ ] Swap the Google Maps embed for your exact pinned location
- [ ] Point a real domain at the deployed app and set up HTTPS
