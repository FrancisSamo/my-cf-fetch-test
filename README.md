# Minimal Reproduction: SvelteKit + Cloudflare Workers - 'Fetch Undefined' Error

This repository contains a minimal SvelteKit project designed to reproduce a `TypeError: Cannot read properties of undefined (reading 'fetch')` when deployed to Cloudflare Workers.

## Problem Description

The error `TypeError: Cannot read properties of undefined (reading 'fetch')` occurs on the Cloudflare Worker (server-side) when the worker attempts to serve SvelteKit's generated JavaScript/CSS assets. This results in HTTP 500 errors for these assets in the browser.

The typical stack trace points to an issue within the `_worker.js` file, for example:

```
at Object.fetch (_worker.js:LINE_NUMBER)
```

## Environment & Versions

*   **SvelteKit (@sveltejs/kit):** `^2.21.1`
*   **Svelte:** `^5.33.4`
*   **@sveltejs/adapter-cloudflare:** `^7.0.3`
*   **Vite:** `^6.3.5`
*   **TypeScript:** `^5.8.3`
*   **Node.js:** (Assumed `nodejs_compat` is used, version may vary, e.g., v20.x)
*   **Wrangler CLI:** (Version may vary, e.g., 4.x)

## `wrangler.toml` Configuration

```toml
name = "my-cf-fetch-test"
main = ".svelte-kit/cloudflare/_worker.js"
compatibility_date = "2024-09-23"
compatibility_flags = [ "nodejs_compat" ]
```

## `svelte.config.js` Configuration

```javascript
import adapter from '@sveltejs/adapter-cloudflare';
import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';

/** @type {import('@sveltejs/kit').Config} */
const config = {
	// Consult https://kit.svelte.dev/docs/integrations#preprocessors
	// for more information about preprocessors
	preprocess: [vitePreprocess({})],

	kit: {
		adapter: adapter()
	}
};

export default config;
```

## `package.json` (Key Dependencies)

```json
{
	"devDependencies": {
		"@sveltejs/adapter-cloudflare": "^7.0.3",
		"@sveltejs/kit": "^2.21.1",
		"@sveltejs/vite-plugin-svelte": "^5.0.3",
		"svelte": "^5.33.4",
		"svelte-check": "^4.2.1",
		"typescript": "^5.8.3",
		"vite": "^6.3.5"
	}
}
```

## Steps to Reproduce

1.  **Clone the repository:**
    ```bash
    gh repo clone FrancisSamo/my-cf-fetch-test
    ```
2.  **Install dependencies:**
    ```bash
    npm install
    ```
3.  **Configure Wrangler (if necessary):**
    If you haven't used Wrangler before, you might need to log in:
    ```bash
    wrangler login
    ```
4.  **Build the project:**
    ```bash
    npm run build
    ```
5.  **Deploy using Wrangler:**
    ```bash
    wrangler deploy
    ```
6.  **Visit the deployed URL** provided by Wrangler (e.g., `https://my-cf-fetch-test.<your-cf-subdomain>.workers.dev`).
7.  **Check for errors:**
    *   Open your browser's **Developer Tools**.
    *   Go to the **Network** tab: Look for HTTP 500 errors when loading JavaScript (`.js`) or CSS (`.css`) files from the `/_app/immutable/...` path.
    *   Go to the **Cloudflare Dashboard** -> Workers & Pages -> `my-cf-fetch-test` -> Logs. Look for a `TypeError` related to `fetch` being undefined.

## Expected Erroneous Behavior

If the bug is reproduced, you will observe:

*   **Browser:** HTTP 500 errors for SvelteKit's generated JS/CSS assets in the Network tab.
*   **Cloudflare Worker Logs:** A `TypeError` similar to the following:

    ```json
    {
      "source": {
        "message": "Cannot read properties of undefined (reading 'fetch')",
        "exception": {
          "stack": "    at Object.fetch (_worker.js:LINE_NUMBER:COLUMN_NUMBER)", 
          "name": "TypeError",
          "message": "Cannot read properties of undefined (reading 'fetch')"
        }
      }
      // ... rest of the log ...
    }
    ```

## Actual Cloudflare Log Output

```json
{
  "source": {
    "message": "Cannot read properties of undefined (reading 'fetch')",
    "exception": {
      "stack": "    at Object.fetch (_worker.js:21546:31)",
      "name": "TypeError",
      "message": "Cannot read properties of undefined (reading 'fetch')",
      "timestamp": 1748366321052
    },
    "$cloudflare": {
      "$metadata": {
        "id": "01JW9C1SCWRAE997G8AFVY9AVY",
        "type": "cf-worker",
        "error": "Cannot read properties of undefined (reading 'fetch')"
      }
    }
  },
  "dataset": "cloudflare-workers",
  "timestamp": "2025-05-27T17:18:41.052Z",
  "$workers": {
    "truncated": false,
    "event": {
      "request": {
        "url": "https://my-cf-fetch-test.<cf-subdomain>.workers.dev/",
        "method": "GET",
        "path": "/"
      }
    },
    "outcome": "exception",
    "scriptName": "my-cf-fetch-test",
    "eventType": "fetch",
    "executionModel": "stateless",
    "scriptVersion": {
      "id": "7bf589a1-2042-4694-95ff-6d6f66df7424"
    },
    "requestId": "94672c427ae2bac0"
  },
  "$metadata": {
    "id": "01JW9C1SCWRAE997G8AFVY9AVY",
    "requestId": "94672c427ae2bac0",
    "trigger": "GET /",
    "service": "my-cf-fetch-test",
    "level": "error",
    "error": "Cannot read properties of undefined (reading 'fetch')",
    "message": "Cannot read properties of undefined (reading 'fetch')",
    "account": "b7e4dac514add5403e662c5ebf100a3b",
    "type": "cf-worker",
    "fingerprint": "6c8bd26f56e625a02177b880ae3c7ce3",
    "origin": "fetch"
  },
  "links": []
}
```

## Link to GitHub Issue

This repository can be used to support a GitHub issue filed for [this bug](https://github.com/sveltejs/kit/issues/13832).
