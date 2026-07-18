------------------------------Day-11 TASK  HTTP Standards & REST APIs---------------------------------------


### 1. What is HTTP?
- HTTP = **H**yper**T**ext **T**ransfer **P**rotocol — the language browsers and servers use to talk to each other.
- Born in 1989–91 at CERN, invented by Tim Berners-Lee (along with HTML and the first browser) to share physics papers.
- A protocol is just a rule book — it defines how a message is formatted, which methods are allowed (GET, POST...), and how results are reported (status codes). It's rules, not a program — like traffic rules.
- **Why it exists:** without a shared rule book, a Mac running Chrome and a Linux server would have no common way to understand each other's messages. HTTP is the agreed common language.

### 2. Client and Server
- **Client** = customer (browser / app / Postman) — always asks.
- **Server** = shopkeeper (Node.js, awake 24×7) — always answers.
- Cycle: ask → think → answer → forget.
- **Key rule:** the server never starts the conversation — it only replies.

### 3. HTTP is Stateless
- The server forgets the client after every single request — it has no memory of previous requests (goldfish memory).
- Example: "Hi, I am Riya, log me in" → "Welcome!" — then on the very next request, the server has already forgotten who Riya is.
- **How apps stay logged in anyway:** every request carries its own identity — a cookie or a token (JWT) rides along in the headers. The server checks the pass each time, not its memory.
- **Interview answer:** "HTTP is stateless — how do sessions work then?" → because identity travels with every request via cookies/tokens, not because the server remembers you.

### 4. HTTP vs HTTPS
| | HTTP | HTTPS |
|---|---|---|
| Security | none — plain text (like a postcard, anyone can read it) | TLS encryption (sealed and locked) |
| Port | 80 | 443 |
| Use for logins? | never | always |

- **Port** = a numbered door — one computer, many doors, each service has its own (`:80` http, `:443` https, `:3000` dev servers).
- `https://x.com` is really `https://x.com:443` — the browser just hides the port.
- Node dev servers commonly use `:3000` and `:8080`.

### 5. What is an API?
- Application Programming Interface — a middleman that lets two programs talk. The client never touches the database directly.
- **Restaurant analogy:** you (client) order → waiter (API) carries the message → kitchen (server) does the work → pantry (database) stores everything → food comes back as the response (JSON).
- **The round trip:** Client → API → Server → Database, then data flows back up as JSON. This same trip happens for every tap you make in any app.
- **Real examples:** Google Maps (route API), Instagram (feed API, like = POST /likes), WhatsApp (message API + delivery receipts), YouTube (video list + stream API), UPI/Paytm (bank APIs settling payments), Spotify (search + stream API), Netflix (recommendation API), Weather apps (`GET api.weather.com/pune` → `{ "temp": 31 }`).
- You use hundreds of APIs before breakfast without realizing it.

### 6. Resources — the "Things"
- A resource = any **noun** your app cares about: Student, Teacher, Product, Order, Book, Payment, Customer, Course, Employee.
- Quick test: if you can put "a / an / the" in front of it, it can be a resource.
- A REST resource is a noun with its own URL:
  - All students → `/students`
  - One student (roll 99) → `/students/99`
  - Their courses → `/students/99/courses`
- **Golden rule:** nouns live in the URL, verbs live in the method — never `/getStudents`.

### 7. CRUD — Only 4 Things Ever Happen to Data
| CRUD | HTTP | SQL | Example |
|---|---|---|---|
| Create | POST | INSERT | `POST /students` — new admission |
| Read | GET | SELECT | `GET /students/99` |
| Update | PUT / PATCH | UPDATE | `PATCH /students/99` — fix marks |
| Delete | DELETE | DELETE | `DELETE /students/99` — TC issued |

- Every app — Instagram to ISRO — is just CRUD on resources, dressed up nicely.
- **Examples across domains:**
  - College: admit (C), view list (R), update marks (U), remove student (D)
  - Library: add book (C), search catalogue (R), mark issued (U), remove torn book (D)
  
  

### 8. GET — "Give Me Data"
- Purpose: fetch data. Never changes anything.
- Analogy: reading a library book — look, don't write in it.
- Used for: lists, details, search results, feeds.



```javascript
app.get('/students/:id', (req, res) => {
  const s = db.find(req.params.id);
  res.json(s); // 200 OK
});
```


9. POST — "Push Something New"

- Purpose: create a NEW resource. The server assigns the id.
- Used for: signup, new order, new post, sending a message.
```
 POST /students HTTP/1.1
Content-Type: application/json

{ "name":"Aman", "course":"BCA" }
```
- URL is the collection (/students) — no id yet, since the server generates it.
```
 app.post('/students', (req, res) => {
  const s = db.create(req.body);
  res.status(201).json(s);
});
```
 - Success code is 201 Created, not 200. 

