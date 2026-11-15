# BIZNEXT DIGITAL COMPANY – Digital Business Platform

Your Digital Business. Your Future. Your Success.

Complete working website: homepage with 3 pricing plans, digital product store, customer login, customer dashboard, Razorpay payments (server-side verified), manual QR / UPI payments, automatic delivery, protected downloads, invoices, admin panel with analytics, WhatsApp support button (9376998999).

**Stack:** Node.js (Express) + PostgreSQL + plain HTML/CSS/JS frontend (Chart.js for charts). *Wallet / withdrawal / referral modules are intentionally not included (as requested).*

---

## 1. Folder structure

```
biznext/
├─ server.js            Backend (auth, payments, downloads, admin APIs)
├─ db.js                PostgreSQL connection
├─ schema.sql           Database schema + default data
├─ scripts/
│  ├─ init-db.js        Creates tables  (npm run db:init)
│  └─ create-admin.js   Creates admin    (npm run admin:create)
├─ public/              Website (HTML, CSS, JS)
│  ├─ index.html …      All pages
│  ├─ js/               app.js, home.js, store.js, auth.js, checkout.js, dashboard.js, admin.js
│  └─ uploads/          Public images (logo, QR, product images)
├─ storage/             PRIVATE files (product files, payment proofs) – never served directly
├─ logs/error.log       Error log
└─ .env.example
```

## 2. Installation (local)

Requirements: Node.js 18+ and PostgreSQL 13+.

```bash
cd biznext
npm install
cp .env.example .env          # then edit .env
createdb biznext              # or create the DB from pgAdmin / hosting panel
npm run db:init               # creates tables + 3 plans + categories + settings
npm run admin:create          # creates admin from ADMIN_* values in .env
npm start                     # open http://localhost:3000
```

Generate a strong `JWT_SECRET`:
`node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"`

## 3. Admin setup

1. Run `npm run admin:create` (uses `ADMIN_EMAIL` / `ADMIN_PASSWORD` from `.env`, min 10 chars). Or: `node scripts/create-admin.js "Your Name" you@mail.com "StrongPassword"`.
2. Login at `/login.html` – admins are sent to `/admin.html`.
3. In **Admin → QR & UPI**: upload your payment QR, set your UPI ID and instructions.
4. In **Admin → Pricing Plans**: check prices (₹1,999 / ₹4,999 / ₹9,999) and **edit the notes of the Complete plan** – domain validity, hosting validity and renewal charges. The defaults are placeholders; write your real terms.
5. In **Admin → Products**: add products (image + file). Tick *Reselling rights authorised* only if the product owner allows reselling. The 3 sample products are hidden – edit or delete them.
6. In **Admin → Website & Branding**: logo, company name, WhatsApp number.

Manual payments: customer pays by QR/UPI → submits UTR + screenshot → order shows in **Admin → Orders → To verify** → check your bank/UPI app → **Approve** (delivers) or **Reject**.

## 4. Razorpay setup (use TEST mode first)

1. Create a Razorpay account → Dashboard → **Test Mode** → Settings → API Keys → generate keys.
2. Put `RAZORPAY_KEY_ID` and `RAZORPAY_KEY_SECRET` in `.env`.
3. Dashboard → Settings → **Webhooks** → Add webhook:
   - URL: `https://YOUR-DOMAIN/api/webhooks/razorpay`
   - Secret: same as `RAZORPAY_WEBHOOK_SECRET` in `.env`
   - Events: `payment.captured`, `order.paid`, `payment.failed`
4. Test with Razorpay test UPI / cards. For local testing of the webhook use a tunnel (e.g. ngrok).
5. For live payments: complete Razorpay KYC/activation, generate **Live** keys, replace them in `.env`, create the webhook again in Live mode.

How payments are protected: price is always read from the database; after checkout we verify the Razorpay **signature**, fetch the payment from Razorpay to confirm status + amount, and only then mark the order paid. The webhook does the same independently. A product is never delivered from a browser-only "success" message. One payment ID can be used for only one order.

## 5. Email / OTP

Set `SMTP_*` in `.env` (any SMTP: Gmail app password, Brevo, Zoho, etc.) and `REQUIRE_EMAIL_VERIFY=true` to force email OTP verification at signup. Forgot-password always uses an emailed OTP. If SMTP is empty, emails/OTPs are printed in the server console (development only).

## 6. Deployment (VPS / Render / Railway)

1. Set `NODE_ENV=production`, `BASE_URL=https://yourdomain.com`, a strong `JWT_SECRET`, `DATABASE_URL` (+ `DB_SSL=true` for most hosted DBs).
2. Run `npm install --omit=dev`, `npm run db:init`, `npm run admin:create`, then `npm start` (use PM2 or systemd on a VPS: `pm2 start server.js --name biznext`).
3. Put Nginx / Caddy in front with HTTPS (Let's Encrypt). HTTPS is required for secure cookies and live payments.
4. **Persistent storage:** `public/uploads` and `storage/` must be on a persistent disk (most free hosts wipe the disk on redeploy). For large scale, move product files to S3 / Cloudflare R2 / Backblaze (only the `/api/download` route and the multer storage need changing).
5. **Backups:** schedule a daily `pg_dump biznext > backup.sql` and copy `storage/` and `public/uploads/` off the server.
6. Errors are written to `logs/error.log`.

## 7. Security included

Password hashing (bcrypt), JWT in httpOnly SameSite cookie, CSRF header check, role-based admin access, rate limiting (global + login/OTP/checkout), Helmet security headers/CSP, server-side validation, DB transactions with row locks for order fulfilment, webhook signature verification, private file storage, access-checked downloads, upload type/size limits.

## 8. Things you must do yourself

- Edit Privacy / Terms / Refund / Reselling policy pages with your real terms (they are templates) and have them reviewed.
- Make sure you really hold the **reselling rights** for every product you mark as resellable.
- Domain / hosting for the ₹9,999 plan and the setup work are delivered by you manually after an order is paid (order shows the customer's business/domain request).
- Refund button in admin only marks the order refunded and stops downloads – send the money back from Razorpay / your bank.
- Add GST details to invoices if you are GST registered (Admin → Website → Invoice footer note).

---

## 9. Adding E-books, AI Software and the AI Course (Admin)

Admin → **Products → + Add Product**. Choose the category (E-books, AI Software, Digital Courses…), set price, upload image, and either:
- upload the **product file** (PDF / ZIP / software), or
- paste a **course / delivery link** (private https link). Customers see it only after payment is verified.

The **AI Business Course (₹999)** is created hidden by `npm run db:init`. Add your course link or file in Admin → Products → edit it, then tick *Visible in store*. Until then the course page shows a WhatsApp waiting-list button.

If you already ran `db:init` earlier, run `npm run db:init` again (safe to repeat) to add the new column and the course.

New pages: `/ai-course.html` and `/businesses.html` (E-book, AI Course and Software Sales business models).
