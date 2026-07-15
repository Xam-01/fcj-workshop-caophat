---
title : "Prerequisite"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

# PART 1 - CODE PREPARATION (on local machine)

## 1A. BACKEND - Modify to run on Lambda

The current Backend is a long-running Express app (`src/server.ts` calls `app.listen()`). We need to separate the app initialization from the `listen` logic, and then add a handler file for Lambda. **The business logic (routers, controllers, models) remains unchanged.**

### Step 1A.1 - Install bridge package

```bash
cd smart-menu-api
npm i @codegenie/serverless-express
```

### Step 1A.2 - Split `src/server.ts` into `app.ts` + `server.ts`

Currently, `src/server.ts` both initializes the app (helmet, cors, compression, rate-limit, mounts `/api/v1` router, error handler) and calls `app.listen()`. Let's split it:

**Create `src/app.ts`** - Move the entire app initialization part from `server.ts` here, omit `app.listen()`, and add `export default app`:

```ts
// src/app.ts
import express from 'express';
import helmet from 'helmet';
import cors from 'cors';
import compression from 'compression';
import cookieParser from 'cookie-parser';
import morgan from 'morgan';
// ...import routers, error handler middleware as currently imported in server.ts

const app = express();

// REQUIRED when behind CloudFront: for rate-limiter to read correct client IP
// and secure cookies to receive correct protocol via X-Forwarded-* header
app.set('trust proxy', 1);

app.use(helmet());
app.use(cors({ origin: true, credentials: true })); // see CORS notes below
app.use(compression());
app.use(express.json());
app.use(cookieParser());
app.use(morgan('tiny'));

// Mount router as current (keep prefix /api/v1)
app.use('/api/v1', /* your rootRouter from src/router/v1 */);

// Global error handler remains (placed at the end)
app.use(/* your errorHandler middleware */);

export default app;
```

**Modify `src/server.ts`** - Solely for local execution:

```ts
// src/server.ts
import app from './app';
import { connectDB } from './lib/db'; // real path to your Mongo connection file in src/lib

const PORT = Number(process.env.PORT) || 3000;

connectDB()
  .then(() => {
    app.listen(PORT, () => {
      console.log(`SmartMenu API (local) running on http://localhost:${PORT}/api/v1`);
    });
  })
  .catch((err) => {
    console.error('Failed to start server:', err);
    process.exit(1);
  });
```

> Note: `connectDB` is the existing MongoDB connection function in your `src/lib/`. If the name/path is different, adjust the import. If the old code connected to MongoDB directly inside `server.ts`, extract it into `src/lib/db.ts` as described in Step 1A.3.

### Step 1A.3 - Cache MongoDB connection (most fragile part on Lambda)

Lambda reuses containers between requests → **do not** call `mongoose.connect()` on every request. Connection must be cached at the module scope.

**Modify/Create `src/lib/db.ts`:**

```ts
// src/lib/db.ts
import mongoose from 'mongoose';

let cached: typeof mongoose | null = null;

export async function connectDB() {
  // If connected and socket is active, reuse the connection
  if (cached && mongoose.connection.readyState === 1) {
    return cached;
  }
  cached = await mongoose.connect(process.env.MONGOOSE_URL as string, {
    serverSelectionTimeoutMS: 5000,
    maxPoolSize: 5, // Lambda: keep pool small to avoid exhausting Atlas connections
  });
  return cached;
}
```

> Use **MongoDB Atlas** (Mongo does not run on Lambda). In Atlas → **Network Access** → add `0.0.0.0/0` (simple for MVP; restrict with VPC/NAT later if needed).

### Step 1A.4 - Create Lambda Entry: `src/lambda.ts`

This is the **only new file** written for Lambda:

```ts
// src/lambda.ts
import serverlessExpress from '@codegenie/serverless-express';
import app from './app';
import { connectDB } from './lib/db';

let cachedServer: ReturnType<typeof serverlessExpress> | null = null;