10. PUT — "Replace the Whole Thing"

- Purpose: replace the resource ENTIRELY with what's sent.
- Used for: "save profile" forms that send every field.
```
 PUT /students/99 HTTP/1.1
Content-Type: application/json

{ "name":"Riya S", "course":"MCA", "year": 2 }
```
```
 app.put('/students/:id', (req, res) => {
  db.replace(req.params.id, req.body);
  res.json(req.body); // 200
});
```
 - Idempotent: PUT with the same body 10 times → same result every time. Safe to retry. Trick: PUT = put a new one in its place.

11. PATCH — "Fix Just This Part"

- Purpose: update SOME fields, leave the rest untouched.
- Used for: change password only, edit caption, mark as read.
```
 PATCH /students/99 HTTP/1.1
Content-Type: application/json

{ "course":"MCA" }
```
```
 app.patch('/students/:id', (req, res) => {
  const s = db.merge(req.params.id, req.body);
  res.json(s); // 200
});
```
- PUT = repaint the whole wall. PATCH = touch up one spot. 
12. DELETE - "destroy it"

- Purpose: remove the resource at that URL.
- Used for: delete post, cancel booking, remove cart item.
```
DELETE /students/99 HTTP/1.1
Authorization: Bearer <token>
```
 - Usually no body at all.
```
 app.delete('/students/:id', (req, res) => {
  db.remove(req.params.id);
  res.status(204).end(); // No Content
});
```
- Idempotent: delete twice → still gone. Second call just returns 404.

## 13. Method Cheat Table
Safe = never changes data. Idempotent = repeating it gives the same result

| Method | Safe? | Idempotent? | Body? | Success Code |
|---|---|---|---|---|
| GET | ✓ | ✓ | ✗ | 200 |
| POST | ✗ | ✗ | ✓ | 201 |
| PUT | ✗ | ✓ | ✓ | 200 |
| PATCH | ✗ | usually | ✓ | 200 |
| DELETE | ✗ | ✓ | rare | 204 |

## 14. URL Anatomy

| Part | Name | Meaning |
|---|---|---|
| `https` | Protocol | how to travel (secure!) |
| `college.com` | Host | which building — DNS finds its IP |
| `:443` | Port | which door — hidden by default |
| `/students` | Path → Resource | which shelf — the noun |
| `/99` | ID | exactly which one |
| `?course=bca` | Query Params | extra filters (`a=1&b=2` style) |
| `#marks` | Fragment | scroll position — never sent to the server! |

## 15. The HTTP Request 

| Header | Meaning |
|---|---|
| `Content-Type` | "my body is JSON" — how the server should read this request |
| `Accept` | "please reply in JSON" — what the client wants back |
| `Authorization` | the ID card — `Bearer <JWT token>` — who is asking |
| `Cookie` | small notes the server gave earlier, sent back every time |
| `Host` | which website on this server — one IP, many sites |
| `User-Agent` | "I am Chrome on Windows" — who's knocking |

## 16. The HTTP Response — the Reply Packet
Three parts:
1. Status line — HTTP/1.1 200 OK (version + code + reason)
2. Headers — Content-Type, Content-Length, Set-Cookie, Cache-Control, etc.
3. Body — the actual JSON data

## Why JSON:
- JavaScript Object Notation — Node speaks it natively.
- Human-readable, machine-parseable.
Lighter than XML (<name>Riya</name> vs "name":"Riya").
- Every language can parse it.

## Reading a response in 3 looks:

- Status code → did it work?
- Content-Type → what came back?
- Body → the actual goods

## 17. Status Codes — the Server's Mood

// 1xx — Informational ("hold on...")
- 100 Continue — "go ahead, send the rest" — used before big uploads.
- 101 Switching Protocols — HTTP → WebSocket upgrade (used by chat apps).

// 2xx — Success ("here you go ✓")
- 200 OK — done, here it is. The everyday hero.
- 201 Created — new thing made (POST should return this, not 200).
- 202 Accepted — got your order, cooking later (queued jobs, email sending).
- 204 No Content — done, nothing to show. Perfect after DELETE, empty body.

// 3xx — Redirection ("go there →")
- 301 Moved Permanently — shop shifted forever, update your address book. SEO value moves too.
- 302 Found — temporarily at a different counter today.
- 304 Not Modified — "your saved copy is still fresh, use it" (works with ETag, saves data).
- 307 Temporary Redirect — like 302, but method must stay the same.
- 308 Permanent Redirect — like 301, method preserved.

