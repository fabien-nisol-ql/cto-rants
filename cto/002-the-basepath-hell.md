# 🛠️ OAuth and the Lost Path: Debugging Next.js, next-auth, and Keycloak in BasePath Hell
 
**Subtitle:** *“I tried to log in and all I got was this dirty debug glove”*
 
---
 
## 👋 Intro
 
Let me tell you a tale of misdirection, missing logs, and middleware madness.  
It started with something that *should* be simple:
 
> Implement login with Keycloak in a Next.js app deployed under `/dashboard`.
 
In 2025, we’re still building web apps on top of stacks so abstracted and “smart” they forget to tell you where things break. This post is a log of my journey through:
 
- 🔥 `next-auth` — silent failure as a service  
- 🔥 `next.config.js` `basePath` — the dark spell that breaks your URLs  
- 🔥 `openid-client` — “it works” (until it doesn’t) and logs nothing when it breaks  
- 🔧 Reverse proxies, redirect URIs, and OAuth2 specs that expect *exact* matches
 
---
 
## 😐 The Symptoms
 
- Redirect URI mismatch errors in Keycloak  
- Code grant flow fails *silently*  
- `next-auth` doesn't throw an error — it just doesn't call the callback  
- `openid-client` doesn't log the request or error unless you dive into internals  
- Middleware’s `request.url` magically **strips `basePath`** without telling you
 
---
 
## 😤 The Root Cause
 
> `basePath` isn’t respected where you actually need it most: in backend routing, auth callbacks, and URL construction.
 
Here’s what Next.js gives you:
 
- ✅ Routing and assets respect `basePath`  
- ❌ `request.url`, middleware, and auth logic **do not**  
- ❌ You can’t access `basePath` in code unless you duplicate it into `env`
 
---
 
## 🧪 The Fix: Manual Surgery
 
I wrote this **beautifully angry utility function** to fix what Next.js broke:
 
```ts
// utils/fixNextBozosBasePath.ts
 
const BASE_PATH = process.env.NEXT_PUBLIC_BASE_PATH || '/dashboard';
 
/**
* Appends basePath to broken URLs that forgot where they live.
*/
export function fixNextBozosBasePath(input: string): string {
  try {
    const url = new URL(input);
    if (!url.pathname.startsWith(BASE_PATH)) {
      url.pathname = `${BASE_PATH}${url.pathname}`;
    }
    return url.toString();
  } catch {
    if (!input.startsWith(BASE_PATH)) {
      return `${BASE_PATH}${input.startsWith('/') ? '' : '/'}${input}`;
    }
    return input;
  }
}
```
 
Now every time I call `fixNextBozosBasePath()`, it’s a little act of rebellion against framework gaslighting.
 
---
 
## 🧤 The Debug Glove
 
With no logs from `next-auth`, and barely any from `openid-client`, I had to go full 1990s: inspect `request.url`, print internal state, reconstruct the redirect URI *by hand*, and verify that it matched exactly what Keycloak expected.
 
Modern JavaScript frameworks: like writing 6502 assembly, but worse because it pretends to be helpful.
 
---
 
## 🥃 Lessons Learned
 
- **Avoid `basePath` if possible.** Use a `/dashboard` folder in your routing structure instead.  
- If you must use `basePath`, **manually inject it into `env`** so you can access it reliably.  
- **Log everything yourself**. `next-auth` and `openid-client` will not help you when it counts.  
- OAuth2 is precise: `redirect_uri` must match *exactly* during the code exchange.
 
---
 
## 💬 Final Thoughts
 
To anyone else out there fighting `next-auth` and `Keycloak` under a `basePath`:
 
You are not crazy.  
You are not alone.  
And yes, assembly really *was* better — at least it did what you told it to.
 
If this helped or made you feel heard, share it with the next dev who's wondering why their OAuth callback returns nothing but existential dread.
 
---
 
## 🧷 Bonus: Suggested commit messages
 
- `fix: make redirect_uri suck less under basePath`  
- `chore: band-aid for Next.js being clever in all the wrong ways`  
- `feat: add middle finger to framework in utility name`  
- `wip: this works and I don't want to talk about it`