export const handler = async (event: any, context: any) => {
  // Do not wait for empty event loop - otherwise Lambda hangs because Mongo socket is still open
  context.callbackWaitsForEmptyEventLoop = false;

  await connectDB();

  if (!cachedServer) {
    cachedServer = serverlessExpress({ app });
  }
  return cachedServer(event, context);
};
```

### Step 1A.5 - Modify Refresh Token Cookie for Production

Your BE sets the refresh token via httpOnly cookie (endpoints: `/auth/login`, `/auth/refresh-token`, `/auth/logout`). Since FE and API share the **same domain** via CloudFront (same-site), you only need to adjust the cookie setting (usually in auth controller `src/controller/v1/`):

```ts
res.cookie('refreshToken', refreshToken, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production', // production requires HTTPS
  sameSite: 'lax',                                // same domain → 'lax' is sufficient
  path: '/api/v1/auth',                           // matches the auth route path
  maxAge: 7 * 24 * 60 * 60 * 1000,
});
```

> Since it is same-domain, **no need** for `sameSite: 'none'`. Only use `'none'` if you separate FE and BE into different domains later (which requires `secure: true`).

### Step 1A.6 - CORS Notes

Same domain via CloudFront ⇒ requests are same-origin ⇒ CORS is rarely triggered. Keep `cors({ origin: true, credentials: true })` for safety (does no harm). No need to whitelist specific domains.

### Step 1A.7 - Verify `tsconfig.json`

Ensure it contains:
```json
{
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src",
    "module": "commonjs",
    "target": "ES2020",
    "esModuleInterop": true
  }
}
```
> The handler on Lambda will be `dist/lambda.handler`, so `lambda.ts` must build to `dist/lambda.js`. If you use path aliases (`tsconfig-paths`), consider building with `esbuild`/`tsc` so that aliases are resolved inside `dist` (see the note at the end of the section).

### Step 1A.8 - Build + Zip Package

```bash
npm run build            # tsc → generates dist/ (must contain dist/lambda.js)

# Reinstall only production dependencies to minimize zip size
rm -rf node_modules
npm ci --omit=dev

# Pack
zip -r function.zip dist node_modules package.json
```

> End of 1A: you have **`function.zip`** - containing `dist/`, `node_modules` (prod), `package.json`. This is the complete BE artifact ready for Lambda upload in Part 2.

> ⚠️ If the project uses `tsconfig-paths` (alias `@/...`): `tsc` **does not** rewrite aliases in the output. Two solutions:
> 1. Build using esbuild into a single bundled file: `npx esbuild src/lambda.ts --bundle --platform=node --target=node20 --outfile=dist/lambda.js` then zip `dist` + `package.json` (no need for `node_modules` if bundled) - cleaner and fastest cold start.
> 2. Or add `tsconfig-paths/register` to the runtime - more complex on Lambda. The esbuild approach is highly recommended.

---

## 1B. FRONTEND - Build 3 Static Apps

The FE builds into 3 apps using Vite modes. Since all share the same domain, APIs are called using the relative path `/api/v1`.

### Step 1B.1 - Create Production Env File

Create `.env.production` (automatically loaded by Vite for production builds). The domain will be the CloudFront domain - but since it doesn't exist yet, leave it placeholder and **come back to fill it in after getting the CloudFront domain in Part 3** (then rebuild). Or fill it now if you already have the domain.

```
# API on the same domain → use relative path
VITE_API_BASE_URL=/api/v1

# Origin of each app - used by tablet to generate QR code pointing to the correct app, and cross-navigation
# Fill in real CloudFront domain after Part 3, e.g.:
VITE_LANDING_ORIGIN=https://dxxxx.cloudfront.net
VITE_OWNER_ORIGIN=https://dxxxx.cloudfront.net/owner
VITE_TABLET_ORIGIN=https://dxxxx.cloudfront.net/tablet
```

> ⚠️ Important: The tablet app generates **QR codes for each table** pointing to `VITE_TABLET_ORIGIN`. If incorrect (e.g., localhost), the printed QR code will not open. Must be the real production domain.
>
> ⚠️ Ensure **not** to enable `VITE_USE_MOCK_API=true` in production build (mock API is only for development). Leave it blank or `false`.

### Step 1B.2 - Build 3 Apps into 3 Separate Folders

`npm run build` builds the current mode by default. Build each mode into its own outDir:

```bash
cd smart-menu-fe
npm ci

