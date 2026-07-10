------------------------------Day-11 TASK  HTTP Standards & REST APIs---------------------------------------

## 1. What HTTP actually is, and why it had to be invented

HTTP stands for **HyperText Transfer Protocol** — text (with links) (Transfer) being moved around, following a set of agreed rules (Protocol).

Here's why the "protocol" part matters so much: imagine a Mac running Chrome trying to talk to a Linux server. Under the hood they're completely different systems. Without an agreed set of rules for how a message should be *formatted*, which *actions* are allowed, and how a *result* gets reported back, they'd have no way to understand each other — like two strangers speaking different languages. HTTP is the language they both agree to speak.

Important nuance for interviews: **HTTP is a protocol, not a program.**
 It's a rulebook, the same way traffic rules are a rulebook — they don't drive the car, they just define how driving is supposed to work.

## 2. Client and server — a strict back-and-forth

- **Client** = the one who always asks (browser, mobile app, Postman)
- **Server** = the one who always answers (your Node.js app, awake and listening)

The cycle is always: ask → think → answer → forget. And that last word, "forget," leads straight into the next big idea.

## 3. HTTP is stateless — and that trips a lot of people up

Here's the catch: **the server forgets you the instant it finishes answering.** If you log in and send "Hi, I'm Riya, log me in," the server says "Welcome!" — but the very next request you send, the server has no memory of who you are anymore. It's like it has goldfish memory.

So how do login sessions actually work across multiple requests? The answer: **every single request has to carry its own proof of identity along with it.** This usually rides along as a cookie or a token (like a JWT) sitting in the request headers. The server isn't remembering you from before — it's re-checking your ID card fresh, every single time.

This is genuinely a common interview question: *"HTTP is stateless — so how do login sessions work?"* And the answer above is exactly it.

## 4. HTTP vs HTTPS

|                   | HTTP              |    HTTPS        |
|------------------ |-------------------|-----------------|
| Security         | none — plain text  | encrypted (TLS) |
| Port             | 80                 | 443             |
| Use for logins?  | never              | always          |

Sending a password over plain HTTP is like writing it on a postcard — anyone handling the mail along the way can read it. HTTPS seals it in a locked envelope instead.

A quick note on ports: think of a port as a numbered door on one building. One server can run many services, and each one answers on its own door — `:80` for http, `:443` for https, and `:3000` is the classic door Node dev servers listen on locally.

## 5. APIs, and the full round trip to a database

Quick recap : an API is the waiter between you (the client) and the kitchen (the server) — you never walk into the kitchen yourself. But there's one more stop in the chain worth adding: the kitchen also has a **pantry**, which is the database. So the full round trip actually looks like:

```
CLIENT → API → SERVER → DATABASE
                  ↓
CLIENT ← API ← SERVER ← DATABASE (data comes back as JSON)
```

This exact round trip happens for every single tap you make in every app you use — scrolling Instagram, checking WhatsApp delivery ticks, searching a song on Spotify, all of it is this same request → route → query → response cycle repeating constantly.

## 6. Resources — the "nouns" REST is built around

A **resource** is any noun your app cares about — a Student, an Order, a Book, a Payment. A simple test: if you can put "a," "an," or "the" in front of it, it can probably be a resource.

REST gives every resource its own URL:
```
/students            → all students
/students/99         → one specific student, id 99
/students/99/courses → that student's courses
```

The rule to remember: **nouns live in the URL, verbs live in the method.** You should never see something like `/getStudents` — the URL says *which* thing you want, and the HTTP method says *what* you want to do with it.

## 7. CRUD — only four things ever happen to data

No matter how complex an app looks, every feature boils down to one of exactly four operations:

| Operation  | HTTP Method | SQL    |        Example                   |
|------------|-------------|--------|----------------------------------|
| Create     | POST        | INSERT | `POST /students` — new admission |
| Read       | GET         | SELECT | `GET /students/99`               |
| Update     | PUT / PATCH | UPDATE | `PATCH /students/99` — fix marks |
| Delete     | DELETE      | DELETE | `DELETE /students/99`            |

Every app you can think of — a college portal, a library system, Amazon, Instagram — is really just CRUD operations on different resources, dressed up with a nice UI on top.

## 8. The five HTTP methods, one at a time

