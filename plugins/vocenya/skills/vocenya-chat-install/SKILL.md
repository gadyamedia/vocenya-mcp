---
name: vocenya-chat-install
description: 'Add the Vocenya website live chat to a site you are building: the one script tag, where it goes in each framework (Next.js, Vite, Nuxt, SvelteKit, Astro, Remix, site builders), Content Security Policy sources and the JavaScript API.'
---

# Add the Vocenya live chat to a website

You are adding a third-party live chat widget to this website. It is one script tag. Follow these instructions exactly.

## What to add

Add this script tag once, so it loads on every page, just before the closing `</body>` tag (or the framework equivalent below):

```html
<script
    src="https://vocenya.com/api/chat/widget.js"
    data-key="pk_YOUR_SITE_KEY"
    async
></script>
```

Replace `pk_YOUR_SITE_KEY` with the site key from the Vocenya portal (Live chat → Install). Each business has its own key.

The chat bubble then appears in the corner of every page. Nothing else is required.

## Rules

- Load the script once, site-wide, in the shared layout or template. Not per page and not twice.
- Load it from https://vocenya.com/api/chat/widget.js exactly. Do not download, bundle, self-host, inline, minify or proxy the script: it is updated on our side.
- Do not change, remove or invent the `data-key` value.
- Keep `async` and do not add `type="module"`.
- It is browser-only. Keep it out of server-only code paths (server components' logic, API routes, SSR data loaders, edge functions). Never call `window.Vocenya` during server rendering; call it only in the browser, from event handlers or effects.
- Do not install an npm package for it: there is none. Do not add a chat component of your own.
- If the site sends a Content Security Policy, add the sources below rather than loosening the policy any further.

## Where it goes, by framework

### Plain HTML or any server-rendered template

Paste the tag just before `</body>` in the template every page shares (footer include, base layout, `theme.liquid`, etc.).

### Next.js (App Router)

In the root layout `app/layout.tsx`, use `next/script` with `strategy="afterInteractive"`, inside `<body>`:

```tsx
import Script from 'next/script';

export default function RootLayout({
    children,
}: {
    children: React.ReactNode;
}) {
    return (
        <html lang="en">
            <body>
                {children}
                <Script
                    src="https://vocenya.com/api/chat/widget.js"
                    data-key="pk_YOUR_SITE_KEY"
                    strategy="afterInteractive"
                />
            </body>
        </html>
    );
}
```

Pages Router: put the same `<Script>` in `pages/_app.tsx`.

### Vite + React (Lovable, Bolt, v0 exports, Replit and most AI-built React apps)

Add the tag to `index.html` at the project root, just before `</body>`. Not inside a React component.

### Vue (Vite) and Nuxt

Vue with Vite: add the tag to `index.html` before `</body>`. Nuxt: add it with `useHead` in `app.vue` (or `app.head.script` in `nuxt.config.ts`):

```ts
useHead({
    script: [
        {
            src: 'https://vocenya.com/api/chat/widget.js',
            'data-key': 'pk_YOUR_SITE_KEY',
            async: true,
            tagPosition: 'bodyClose',
        },
    ],
});
```

### SvelteKit

Add the tag to `src/app.html`, just before `</body>`.

### Astro

Add the tag to the base layout every page uses (for example `src/layouts/Layout.astro`), just before `</body>`, with `is:inline` so Astro does not bundle it:

```astro
<script is:inline src="https://vocenya.com/api/chat/widget.js" data-key="pk_YOUR_SITE_KEY" async></script>
```

### Remix and React Router (framework mode)

In `app/root.tsx`, inside `<body>` next to `<Scripts />`:

```tsx
<script
    src="https://vocenya.com/api/chat/widget.js"
    data-key="pk_YOUR_SITE_KEY"
    async
></script>
```

### Webflow, Framer, Wix, Squarespace and other site builders

Use the site-wide custom code setting, not a page embed:

- Webflow: Site settings → Custom code → Footer code. Publish.
- Framer: Site settings → General → Custom code → End of <body> tag. Publish.
- Wix: Settings → Custom code → Add custom code, All pages, Load once, Body - end.
- Squarespace: Settings → Advanced → Code injection → Footer.
- WordPress: use the Vocenya Chat plugin from the Vocenya portal, or the theme footer.

## Content Security Policy

Only if the site sets a Content-Security-Policy (header or meta tag), add these sources to the existing directives:

```
script-src https://vocenya.com
connect-src https://vocenya.com wss://ws.vocenya.com
img-src https://vocenya.com
frame-src https://vocenya.com
```

`connect-src` covers the chat API and the live-reply websocket. `frame-src` is only needed if you also embed the hosted chat page in an iframe.

## Optional: the JavaScript API

Only if asked. To call the chat before the script has loaded, add this stub above the script tag; calls are queued and replayed:

```html
<script>
    window.Vocenya =
        window.Vocenya ||
        function () {
            (window.Vocenya.q = window.Vocenya.q || []).push(arguments);
        };
</script>
```

- `Vocenya('open')`, `Vocenya('close')`, `Vocenya('toggle')`: control the chat window.
- `Vocenya('identify', { name, email, phone })`: prefill a signed-in visitor so the team sees who they are.
- `Vocenya('on', 'message' | 'open' | 'close' | 'lead', callback)` and `Vocenya('off', event, callback)`: listen for chat events.

A "Chat with us" button:

```html
<button type="button" onclick="window.Vocenya && window.Vocenya('open')">
    Chat with us
</button>
```

Identify a logged-in user in React (browser only):

```tsx
useEffect(() => {
    if (user) {
        window.Vocenya?.('identify', { name: user.name, email: user.email });
    }
}, [user]);
```

In TypeScript, declare it once: `declare global { interface Window { Vocenya?: (...args: unknown[]) => void } }`.

## Allowed domains

The chat only runs on the domains listed under Live chat → Install in the Vocenya portal. Make sure the site's live domain is on that list; preview or staging domains (for example `*.lovable.app` or `*.vercel.app`) must be added too if the chat should work there.

## Check it worked

1. Publish or deploy the site.
2. Open the live site in a private window: a chat bubble appears in the corner. Click it and send a test message.
3. No bubble? Open the browser console: look for a blocked script or connection (Content Security Policy) or a domain that is not allowed.
4. Within a few minutes the Vocenya portal shows the chat as "Installed".

## When you are done

Tell the owner which file you changed and confirm the script tag is unchanged. Docs: https://vocenya.com/developers/chat-install.md