npm run build -- --mode landing --outDir dist/landing --emptyOutDir
npm run build -- --mode owner   --outDir dist/owner   --emptyOutDir
npm run build -- --mode tablet  --outDir dist/tablet  --emptyOutDir
```

> If the `build` script in `package.json` has fixed mode/outDir configurations preventing these flags from taking effect, add 3 scripts to `package.json`:
> ```json
> "build:landing": "vite build --mode landing --outDir dist/landing --emptyOutDir",
> "build:owner":   "vite build --mode owner   --outDir dist/owner   --emptyOutDir",
> "build:tablet":  "vite build --mode tablet  --outDir dist/tablet  --emptyOutDir"
> ```
> then run `npm run build:landing && npm run build:owner && npm run build:tablet`.

The result should be 3 folders, each representing a complete SPA:
```
dist/landing/index.html + assets/
dist/owner/index.html   + assets/
dist/tablet/index.html  + assets/
```

> ⚠️ Base path issue: when owner/tablet run in sub-paths (`/owner/`, `/tablet/`), Vite needs to know the `base` to load `assets/` correctly. Add to `vite.config.ts`:
> ```ts
> // vite.config.ts
> export default defineConfig(({ mode }) => ({
>   base:
>     mode === 'owner' ? '/owner/' :
>     mode === 'tablet' ? '/tablet/' :
>     '/',
>   // ...remaining config
> }));
> ```
> Without setting `base`, the sub-apps will show a blank page because assets are requested from the wrong path.

> End of 1B: you have **3 dist folders** ready to be uploaded to S3.

---

## 1C. AWS S3 BUCKET - Create Bucket for Menu Images

To enable the application to display and upload menu images, we need to prepare an S3 Bucket with public read access.

### Step 1C.1 - Create S3 Bucket for Assets
1. Access the AWS Console, search for and select **S3**.
2. Click the **Create bucket** button.
   ![S3 List](/images/5-Workshop/5.2-Prerequisite/s3_list.png)
3. Configure Bucket Information:
   - **Bucket name:** Enter a globally unique name, e.g., `smartmenu-assets-2026`.
   - **AWS Region:** Select **Asia Pacific (Singapore) ap-southeast-1** (or your preferred region).
   ![S3 Create Settings](/images/5-Workshop/5.2-Prerequisite/s3_create.png)
4. Configure Public Access Settings:
   - Scroll down to the **Block Public Access settings for this bucket** section.
   - Uncheck **Block all public access**.
   - Check the acknowledgement box: **I acknowledge that the current settings might result in this bucket and the objects within becoming public**.
   ![S3 Public Access](/images/5-Workshop/5.2-Prerequisite/s3_public_access.png)
5. Click **Create bucket at the bottom of the page. You will see a success message.
   ![S3 Created](/images/5-Workshop/5.2-Prerequisite/s3_created.png)

### Step 1C.2 - Configure Bucket Policy (Allow Public Read)
1. Select your newly created bucket, go to the **Permissions** tab.
2. Scroll down to the **Bucket policy** section, click **Edit**.
   ![S3 Policy Edit](/images/5-Workshop/5.2-Prerequisite/s3_policy_edit.png)
3. Paste the following JSON policy into the Policy Editor to allow public read access (replace `smartmenu-assets-2026` with your actual bucket name):
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "PublicReadGetObject",
               "Effect": "Allow",
               "Principal": "*",
               "Action": "s3:GetObject",
               "Resource": "arn:aws:s3:::smartmenu-assets-2026/*"
           }
       ]
   }
   ```
   ![S3 Policy JSON](/images/5-Workshop/5.2-Prerequisite/s3_policy_json.png)
4. Click **Save changes**. The bucket status will now display as **Public**.
   ![S3 Policy Success](/images/5-Workshop/5.2-Prerequisite/s3_policy_success.png)