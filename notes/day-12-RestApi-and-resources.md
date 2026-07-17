--------------------- Day 12 — REST APIs & Resource Reference---------------------------------------



## 1. Overview

This document consolidates the REST API conventions and full resource/endpoint map.
It is intended as both a learning record and a working reference for backend implementation decisions.



## 2. REST Fundamentals

### 2.1 Conceptual Model

A REST API mediates communication between a client and a server Neither side communicates directly with the other's internals — all interaction passes through a defined interface of resources and operations.

This separation of concerns allows the frontend and backend to be developed, deployed, and scaled independently, provided both sides honor the same API contract.

### 2.2 Resource Addressing

Every entity the system manages — a resume, a template, a user — is treated as a **resource**, and every resource is identified by a unique URL.

```
/api/documents/42
```

This addresses exactly one resource: the document with ID `42`. A URL without an ID (`/api/documents`) refers to the collection as a whole.

### 2.3 HTTP Methods 

REST operations are limited to a small, fixed set of HTTP methods. The same URL can support multiple methods, each producing a different outcome:

| Method | Operation | Description |
|---|---|---|
| `GET` | Read | Retrieve a resource or collection |
| `POST` | Create | Create a new resource, or trigger an action |
| `PUT` | Update (full) | Replace a resource entirely |
| `PATCH` | Update (partial) | Modify part of a resource |
| `DELETE` | Delete | Remove a resource |

**Example:**

```
GET    /api/documents/42   →  Retrieve document 42
DELETE /api/documents/42   →  Delete document 42
```

The URL identifies *what* is being acted on; the HTTP method defines *what action* is performed.

### 2.4 Statelessness

REST APIs are **stateless**: the server retains no memory of previous requests from a given client. Each request must carry all information required to process it independently (typically an authentication token).



### 2.5 Data Format

Request and response bodies are transmitted as **JSON** (JavaScript Object Notation) — a lightweight, structured text format that both client and server can parse unambiguously.

```json
{
  "id": 42,
  "title": "Software Engineer Resume",
  "createdAt": "2026-07-01"
}
```

### 2.6 HTTP Status Codes

Every response includes a status code summarizing the outcome of the request:

| Code | Meaning |
|---|---|
| `200` | OK — request succeeded |
| `201` | Created — resource successfully created |
| `400` | Bad Request — malformed input |
| `401` | Unauthorized — authentication required |
| `403` | Forbidden — authenticated, but not permitted |
| `404` | Not Found — resource does not exist |
| `500` | Internal Server Error — failure on the server side |

### 2.7 Core Terminology

| Term | Definition |
|---|---|
| **Resource** | A "thing" the API manages (e.g. a resume, a template) |
| **Endpoint** | A specific URL + HTTP method combination, e.g. `POST /api/documents` |

---

## 3. Glossary of Terms

