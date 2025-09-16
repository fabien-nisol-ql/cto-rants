# Stop Passing `open` Around — React + TypeScript “Best Practices” Are Broken

> *“If your parent component knows when a dialog opens, you’ve already lost.”*  

---

## The Problem

Every React + TypeScript tutorial shows this “best practice” for dialogs:

```tsx
const [open, setOpen] = useState(false);

<UploadDialog open={open} onOpenChange={setOpen} />;
```

This looks clean. It’s **easy to teach**.  
But if you’re building a **complex app**, it’s **terrible design**.

Here’s why:

- The **parent** now decides when the dialog opens or closes.
- The **parent** must know how to reset the dialog’s internal state.
- Every **new dialog** requires editing multiple unrelated parents.
- Your parent components become **god objects** that micromanage everything.

This is **not** component encapsulation.  
It’s **prop-drilling hell** — and React + TS tutorials keep promoting it as if it’s gospel.

---

## Why This Happened

### 1. React Went All-In on Functional Purity

React’s philosophy is:

> **UI = pure function(state) → JSX**

So tutorials teach you to **“lift state up”** — dialogs become **stateless shells**, and parents **own their lifecycle**.

That’s fine for **simple components** like buttons or icons.  
But dialogs, uploaders, and wizards are **stateful UI objects**.  
Treating them as “dumb shells” breaks encapsulation.

---

### 2. TypeScript Followed React’s Lead

When TypeScript came along, many expected it to **nudge frontend devs** toward better **OO design**.

Instead, it just **typed React tutorials**:

- Same prop patterns
- Same “controlled components” obsession
- Same violation of **encapsulation**

TypeScript became **React with types**, not **OO JS**.

---

### 3. Tutorials Optimize for Juniors

Why is this pattern everywhere?

- Easy to **teach** ✅  
- Easy to **copy/paste** ✅  
- Scales like garbage ❌

For small apps, you won’t notice.  
For enterprise-grade UIs with **multiple dialogs, wizards, uploads, and workflows**…  
you’ll drown in **prop-drilling, boilerplate, and tangled state**.

---

## The Better Way: Encapsulated, OO-Style Components

A **dialog** is a **stateful UI object**.  
It should **manage its own state** and expose a **simple API**:

```ts
openUploadDialog(data);
```

That’s it.  
No parent state. No prop juggling.

---

### Example

#### ✅ Encapsulated Dialog

```ts
openUploadDialog({ files });
```

Inside `<UploadDialog />`:

- Handles `open/close` internally.
- Starts uploading automatically.
- Manages progress, errors, retries.
- Outside world **doesn’t care how**.

---

#### ❌ Props-Driven Dialog

```tsx
<UploadDialog
    open={uploadDialogOpen}
    onClose={() => setUploadDialogOpen(false)}
    files={selectedFiles}
    resetWizardSteps={resetWizardSteps}
    onRetry={retryLastUpload}
/>
```

- Parent knows **everything**.
- Parent resets state manually.
- Adding a new feature → edit **every caller**.

This is **tight coupling** disguised as “best practice”.

---

## Tell, Don’t Ask

A classic OO principle:

> *Don’t ask an object for its state and decide what to do.  
Tell the object what you want it to do.*

With prop-driven dialogs, you **ask**:  
> “Are you open? Should I open you? Should I reset you?”

With encapsulated dialogs, you **tell**:  
> “Start uploading these files.”

And the dialog **handles itself**.

---

## Performance: OO Encapsulation Wins

There’s a myth that “lifting state up” is faster. It’s **not**.

| Aspect                  | Props-driven (trendy)                               | Encapsulated (OO)   |
|-------------------------|------------------------------------------------------|----------------------|
| **Re-renders**          | Every `open` toggle re-renders parent + children     | Only dialog re-renders |
| **Mount cost**          | Often unmounted/remounted, state resets each time    | Mounted once, state reused |
| **Trigger cost**        | State updates cascade → **O(n)** re-renders          | One function call → **O(1)** |
| **Readability**         | State spread across multiple parents                 | State local to dialog |

For complex apps, encapsulated dialogs are **more performant**:

- No unnecessary parent re-renders.  
- No prop-drilling.  
- No wasted mounts/unmounts.  

---

## How to Know Dialog State (When Needed)

Encapsulation doesn’t mean isolation.  
If the parent **must** know, expose an **API**, not internal state.

### Option A — Read-Only Getter

```ts
export function getUploadDialogStatus() {
    return getUploadDialogState?.() ?? "closed";
}

if (getUploadDialogStatus() === "uploading") {
    console.log("Dialog is busy");
}
```

### Option B — Callbacks

```tsx
<UploadDialog onFinish={(docs) => refreshProductList(docs)} />
```

- Parent reacts only to **meaningful events**.  
- Dialog lifecycle stays internal.  

---

## Why This Matters

In real-world enterprise apps we have:

- Multi-step **wizards**
- Multi-file **uploads** with progress
- Complex **workflows**

If we followed the “prop-driven” model:

- Parent components would explode with state.  
- Every small refactor would break unrelated code.  
- Adding a new dialog would be painful.  

Instead, we use **encapsulated dialogs**:

```ts
openProductWizard();
openUploadDialog(data);
```

- Self-contained components  
- Predictable APIs  
- Safer refactors  
- Cleaner mental model  

---

## Rules to Live By

### For Dialogs & Wizards

- ✅ Manage `open/close` **internally**  
- ✅ Expose **intention-driven APIs** like `openDialog()`  
- ✅ Fire **events**, not props, when external components need updates  
- ✅ Keep **parents clean** — they don’t care how dialogs work  

### Avoid

- ❌ Passing `open` props around  
- ❌ Making parents reset dialog state  
- ❌ Prop-drilling `onClose`, `onSubmit`, `onRetry` everywhere  
- ❌ Treating **stateful components** like **stateless shells**  

---

## Final Thought

React’s “best practices” are designed for **teaching beginners**, not **building scalable systems**.

For enterprise-grade apps:  
- Encapsulation isn’t “old school” — it’s **engineering discipline**.  
- Dialogs, wizards, uploaders → **stateful objects**, not dumb JSX shells.  
- Stop lifting state up for everything.  
- **Tell, don’t ask.**  
- **Encapsulate, don’t prop-drill.**  

---

**TL;DR:**  
> If you’re passing `open` as a prop, you’re doing React’s job for it.  
> Make your dialogs **stateful**, expose **APIs**, and keep your parents clean.  
