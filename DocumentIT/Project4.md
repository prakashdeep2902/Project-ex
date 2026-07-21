## Built a conversational AI assistant enabling employees to perform HR operations using natural language. TypeScript • Node.js • React • OpenAI API • RAG • PostgreSQL • LLM • REST APIs

Below is a concise interview revision sheet you can read in 5–10 minutes before an interview.

Sure, in simple words —

---

**Why JavaScript to TypeScript:**

In plain JavaScript, there's no type checking — so if a function expects a number but someone passes a string by mistake, JavaScript won't complain, it'll just break later, sometimes even in production. We were facing this a lot — API response mismatches, wrong data passed to components, and these bugs were showing up after deployment instead of during development.

TypeScript fixes this by checking types while writing the code itself. So if something's wrong, we see it immediately in the editor, not after the user faces it. It also makes refactoring safer — if I change something in one place, TypeScript tells me everywhere else that needs to change too, instead of me manually hunting for it.

**Why React to Next.js:**

Our app was a plain React SPA (single page application), and it wasn't using a lot of things Next.js gives for free — like:

- **File-based routing** — instead of manually setting up React Router, Next.js automatically creates routes based on folder structure, simpler to manage.
- **Server-side rendering (SSR)** — pages can load faster and are better for SEO, since content can render on the server first instead of everything loading in the browser.
- **Better performance optimizations** — like automatic code splitting, image optimization, so pages load faster without us manually configuring all of this.
- **More structured project setup** — Next.js has a more opinionated, organized folder structure, which helps as the codebase grows.

**In short:** TypeScript was chosen to catch bugs early and make the code safer to change. Next.js was chosen because it gave us performance, routing, and SEO benefits that plain React wasn't using, plus a cleaner project structure as our HRMS kept growing bigger.

---

Sure, in simple words —

---

**Issue 1 — Too many TypeScript errors at the start**

When we started converting from JavaScript to TypeScript, almost every file had errors because JavaScript doesn't check types, but TypeScript does. Many places were just using `any` type everywhere, which basically means "no real type checking," so it didn't help much.

**How I fixed it:** I didn't try to fix everything at once. I migrated module by module — one small part at a time — added proper interfaces for the data, and slowly turned on strict typing instead of doing it all in one go. This made it manageable instead of overwhelming.

---

**Issue 2 — API response data had no proper type**

Before, when we called an API like `getEmployee()`, the response was just typed as `any` — meaning TypeScript had no idea what fields it returns. So if the backend changed something, we wouldn't know until it broke in production.

**How I fixed it:** I created proper interfaces, like `Employee`, and made the API return `Promise<Employee>` instead of `any`. Now if the API response doesn't match what we expect, TypeScript shows the error immediately while coding, not after deployment.

---

**Issue 3 — Reusable components didn't have clear rules**

Components like Table, Search Box, Modal, Pagination were used in many places, but there was no clear definition of what data (props) they expect. So sometimes wrong data got passed by mistake, and it broke silently.

**How I fixed it:** I added strongly typed props to these components. So now, if someone passes wrong data type to a component, TypeScript catches it immediately, before it even runs.

---

**Issue 4 — Big employee form was error-prone**

The Add/Edit Employee form had many fields — name, email, department, manager, salary grade, joining date — and it was easy to send wrong or missing data.

**How I fixed it:** I created a proper interface for the form data, so all fields have a defined type. This improved validation and reduced bugs where wrong data type could go through silently.

---

**Issue 5 — SSR errors when moving to Next.js**

Some code used browser-only things like `window` or `localStorage`. This works fine in normal React, but in Next.js, some code runs on the server first, where `window` doesn't exist. So it was throwing errors like `window is not defined`.

**How I fixed it:** I moved that browser-specific code inside `useEffect`, which only runs in the browser, or added a check like `typeof window !== "undefined"` before using it. This avoided the SSR crash.

---
