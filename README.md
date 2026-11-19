# FitTrack — Gym Management

A responsive gym-management application using the charcoal + coral visual direction from the supplied mobile UI reference. The same React frontend runs as a web app and can be packaged as an Android app with Capacitor.

## Included features

- Admin registration (first admin only) and JWT login
- Member profiles: name, address, primary phone, WhatsApp/alternate phone, email, age, gender, plan, joining date, expiry date, fee and notes
- Member search, active/expired filtering, edit, delete and CSV export
- Dashboard KPIs, membership expiry watchlist and revenue snapshot
- Payment ledger with amount, date, payment mode, type and transaction reference
- Daily attendance check-in with duplicate prevention and expired-membership guard
- Reports for membership status, collections and attendance
- Membership expiry notifications based on current member records
- Persistent dark and light color themes
- Meta WhatsApp Cloud API template-message endpoint and daily expiry reminder scheduler
- Local mode: if the API is unavailable, records are saved on this device only; the app starts with no sample records

## Stack

- Frontend: React 18, Vite, Tailwind CSS, Lucide, responsive CSS
- Mobile packaging: Capacitor 6 (Android)
- Backend: Node.js, Express, Mongoose, MongoDB
- Security: Helmet, CORS, rate limiting on authentication, bcrypt password hashing, JWT
- Messaging: Meta WhatsApp Cloud API

## Requirements

- Node.js 20 LTS recommended
- MongoDB 6+ (local or Atlas)
- Android Studio + Android SDK for Android builds

## Run locally

1. Install dependencies from the repository root:

   ```bash
   npm install
   ```

2. Configure the API:

   ```bash
   cp server/.env.example server/.env
   ```

   Set `MONGODB_URI` and a long random `JWT_SECRET`. Keep the `.env` file private.

3. Start MongoDB, then start frontend and backend:

   ```bash
   npm run dev
   ```

   - Web app: http://localhost:5173
   - API health: http://localhost:5000/api/health

4. On first launch, choose **Create the first admin account**. The API allows registration only while there are no users. After creating the admin, registration is automatically closed; use sign-in thereafter.

5. If the API is unavailable, the app opens in local mode with empty member, payment and attendance lists. Records created in local mode are saved only in that browser or device and are not shared with other devices.

## Android app (Capacitor)

The responsive React app is shared by web and Android. From the repository root:

```bash
npm run build
cd apps/web
npx cap add android
npx cap sync android
npx cap open android
```

Build and run the Android project from Android Studio. For a physical phone, set the API URL to a server reachable from the device. Do not use `localhost` for a remote phone. Configure `VITE_API_URL` before building, for example:

```bash
# apps/web/.env.production
VITE_API_URL=https://your-api-domain.example/api
```

Then run `npm run build` and `npx cap sync android` again.

## API overview

All routes except `/api/health` and `/api/auth/*` require `Authorization: Bearer <JWT>`.

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/health` | Health check |
| POST | `/api/auth/register` | Create the first admin |
| POST | `/api/auth/login` | Sign in |
| GET/POST | `/api/members` | List/create members |
| GET/PUT/DELETE | `/api/members/:id` | Read/update/delete member |
| GET/POST | `/api/payments` | List/create payments |
| DELETE | `/api/payments/:id` | Delete a payment |
| GET/POST | `/api/attendance` | List today's attendance / check in |
| GET | `/api/dashboard` | Dashboard aggregate metrics |
| GET | `/api/whatsapp/status` | WhatsApp configuration status |
| POST | `/api/whatsapp/remind/:memberId` | Send a template reminder |

## WhatsApp Cloud API setup

1. Create/configure a Meta app with the WhatsApp product and a WhatsApp Business phone number.
2. Set `WHATSAPP_TOKEN`, `WHATSAPP_PHONE_NUMBER_ID`, `WHATSAPP_API_VERSION`, `WHATSAPP_TEMPLATE_NAME` and `WHATSAPP_LANGUAGE` in `server/.env`.
3. Create and get approval for a WhatsApp message template. The reminder scheduler sends template parameters `[member name, days remaining]`; make sure your approved template has matching body placeholders and adjust `server/src/services/reminders.js` if your template differs.
4. The scheduler checks daily at 09:00 India time for expiry dates in `REMINDER_DAYS` (default `7,3,1,0`). A reminder is marked as sent only after Meta accepts it.

WhatsApp delivery requires Meta approval, valid credentials, opt-in/consent from members, and compliance with WhatsApp Business messaging rules. The app does not bypass template or consent requirements.

## Production checklist

- Use MongoDB Atlas or a managed MongoDB deployment and automated backups.
- Set a strong unique `JWT_SECRET`, `NODE_ENV=production`, and exact `CLIENT_ORIGIN`.
- Serve the API over HTTPS; keep secrets in the hosting provider's secret manager.
- Add role/permission management before creating staff accounts; current first release uses admin authentication as the gate for all operational routes.
- Add a formal privacy notice, member consent, retention/deletion policy and access audit log before storing real member data.
- Configure a proper transactional email/SMS/WhatsApp provider and monitor scheduler failures.
- The overview revenue chart is calculated from recorded payments. It remains empty until payments are added.
