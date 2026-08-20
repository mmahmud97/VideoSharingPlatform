# Video Sharing Platform

A video sharing app: users sign in with Google, upload videos, and watch transcoded 360p playback.

The repo is a template. Cloud names are `PLACEHOLDER_*` strings. Replace them with your Firebase / GCS values before running.

## Architecture

| Service | Path | Role |
| --- | --- | --- |
| Web client | `web-client/` | Next.js UI (list, watch, sign-in, upload) |
| API | `api-service/` | Firebase Functions: user docs, signed upload URLs, video list |
| Processor | `video-processing-service/` | Express + FFmpeg: Pub/Sub → 360p → public GCS object |

```text
Browser  →  Firebase Auth (Google)
         →  Cloud Functions (generateUploadUrl, getVideos, createUser)
         →  GCS raw bucket
         →  Pub/Sub (implied) → video-processing-service
         →  GCS processed bucket + Firestore videos
         →  Watch page loads https://storage.googleapis.com/<processed-bucket>/...
```

## Placeholders

Replace every `PLACEHOLDER_*` value with the resource you create.

| Placeholder | Used for |
| --- | --- |
| `PLACEHOLDER_FIREBASE_API_KEY` | Firebase web API key |
| `PLACEHOLDER_FIREBASE_AUTH_DOMAIN` | `{projectId}.firebaseapp.com` |
| `PLACEHOLDER_FIREBASE_PROJECT_ID` | GCP / Firebase project ID |
| `PLACEHOLDER_FIREBASE_APP_ID` | Firebase web app ID |
| `PLACEHOLDER_RAW_VIDEO_BUCKET` | GCS bucket for original uploads |
| `PLACEHOLDER_PROCESSED_VIDEO_BUCKET` | GCS bucket for 360p outputs |
| `PLACEHOLDER_VIDEOS_COLLECTION` | Firestore collection for videos |
| `PLACEHOLDER_USERS_COLLECTION` | Firestore collection for users |
| `PLACEHOLDER_GENERATE_UPLOAD_URL_FUNCTION` | Callable name (must match the deployed function) |
| `PLACEHOLDER_GET_VIDEOS_FUNCTION` | Callable name (must match the deployed function) |
| `PLACEHOLDER_PORTFOLIO_URL` | Navbar “Back to Portfolio” link |

Files:

- `web-client/app/firebase/firebase.ts`
- `web-client/app/firebase/functions.ts`
- `web-client/app/watch/page.tsx`
- `web-client/app/navbar/navbar.tsx`
- `api-service/functions/src/index.ts`
- `video-processing-service/src/storage.ts`
- `video-processing-service/src/firestore.ts`

## Cloud setup

1. Create a Firebase / GCP project.
2. Enable Authentication (Google provider), Cloud Functions, Firestore, and Cloud Storage.
3. Create the two GCS buckets.
4. Deploy functions (`createUser` Auth trigger, plus the two callables). Keep callable **export names** in sync with the client placeholders.
5. Run `video-processing-service` on Cloud Run (or similar) with Application Default Credentials.
6. Wire a Pub/Sub push on raw-bucket object create to `POST /process-video`.
7. Host `web-client` (Vercel, Cloud Run, or `npm start` after build).

`api-service/.firebaserc` and `api-service/firebase.json` are empty until you fill them for Firebase CLI deploy.

## Local development

Requires Node 18+. The processor also needs `ffmpeg` on `PATH` (the processor Dockerfile installs it).

**Web client**

```bash
cd web-client
npm install
npm run dev
```

**Functions**

```bash
cd api-service
npm install
npm run serve
```

**Video processor**

```bash
cd video-processing-service
npm install
npm start
```

Listens on `PORT` or `3000`. Expects a Pub/Sub-style body: `req.body.message.data` is base64 JSON with a `name` field (GCS object name).

**Docker**

```bash
docker build -t web-client ./web-client
docker build -t video-processing-service ./video-processing-service
```

## Data model

**User** (`PLACEHOLDER_USERS_COLLECTION` / `{uid}`): `uid`, `email`, `photoUrl`

**Video** (`PLACEHOLDER_VIDEOS_COLLECTION`): `id`, `uid`, `filename`, `status` (`processing` | `processed`), `title`, `description`

Upload object names: `{uid}-{timestamp}.{ext}`  
Processed object names: `processed-{uid}-{timestamp}.{ext}`

## License

ISC (video-processing-service). Other packages are private / unspecified.
