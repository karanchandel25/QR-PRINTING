# QR Print website (Cloudflare Worker + R2 + KV)

## Features
- ₹10 per page, multiplied by copies
- CASH orders enter `READY_TO_PRINT` immediately
- ONLINE orders receive a UPI intent QR with the exact amount and order ID in its note
- Online order remains `PENDING_PAYMENT` until admin manually confirms actual incoming payment and clicks approve
- 20 MB upload limit enforced on the Worker
- Private uploaded files in R2; admin-only download endpoint
- Admin panel secured with `ADMIN_TOKEN`

## Important limitations
- The Worker supports the listed file extensions, but does **not** convert office documents to PDF or physically print files. `READY_TO_PRINT` means ready in the shop's queue, not an automatic printer command.
- PDF page count is read in the browser. For non-PDF files, customer enters page count; check it before printing. Image files count as one page.
- Do not approve based on QR display, screenshot, or UTR alone. Verify the actual credit and amount in your bank/UPI app.
- Use HTTPS and keep `ADMIN_TOKEN` private. For production, add Cloudflare Turnstile/rate limiting and set a strict origin policy if frontend and API are hosted on separate domains.

## Setup
1. Install Node.js and Wrangler: `npm install -g wrangler`
2. Login: `wrangler login`
3. Create KV namespace: `wrangler kv namespace create ORDERS`. Copy the returned ID into `wrangler.toml`.
4. Create R2 bucket: `wrangler r2 bucket create qr-print-files`
5. Copy `index.html` into `public/index.html`; copy `worker.js` into `src/worker.js`.
6. From project folder, set secret: `wrangler secret put ADMIN_TOKEN` and enter a long random password.
7. Deploy: `wrangler deploy`
8. Open the deployed Worker URL, then test a small CASH order and ONLINE test order.

## Project files
- `public/index.html`: frontend
- `src/worker.js`: API Worker
- `wrangler.toml`: bindings/config template

## Endpoints
- `POST /api/orders` — upload order
- `POST /api/orders/:id/utr` — submit UTR
- `GET /api/admin/orders` — list orders (admin bearer token)
- `GET /api/admin/orders/:id/file` — download original file (admin)
- `POST /api/admin/orders/:id/approve` — approve online order after manual verification
- `POST /api/admin/orders/:id/reject` — reject order (admin)

## Before going live
- Add a privacy notice and retention/deletion policy for uploaded customer files.
- Consider deleting uploaded files after printing.
- Configure Cloudflare rate limiting and bot protection.
- Verify the UPI ID is correct and test QR with your UPI app. A UPI intent QR only opens a payment request; it cannot confirm receipt.