| Term | Definition |
|---|---|
| **API** (Application Programming Interface) | A defined contract that allows two software systems — here, the Angular frontend and Express backend — to communicate. |
| **REST** (Representational State Transfer) | An architectural style for designing networked APIs, built around resources, URLs, and standard HTTP methods. It is a set of conventions rather than a strict protocol. |
| **RESTful** | Describes an API that adheres to REST conventions (resource-based URLs, correct verb usage, statelessness). |
| **HTTP** (HyperText Transfer Protocol) | The underlying protocol of the web. REST APIs reuse HTTP's existing methods and status codes rather than defining new ones. |
| **Endpoint** | A single URL + HTTP method pairing that performs one defined operation. |
| **Resource** | A noun-based entity managed by the API (user, document, template). Typically represented as a JSON object. |
| **CRUD** | **C**reate, **R**ead, **U**pdate, **D**elete — the four foundational operations any resource supports, mapped to `POST`, `GET`, `PUT`/`PATCH`, `DELETE`. |
| **Route** | The URL pattern defined server-side that incoming requests are matched against, e.g. `/api/documents/:id`. |
| **Route parameter** | A dynamic segment in a route, denoted with a colon, that captures a value from the URL (e.g. `:id` captures `42` from `/api/documents/42`). |
| **Request body / payload** | Data submitted with `POST`, `PUT`, or `PATCH` requests, typically a JSON object. |
| **Query parameter** | Optional key-value data appended to a URL after `?`, used for filtering, sorting, or pagination (e.g. `/api/documents?sort=recent`). |
| **Statelessness** | The principle that the server does not retain context between requests; each request must be self-contained. |
| **Idempotency** | A property of an operation where repeated execution produces the same result as a single execution. `GET`, `PUT`, and `DELETE` are expected to be idempotent; `POST` is not. |
| **JSON** (JavaScript Object Notation) | A structured, human-readable data-interchange format used for API request and response bodies. |
| **JWT** (JSON Web Token) | A compact, cryptographically signed token used to verify identity without requiring server-side session storage. |
| **Access token** | Credential (often a JWT) presented with each request to prove authentication. |
| **HTTP header** | Metadata transmitted alongside a request or response, separate from the body — commonly used for authentication (`Authorization: Bearer <token>`) or content typing. |
| **Nested route** | A URL structure reflecting a hierarchical relationship between resources, e.g. `/documents/:id/sections/:sectionId`. |
| **Action resource** | An endpoint representing a computation or process rather than persisted data (e.g. `/api/ai/bullets`). |
| **Metering** | Tracking usage of a limited, often paid, capability — typically tied to a subscription tier. |
| **Slug** | A short, URL-safe, typically randomized identifier used in place of a raw database ID for public-facing routes. |
| **Enumeration attack** | A security vulnerability where sequential or predictable identifiers are guessed to access unauthorized resources. |
| **CORS** (Cross-Origin Resource Sharing) | A browser security mechanism restricting cross-domain requests unless explicitly permitted by the server. |
| **Middleware** | Server-side logic executed between receiving a request and producing a response (e.g. authentication checks). |
| **Pagination** | The practice of dividing large result sets into discrete, sequentially retrievable pages. |

---

## 4. Resource Model

The following resources are derived from the PRD data model. Resources are divided into two categories: **persisted resources** (stored as database rows) and **action resources** (computations that return a result, without necessarily persisting state).

| | Resource | Description | Type |
|---|---|---|---|
| 1 | Auth / session | Registration, login, tokens | Action |
| 2 | User | Account, subscription tier, AI usage | Persisted |
| 3 | Document | A resume or cover letter — the central resource | Persisted |
| 4 | Section | A block within a document (experience, education, skills) | Persisted |
| 5 | Section item | A single entry within a section | Persisted |
| 6 | Version | A saved snapshot of a document | Persisted |
| 7 | Template | An available design | Persisted |
| 8 | AI | Writing actions (bullets, summary, rewrite) | Action |
| 9 | ATS check | Resume scoring against ATS criteria | Action |
| 10 | Tailoring | Alignment of a resume to a job description | Action |
| 11 | Export | Generated PDF/DOCX output | Action |
| 12 | Share | Public link to a document | Persisted |
| 13 | Application | Tracked job application | Persisted |

---

## 5. API Surface

### 5.1 Auth

| Endpoint | Purpose |
|---|---|
| `POST /api/auth/register` | Create account |
| `POST /api/auth/login` | Obtain an access token |
| `POST /api/auth/logout` | Invalidate the session |
| `POST /api/auth/forgot-password` | Begin password recovery |
| `POST /api/auth/reset-password` | Complete password reset |

All routes use `POST`, as each initiates an action (session creation or invalidation) rather than retrieving existing data.

### 5.2 Users

| Endpoint | Purpose |
|---|---|
| `GET /api/users/me` | Retrieve current profile, tier, and AI credit balance |
| `PUT /api/users/me` | Update profile |
| `DELETE /api/users/me` | Delete account and associated data |

The `me` keyword resolves to the authenticated user based on the access token, rather than an ID passed in the URL.

### 5.3 Documents (Core Resource)

| Endpoint | Purpose |
|---|---|
| `GET /api/documents` | List all resumes and cover letters |
| `POST /api/documents` | Create a document (blank or from a template) |
| `POST /api/documents/import` | Create a document by parsing an upload or LinkedIn data |
| `GET /api/documents/:id` | Retrieve a document with full content |
| `PUT /api/documents/:id` | Save edits |
| `POST /api/documents/:id/duplicate` | Duplicate a document |
| `DELETE /api/documents/:id` | Delete a document |

