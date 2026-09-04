# OFTURK Tarot

The frontend is a static GitHub Pages site. GitHub Pages cannot execute the server-side AI endpoint or protect an API key, so the AI reading endpoint must be deployed separately.

## Deployment architecture

The frontend remains a static GitHub Pages site. GitHub Pages serves `index.html`, `index.css`, `script.js`, `api-config.js`, and the card assets only; it does not run server-side JavaScript and must never receive the OpenAI secret.

The AI endpoint is the Vercel Serverless Function at `api/reading.js`. Deploy this repository as a separate Vercel project, or deploy the `api` function in an API project. The endpoint uses the OpenAI Responses API and creates the prompt server-side.

## Vercel setup

1. Create a Vercel project for the API and deploy `api/reading.js`.
2. In Vercel Project Settings, add the environment variable `OPENAI_API_KEY` for the required environments. Do not commit its value.
3. Optionally add `OPENAI_MODEL`; the default is `gpt-4o-mini`.
4. Add `ALLOWED_ORIGINS` with `https://ofturk.com.tr,https://www.ofturk.com.tr`. Local development may add an explicit localhost origin separated by commas.
5. Confirm the deployed endpoint is `https://YOUR-VERCEL-DOMAIN/api/reading`.

Update the public, non-secret `readingEndpoint` value in `api-config.js`:

```js
window.OFTURK_CONFIG = {
	readingEndpoint: 'https://YOUR-VERCEL-DOMAIN/api/reading'
};
```

Do not put `OPENAI_API_KEY` in `api-config.js`, HTML, CSS, or any frontend JavaScript. The GitHub Pages frontend sends the selected deck, spread, question, category, language, and card directions to the configured Vercel URL only when the user clicks `Yorumunu Gör`.

## API contract

The endpoint accepts only 3, 5, 7, or 10 cards and validates card IDs, positions, orientations, meanings, language, and category. It handles `OPTIONS` preflight and allows only the configured origins. Errors use this shape:

```json
{ "error": { "code": "AI_PROVIDER_ERROR", "message": "Reading service unavailable" } }
```

Successful responses use:

```json
{
	"overall": "...",
	"cardReadings": [{ "position": "...", "interpretation": "..." }],
	"advice": "..."
}
```

The in-memory rate limit is intentionally basic and per serverless instance. It reduces accidental bursts but is not a global distributed limit. For production scale, add a shared store or an API gateway rate limiter.

## Local checks

Run:

```powershell
node --check script.js
node --check api/reading.js
```

The frontend can be previewed with a static server, but a static server cannot execute `api/reading.js`; use a Vercel preview/deployment to test a real OpenAI call. The browser flow also supports mocked `/api/reading` responses for loading, success, error, retry, language, and reset tests.
