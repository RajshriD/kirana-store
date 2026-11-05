# Kirana Counter

Fast billing for a kirana shop. It works with no internet, checks expiry dates before selling, keeps an udhaar (credit) ledger, shows a UPI QR on the bill and sends the invoice on WhatsApp. The shopkeeper can speak the customer's list in English, Hindi or Telugu.

An optional cloud backup (this repo's `api/`) keeps a copy of the shop online, so a lost phone is not a lost shop, and a second phone can use the same data.

## What is inside

```
public/            the app (plain HTML, CSS and JavaScript, no build step)
  index.html, style.css, app.js     screens
  core.js                           billing, stock and expiry rules (also used by the tests)
  sw.js, manifest.webmanifest       lets the app open offline and be added to the home screen
  qrcode.js                         UPI QR drawing (MIT licence, Kazuhiko Arase)
api/sync.js        the one backend function: account, login, backup, restore
lib/store.js       database access (Neon Postgres, or memory for tests)
tests/             44 automatic checks
scripts/dev.js     run everything on your own computer without a database
vercel.json        Vercel settings
```

## Put it online (GitHub + Vercel + Neon), about 15 minutes

1. **GitHub.** Create a new empty repository. Unzip this project and upload everything in it (or run `git init`, `git add .`, `git commit`, `git push`). The files must sit at the top of the repository, not inside an extra folder.
2. **Vercel.** On vercel.com choose Add New, then Project, then import the repository. Leave the settings as they are (`vercel.json` already tells Vercel what to do) and press Deploy. The app is now live, and billing already works.
3. **Database (for cloud backup).** In the Vercel project open Storage, then Create Database, choose Neon, and connect it to this project. Vercel adds the `DATABASE_URL` setting by itself. Then open Deployments and Redeploy once.
4. **Check.** Open `https://YOUR-PROJECT.vercel.app/api/sync` in a browser. You should see `"database":true`.
5. **Use it.** Open `https://YOUR-PROJECT.vercel.app` in Chrome on the phone. Open the menu and choose Add to Home screen. In the app open More, then Cloud backup, then Create account. Write down the shop code and PIN.

Good to know:
- **Voice typing** works on the Vercel address (https) in Chrome or Edge. It needs internet. Billing itself never does.
- **Region.** For shops in India, set the function region to Mumbai (Vercel project Settings, then Functions) and choose a Neon region close to India when you create the database.
- **Optional settings** (Vercel project Settings, then Environment Variables): `INVITE_CODE` (only people who know it can create a cloud account) and `SESSION_SECRET` (any long random text; if empty the database address is used).
- **Vercel's free Hobby plan is meant for personal, non-commercial use.** If you sell this to shops, check Vercel's current terms and plans.

## Run it on your own computer

```
npm install
npm run dev      # http://localhost:3000, cloud data is kept in memory only
npm test         # 44 checks
```

## How the cloud backup works

- The phone is always the main copy. Every bill is saved on the phone first. When there is internet, the change is sent to the server a few seconds later.
- A second phone logs in with the shop code and PIN and downloads the shop.
- If two phones change the shop at the same time, the server refuses the second save. The app then asks the shopkeeper which copy to keep. Nothing is merged silently, so stock numbers never get mixed up. For the smoothest result, use one phone for billing and the other for looking things up.
- The PIN is 6 to 12 digits and is stored hashed (scrypt). Five wrong tries lock the shop for 15 minutes. Shop data is stored as readable JSON in your own Neon database.

## Limits to plan for

- Whole-shop data must stay under about 4 MB for cloud backup. That is many thousands of bills. Use Copy backup under More to keep a yearly copy.
- No GST breakdown yet. Prices are final prices.
- Barcodes work with a USB or Bluetooth scanner that types into the search box. Camera scanning is not included.
- The login has no password reset. If the PIN is lost, the data on the phone is still there, and you can create a new cloud account from it.