### 5.4 Sections & Items (Nested Under a Document)

| Endpoint | Purpose |
|---|---|
| `POST /api/documents/:id/sections` | Add a section |
| `PATCH /api/documents/:id/sections/:sectionId` | Edit or reorder a section |
| `DELETE /api/documents/:id/sections/:sectionId` | Remove a section |
| `POST /api/.../sections/:sectionId/items` | Add an entry |
| `PATCH /api/.../items/:itemId` | Edit or reorder an entry |
| `DELETE /api/.../items/:itemId` | Remove an entry |

 **Implementation note:** `PATCH` is used here rather than `PUT` because these operations modify part of a resource, not the whole.
  In practice, many implementations persist the entire document via a single `PUT /api/documents/:id` call rather than issuing per-section requests. 
  Both approaches are valid; nested routes are primarily useful when fine-grained, field-level autosave behavior is required.

### 5.5 Versions

| Endpoint | Purpose |
|---|---|
| `GET /api/documents/:id/versions` | List saved versions |
| `POST /api/documents/:id/versions` | Save current state as a new version |
| `POST /api/documents/:id/versions/:versionId/restore` | Restore a previous version |

### 5.6 Templates

| Endpoint | Purpose |
|---|---|
| `GET /api/templates` | List available designs |
| `GET /api/templates/:id` | Retrieve a single template's configuration |

Templates are read-only from the client's perspective, hence only `GET` routes are exposed.

### 5.7 AI (Metered Action Resource)

| Endpoint | Purpose |
|---|---|
| `POST /api/ai/bullets` | Generate or improve bullet points |
| `POST /api/ai/summary` | Generate a summary or headline |
| `POST /api/ai/rewrite` | Improve selected text |
| `POST /api/ai/prompt` | Apply a freeform instruction to a section |

Each call consumes a portion of the user's AI credit allowance. `POST` is used because these operations produce side effects (credit consumption) even where no data is persisted.

### 5.8 ATS Check, Tailoring, Exports

| Endpoint | Purpose |
|---|---|
| `POST /api/ats/check` | Score a document; basic tier free, full detail on Pro |
| `POST /api/tailoring` | Align a document to a job description (Pro) |
| `POST /api/exports/pdf` | Render and return a PDF file URL |
| `POST /api/exports/docx` | Render editable Word output |
| `GET /api/exports/:id` | Retrieve a previously generated export |

*ATS (Applicant Tracking System)* refers to the automated software employers use to filter resumes prior to human review. The `ats/check` endpoint evaluates a document against common ATS parsing criteria.

### 5.9 Sharing

| Endpoint | Purpose |
|---|---|
| `POST /api/documents/:id/share` | Create or refresh a public share link |
| `DELETE /api/documents/:id/share` | Revoke a share link |
| `GET /api/share/:slug` | Public, unauthenticated view of a shared document |

### 5.10 Applications (Tracker)

| Endpoint | Purpose |
|---|---|
| `GET /api/applications` | List tracked applications |
| `POST /api/applications` | Log a new application |
| `PATCH /api/applications/:id` | Update application status |
| `DELETE /api/applications/:id` | Remove an application |

---

## 6. Design Patterns

Several conventions recur throughout the API surface and are worth documenting explicitly:

**Nesting reflects ownership.** Section items are nested under sections, which are nested under documents, so the URL structure mirrors the data hierarchy: `/documents/:id/sections/:sectionId/items/:itemId`.

**Non-CRUD operations are modeled as `POST` to verb-like resources.** AI, ATS, tailoring, and export operations all produce a result via computation rather than persisted storage. `POST` is appropriate here since `GET` requests are expected to be safe (free of side effects) and idempotent, whereas these operations are neither.

**One endpoint is intentionally unauthenticated.** `GET /api/share/:slug` omits authentication by design, since public accessibility is the purpose of a share link. It uses a randomized slug rather than the raw document ID to prevent **enumeration attacks** — the practice of guessing sequential identifiers to access unauthorized resources.

**HTTP verbs carry intent; URLs carry identity.** A single URL can support read, update, and delete operations, differentiated solely by method. This minimizes the total number of distinct routes required and keeps the API surface predictable.

---