### GET — "give me data"
Purpose: fetch data, never changes anything. Think of it like reading a library book — you look, you don't write in it.
```js
app.get('/students/:id', (req, res) => {
  const s = db.find(req.params.id);
  res.json(s); // 200 OK
});
```
Common mistake: sending a body with GET, or using GET to trigger a change (a web crawler could accidentally click a "delete" link built as a GET request!).

### POST — "create a new one"
Purpose: create a brand-new resource. The server decides the new id, not you. URL points at the *collection*, e.g. `/students`, with no id yet.
```js
app.post('/students', (req, res) => {
  const s = db.create(req.body);
  res.status(201).json(s); // 201 Created, not 200!
});
```
Common mistakes: POSTing to `/students/99` to "create," forgetting the `Content-Type` header, or assuming POST is safe to repeat — sending the same POST twice (double-clicking submit) can create two records.

### PUT — "replace the whole thing"
Purpose: replace the entire resource with exactly what you send. Like swapping out a whole SIM card — the old one is completely gone.
```js
app.put('/students/:id', (req, res) => {
  db.replace(req.params.id, req.body);
  res.json(req.body); // 200
});
```
Common mistake: sending only one field with PUT — every field you *don't* send gets wiped out/nulled, because PUT means total replacement. **PUT is idempotent** — sending the exact same request 10 times gives the same end result, so it's safe to retry.

### PATCH — "fix just this one part"
Purpose: update only the fields you send, leave everything else untouched. Like patching a puncture instead of replacing the whole tyre.
```js
app.patch('/students/:id', (req, res) => {
  const s = db.merge(req.params.id, req.body);
  res.json(s); // 200
});
```
Common mistake: using PUT when you actually meant PATCH — that silently wipes fields you didn't intend to touch.

### DELETE — "remove it"
Purpose: remove the resource at that exact URL. Usually sent with no body at all.
```js
app.delete('/students/:id', (req, res) => {
  db.remove(req.params.id);
  res.status(204).end(); // No Content
});
```
**DELETE is idempotent** — deleting the same thing twice still leaves it gone; the second call just returns a 404 instead of doing anything harmful. Common mistake: `DELETE /students` without an id wipes the *entire* collection.

## 9. Method cheat table

*Safe = never changes data. Idempotent = repeating it gives the same result.*

| Method | Safe? | Idempotent? | Has body? | Success code |
|--------|-------|-------------|-----------|--------------|
| GET    | yes   | yes         | no        | 200          |
| POST   | no    | no          | yes       | 201          |
| PUT    | no    | yes         | yes       | 200          |
| PATCH  | no    | usually     | yes       | 200          |
| DELETE | no    | yes         | rarely    | 204          |

A helpful comparison: idempotent is like a lift button — pressing it 5 times still only sends you to one floor. POST is like a doorbell — press it 5 times, it rings 5 times.

## 10. Breaking down a URL piece by piece

```
https://college.com:443/students/99?course=bca#marks
```

| Piece         | Name             |        What it means                                    |
|---------------|------------------|---------------------------------------------------------|
| `https`       | Protocol         | how to travel (encrypted, in this case)                 |
| `college.com` | Host             | which server/building to reach                          |
| `:443`        | Port             | which door on that server (hidden by default for https) |
| `/students`   | Path → Resource  | which "shelf" — the noun                                |
| `/99`         | ID               | exactly which item on that shelf                        |
| `?course=bca` | Query params     | extra filters                                           |
| `#marks`      | Fragment         | scroll position in the page — this part never even gets |
|               |                  |     sent to the server                                  |



## 11. Anatomy of a request — what's actually inside the envelope

```
1. Start line:  POST /students HTTP/1.1
2. Headers:     Host: college.com
                Content-Type: application/json
                Authorization: Bearer eyJhbG...
                Cookie: session=abc123
3. (blank line — separates headers from body)
4. Body:        { "name": "Divya", "course": "BCA" }
```

GET and DELETE requests usually skip the body entirely — they're just the envelope, no parcel inside.

Headers worth knowing:
- **Content-Type** — "here's the format of what I'm sending you" (e.g. `application/json`)
- **Accept** — "here's the format I want back"
- **Authorization** — the ID card, usually `Bearer <token>`
- **Cookie** — small notes the server gave you earlier, sent back automatically
- **Host** — which website on this server (one server can host many)
- **User-Agent** — identifies the browser/client making the request

Classic bug: sending a POST with a JSON body but forgetting the `Content-Type: application/json` header — the server won't know how to parse it, and `req.body` ends up `undefined`.

