# ASPIER Platform

Production-grade academic, research, and academy platform.

## Local development

```bash
npm run install:all
npm run dev
```

Frontend: http://localhost:5173

Backend health: http://localhost:5000/api/v1/health

Administrator sign-in is restricted to `s2310956146@ru.ac.bd`. Set `ADMIN_PASSWORD` in `server/.env`; never put the password in tracked source or documentation. The development seed synchronizes that account from the configured fixed credentials.

Demo student: `student@aspier.org` / `Student@1234`

The server automatically uses an in-memory MongoDB instance when `MONGO_URI` is empty or unavailable in development. Cloudinary and Gmail are optional for general development mail; configure `SMTP_USER` and `SMTP_APP_PASSWORD` in `server/.env` to deliver the administrator's authenticator setup email to the fixed admin inbox.

## Admin content and membership workflow

- Manage Wings, Research, team profiles, News, and Academy courses under `/admin/resources/*`. Image upload controls provide slot-specific size recommendations, client-side size limits, and previews.
- The default research-writing course is seeded automatically. Edit course details and fee under **Admin → ASPIER Academy / Courses** and edit/reorder its syllabus under **Curriculum**.
- Edit site identity, homepage artwork, founder profile, payment numbers, and the BDT 1,000 initial membership fee under **Admin → Site & payment settings**.
- Applicants submit `/join`, keep the emailed application ID and private access code, and use `/join/portal` to check review status, complete profile/payment details after approval, and view the result.
- Review Step 1 and Step 2 immediately under **Admin → Membership applications**. Step 2 has no time-based review lock; approving a valid submission publishes its profile to the requested Team section.
- bKash and Nagad default to `01603012448`; both numbers and the membership fee can be changed by an administrator.
- The initial application collects the required Supervisor/Young Scholar role and one of the ten named wings. Stage 1 approval emails a one-time activation code; applicants can also view it in the secured applicant notification center. Member activation requires a unique `@gmail.com` address and a strong password.
- Admin notifications cover new applications, membership submissions, course payments, and contact inquiries. Admin and signed-in member notification bells refresh automatically and support read/unread tracking.
- Admin sign-in requires the fixed administrator credentials and authenticator-based TOTP. The first-time TOTP key and QR image are emailed only to the fixed administrator inbox and are never shown on the login screen. Admin TOTP cannot be disabled.
- Applicants can open the same sign-up form from the navigation **Sign Up** action or the notification bell. Administrators review submissions under **Admin → Membership applications**, assign the Application ID and Private Access Code, then approve and email the credentials. Applicant portal access requires authenticator verification.
- Research image uploads recommend 800 × 450 px (16:9), max 2 MB. Research, wing, course, and news card thumbnails use compact responsive image sizing; empty image slots remain hidden.

## Vercel deployment

Deploy the `aspier` workspace as one Vercel project:

1. In Vercel, set **Root Directory** to the directory containing this root `package.json` and `vercel.json` (the workspace root, not `client` or `server`). Do not override the commands in `vercel.json`: it runs `npm ci`, `npm run build`, and publishes `client/dist`.
2. The root `api/[...path].ts` function forwards `/api/*` requests to the Express application and connects to MongoDB once per warm function instance. The Vercel rewrite serves the built React app for browser routes. Leave `VITE_API_BASE_URL` unset to use same-origin `/api/v1`; this keeps cookie and CSRF behavior same-origin.
3. Configure these Vercel environment variables for every environment you deploy:
   - `ADMIN_PASSWORD`: fixed administrator password, at least 12 characters.
   - `MONGO_URI`: reachable MongoDB connection string (MongoDB Atlas is suitable); Vercel cannot use the development in-memory database.
   - `JWT_ACCESS_SECRET` and `JWT_REFRESH_SECRET`: different random secrets, each at least 32 characters.
   - `FIELD_ENC_KEY`: a persistent secret for encrypting protected fields; do not rotate it without migrating encrypted data.
   - `CLIENT_URL`: the exact deployed site origin, such as `https://your-project.vercel.app`, for CORS.
   - `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, and `CLOUDINARY_API_SECRET` for persistent uploads. Vercel function disk is ephemeral; do not use local uploads in production.
   - `SMTP_USER` and `SMTP_APP_PASSWORD` to enable email delivery, including the administrator authenticator setup email.
4. Seed the production database once from a trusted environment with the same production variables: `npm run seed --workspace @aspier/server`. Do not run seeding on every serverless request. Keep pulled environment files out of version control.

The development server remains unchanged: Vite proxies `/api` to localhost, while the production frontend and API share the Vercel origin.