## 7. Client-Side Storage Reference

The Angular frontend requires local persistence for authentication tokens, drafts, and preferences. Four browser storage mechanisms are relevant, each suited to a different use case.

### 7.1 Cookies

Small data packets automatically attached to every outgoing HTTP request to a matching domain. Functionally comparable to a visitor badge — issued once, then recognized automatically on subsequent visits.

| Property | Value |
|---|---|
| Capacity | ~4 KB |
| Sent to server | Yes, automatically |
| Expiration | Configurable (session or fixed duration) |
| Typical use | Authentication, session identifiers |

### 7.2 Local Storage

Persistent key-value storage scoped to the browser, retained indefinitely until explicitly cleared.

| Property | Value |
|---|---|
| Capacity | ~5–10 MB |
| Sent to server | No |
| Expiration | None (persists until cleared) |
| Typical use | User preferences (e.g. theme) |

```js
localStorage.setItem("theme", "dark");
localStorage.getItem("theme");
localStorage.removeItem("theme");
```

### 7.3 Session Storage

Key-value storage scoped to a single browser tab, cleared automatically when that tab is closed.

| Property | Value |
|---|---|
| Capacity | ~5 MB |
| Sent to server | No |
| Expiration | On tab close |
| Typical use | Temporary, in-progress form data |

```js
sessionStorage.setItem("name", "Dinesh");
sessionStorage.getItem("name");
```

### 7.4 IndexedDB

A full in-browser database capable of storing large volumes of structured data, including binary objects such as files and images.

| Property | Value |
|---|---|
| Capacity | Hundreds of MB or more |
| Sent to server | No |
| Expiration | None (persists until cleared) |
| Typical use | Offline application data, cached files |

### 7.5 Comparative Summary

| Feature | Cookies | Local Storage | Session Storage | IndexedDB |
|---|---|---|---|---|
| Capacity | 4 KB | 5–10 MB | 5 MB | Hundreds of MB+ |
| Sent to server | Yes | No | No | No |
| Expires | Yes | No | On tab close | No |
| Suitable for large data | No | No | No | Yes |
| Stores structured objects | No | Strings only | Strings only | Yes |
| Primary use case | Login, auth | Preferences | Temporary form state | Offline, large data |

### 7.6 Reference Examples

| Application | Storage Mechanism | Rationale |
|---|---|---|
| Gmail | Cookies | Session persistence |
| YouTube | Local Storage | Preference retention (theme, volume) |
| Amazon | Cookies + Local Storage | Authentication, cart, preferences |
| Google Docs | IndexedDB | Offline document access |
| Spotify | IndexedDB | Offline song caching |
| Google Maps | IndexedDB | Offline map tile storage |

---

## 8. Review Questions

| Question | Answer |
|---|---|
| Which storage mechanism is transmitted with every server request? | Cookies |
| Which mechanisms persist across a browser restart? | Local Storage, IndexedDB |
| Which mechanism is cleared when the tab closes? | Session Storage |
| Which mechanism is best suited to offline applications and large datasets? | IndexedDB |
| Can Local Storage store structured objects directly? | No — string values only; use `JSON.stringify()`/`JSON.parse()`. IndexedDB supports structured objects natively. |
| What distinguishes `PUT` from `PATCH`? | `PUT` replaces a resource in full; `PATCH` modifies part of it. |
| Why do AI/ATS/export endpoints use `POST` despite not always persisting data? | They produce side effects (credit consumption, file generation), disqualifying them from `GET`, which must remain safe and idempotent. |
| Why does `GET /api/share/:slug` omit authentication? | Public accessibility is the intended function of a share link. |
| Why use a slug rather than a raw document ID for sharing? | To prevent enumeration attacks against sequential identifiers. |

---

## 9. Application to ResumeFlow


- `/api/ai/*`, `/api/ats/check`, `/api/exports/*` → use `POST`.
- `GET /api/share/: slug` → only unauthenticated route; use a random slug, never the raw document ID.
- Token storage: cookie or memory (not Local Storage ). Preferences: Local Storage. Unsaved form state: Session Storage.
- Keep CRUD mapping consistent: `POST`=Create, `GET`=Read, `PUT`/`PATCH`=Update, `DELETE`=Delete.