## 12. Anatomy of a response

```
1. Status line: HTTP/1.1 200 OK
2. Headers:     Content-Type: application/json
                Content-Length: 58
                Set-Cookie: session=abc123
                Cache-Control: max-age=3600
3. Body:        { "id": 99, "name": "Riya", "course": "BCA" }
```

A good habit for reading any response: look at three things in order — the **status code** (did it work?), the **Content-Type** (what format came back?), 
then the **body** (the actual data).

Why JSON specifically became the standard: it's what JavaScript objects natively look like, so Node reads it without extra work; it's human-readable; and it's noticeably lighter than XML (`{"name":"Riya"}` vs `<name>Riya</name>`).

## 13. Status codes — reading the server's "mood"

- **1xx — hold on** (informational, rarely seen day-to-day)
- **2xx — here you go** (success)
- **3xx — go over there** (redirection)
- **4xx — your fault** (client made a mistake)
- **5xx — my fault** (server made a mistake, only the dev can fix it)

**2xx in practice:**
- `200 OK` — the everyday success response
- `201 Created` — specifically after a successful POST (not 200!)
- `202 Accepted` — "got your request, processing it later" (queued jobs)
- `204 No Content` — success, nothing to send back (perfect fit after DELETE)

**3xx in practice:**
- `301 Moved Permanently` — the resource's address changed for good
- `302 Found` — temporarily at a different location
- `304 Not Modified` — "your cached copy is still fresh, use that" (saves data)

**4xx in practice:**
- `400 Bad Request` — malformed request, broken JSON, missing fields
- `401 Unauthorized` — really means *un-authenticated* — no valid ID at all
- `403 Forbidden` — you *are* identified, but you're still not allowed
- `404 Not Found` — no such resource at that URL
- `409 Conflict` — e.g. trying to register a duplicate email
- `422 Unprocessable Entity` — valid JSON, but nonsensical values (like `"age": -5`)
- `429 Too Many Requests` — you're calling the API too fast

**5xx in practice:**
- `500 Internal Server Error` — an uncaught exception in your own server code — always check the server logs first
- `502 Bad Gateway` — the middleman got a garbage response from your app behind it
- `503 Service Unavailable` — server overloaded or down for maintenance
- `504 Gateway Timeout` — the middleman waited on a slow backend/DB and never got an answer

The important distinction to remember: `401` = *who even are you?* while `403` = *I know exactly who you are — still no.*



## 14. Versioning and a quick consistency checklist

APIs change over time, but you can't just break every app that's already using the old shape. That's why APIs get versioned in the URL:
```
/api/v1/students   ← old apps keep working against this
/api/v2/students    ← new shape ships here instead
```

A quick checklist for writing consistent REST APIs:
- Plural nouns everywhere (`/students`, `/orders`)
- Lowercase, kebab-case paths (`/course-modules`)
- Filters go in the query string (`/students?year=2&sort=name`)
- Version prefix on everything (`/api/v1/...`)
- Correct status codes used consistently (`201` for created, `204` for deleted, `404` for missing, `422` for invalid data)
- The same JSON error shape returned everywhere, not a different format per endpoint

## 15. The full journey of one click

It's worth tracing what actually happens between hitting Enter on a URL and seeing a page render:

```
browser → DNS lookup → TCP + TLS handshake → API gateway (auth/rate-limit)
→ Node/Express matches the route → controller validates the request
→ service layer runs the business logic → database query runs
→ JSON response returns → browser renders it
```

In one breath: **browser → DNS → internet → API → Node → controller → service → DB → JSON → browser** — and this entire chain typically completes in around 200ms.

Where things can fail at each stop is worth knowing too: DNS failing means "server not found" before HTTP even starts; 
the API gateway rejecting you shows up as 401/403/429;  
a controller rejecting bad input shows up as 400/422;
a crash in your own code shows up as 500; 
and a slow database behind a proxy shows up as a 504 timeout.

## Big takeaways

- HTTP is a stateless protocol — every request has to prove identity on its own, usually via a cookie or token
- URLs are nouns (plural), HTTP methods are verbs — never mix the two
- GET/PUT/DELETE are idempotent (safe to repeat), POST is not
- Always check the status code family first — 2xx success, 4xx your bug, 5xx server's bug
- `201` after POST, `204` after DELETE — using `200` everywhere is a common beginner habit worth breaking

---