// 4xx — Client Error ("YOU messed up")
- 400 Bad Request — broken JSON, missing fields.
- 401 Unauthorized — no ID card, not logged in (really means un-authenticated).
- 403 Forbidden — ID seen, still no — logged in ≠ allowed.
- 404 Not Found — no such resource (e.g. a URL typo).
- 405 Method Not Allowed — right door, wrong action.
- 406 Not Acceptable — client only accepts a format the server doesn't provide.
- 408 Request Timeout — client took too long.
- 409 Conflict — duplicate entry, e.g. two bookings for the same seat.
- 410 Gone — existed once, deleted forever (stronger than 404).
- 415 Unsupported Media Type — a file type the server can't process.
- 422 Unprocessable Entity — valid JSON, but nonsense values (e.g. "age": -5).
- 429 Too Many Requests — rate limit hit, slow down.
// 5xx — Server Error ("MY fault, sorry")
- 500 Internal Server Error — an uncaught exception in the server code — always check server logs.
- 501 Not Implemented — "we don't serve that dish yet" — method not built.
- 502 Bad Gateway — the middleman got garbage back — Nginx is fine, but the Node app behind it crashed.
- 503 Service Unavailable — "closed for maintenance" — server overloaded or deploying.
- 504 Gateway Timeout — the middleman waited, but the kitchen never answered — e.g. a slow DB query behind a proxy.

// 5-line summary:
- 1xx → hold on
- 2xx → here you go ✓
- 3xx → go there →
- 4xx → you messed up
- 5xx → I messed up (server's fault — the user can't fix it)

## 19. Versioning & Consistency

- Versioning — never break old apps:
  - /api/v1/students — old apps keep working
  -  /api/v2/students — new shape ships here

- Safe & idempotent recap:
  - Safe methods: GET (+ HEAD, OPTIONS) — window shopping: look, never touch.
  - Idempotent: GET, PUT, DELETE — like an elevator button.
  - NOT idempotent: POST — like a doorbell, every press counts.
  - Networks retry failed requests — retrying a POST can double-charge a payment.


## 20. HTTP Facts — History & Plumbing
- Born at CERN: Tim Berners-Lee invented HTTP (+ HTML + the first browser) in 1989–91 to share physics papers.
- HTTP/1.1 (1997): added keep-alive — reuse one TCP connection for many requests instead of reconnecting every time. Still everywhere.
- HTTP/2 (2015): multiplexing — many requests travel through one connection at once.
- HTTP/3 + QUIC: runs on UDP, not TCP. Google's QUIC protocol is faster on flaky mobile networks — YouTube already uses it.
- DNS comes first: the internet's phonebook — college.com → 142.250.4.1. No DNS answer means no HTTP at all.
- TCP handshake + TLS: SYN → "hello?", SYN-ACK ← "hello! ready", ACK → "ok, talking now" — then TLS swaps keys (the S in HTTPS) — all before any HTTP byte moves.

## 21. HTTP Facts — Speed & Identity
- Why browsers cache: fetching again is slow and costly. Cache-Control: max-age=3600 → "keep this for 1 hour, don't ask me again."
- ETag = fingerprint: server tags data (ETag "v42"). Next time the client asks "still v42?" the server can reply 304 Not Modified with zero data resent.
- Cookies vs sessions vs JWT:
Cookie = a note in your pocket (browser-side)
Session = a register at reception (server remembers)
JWT = a tamper-proof wristband — the server checks the seal but remembers nothing. This is what cures statelessness.
- Compression: Accept-Encoding: gzip, br — the server squeezes the response. Gzip is the classic; Brotli (Google's) is roughly 20% smaller. Free speed.
- CDN = local kirana (corner store): copies of files stored near the user (Mumbai, Delhi, Chennai edges). Netflix video streams from your city, not California.

## 22. One Click, Full Journey

- Browser — types URL, hits Enter
- DNS — phonebook lookup → IP address
- Internet — TCP + TLS handshake, network hops
- API gate — auth + rate limiting
- Node server — Express matches the route
- Controller — validates the request (traffic police)
- Service — business logic (the chef)
- Database — SELECT * FROM students
- JSON returns — 200 OK + gzip compression
- Browser paints — JSON becomes visible UI, all in ~200ms


- Errors by stop:

  - DNS fails → "server not found"
  - Gate says no → 401 / 403 / 429
  - Controller rejects → 400 / 422
  - Service crashes → 500
  - DB slow → 504

## Takeaways
- HTTP is stateless — identity travels with every request via tokens/cookies, the server itself remembers nothing.
- URLs are nouns (plural), methods are verbs — never put a verb in the URL.
- POST creates (201), PUT replaces fully, PATCH updates partially, DELETE removes (204) — know which is idempotent and which isn't.
- Always read the status code first, then Content-Type, then the body.
- 4xx = client's mistake, 5xx = server's mistake.
