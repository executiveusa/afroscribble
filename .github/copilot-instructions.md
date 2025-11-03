# Codex Quickstart for Afroscribble

## Project shape
- Next.js 13 app with classic `pages/` router; Tailwind utility classes drive styling.
- Core drawing flow lives in `pages/index.js` with shared UI in `components/` (canvas, prompt form, predictions, welcome modal).
- API routes under `pages/api/` use the Edge runtime for Replicate calls, webhooks, and OG image generation; respect streaming body parsing helpers already in place.
- Prisma models (`prisma/schema.prisma`) persist successful predictions to MySQL; `lib/prisma.js` keeps a singleton client in non-prod.

## User journey & data flow
1. Users sketch via `components/canvas.js` (react-sketch-canvas) and submit prompts through `components/prompt-form.js`.
2. `uploadFile` (`lib/upload.js`) uploads the canvas PNG to Bytescale using the global `Upload` object injected by the script tag on the homepage—avoid importing it directly.
3. `/api/predictions` creates a Replicate prediction with the provided prompt/image and optional local storage token; progress is polled via `/api/predictions/[id]`.
4. Replicate webhooks hit `/api/replicate-webhook`, which upserts completed predictions into MySQL through `lib/db.js`.
5. `/scribbles/[id].js` renders saved predictions server-side so OG metadata stays accurate; OG previews are produced by `/api/og`.

## Environment & config
- Required env vars: `REPLICATE_API_TOKEN` (client-side prompt form expects users to supply it), `DATABASE_URL` for Prisma, `NGROK_HOST` for local webhooks, optional `VERCEL_URL` for production host detection.
- `package.json` scripts: `npm run dev`, `npm run lint`, `npm run build`, `npm run start`. `npm run test` runs lint + build.
- `postinstall` triggers `prisma generate`; run `npx prisma migrate dev` before using the local database.

## Patterns to follow
- Keep Edge handlers response shapes aligned with `NextResponse.json` usage already present.
- Maintain polling loop structure in `pages/index.js` if adjusting prediction handling; it expects 200/201 statuses and `prediction.status` transitions.
- UI copy relies on `package.json` metadata (`appName`, `appSubtitle`, etc.); update there when changing branding.
- `components/predictions.js` assumes predictions include both `input.image` and `output[]`; guard changes accordingly to avoid null renders.
- Tailwind utility classes are preferred; check `styles/globals.css` for shared tokens like `.lil-button` and animation helpers.

## Testing & debugging tips
- Use `npm run lint` to catch most issues; the codebase does not ship unit tests.
- Replicate routes run on Edge—avoid Node-only APIs (e.g., `fs`) there.
- When exercising webhooks locally, expose your dev server and set `NGROK_HOST`; `lib/db.js` skips incomplete predictions by design.
- If the Bytescale upload script is absent, `Upload` will be undefined—ensure `<Script src="https://js.bytescale.com/upload-js-full/v1" />` remains on pages that call `uploadFile`.

## PR expectations
- Document new env vars in `README.md` and extend Prisma schema/migrations together.
- Update OG metadata helpers if you rename API routes or change image dimensions.
- Add new UI components under `components/` and export reusable logic from `lib/` rather than mixing into pages.
