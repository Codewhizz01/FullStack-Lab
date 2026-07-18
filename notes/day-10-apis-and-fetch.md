-------------------------------- Day-10 TASK Working with APIs & Fetch----------------------------------





### 1. What is an API?
- API stands for Application Programming Interface — a way for two programs to talk to each other.
- **Restaurant analogy:** the client (you) orders from a menu, the API is the waiter carrying the request back and forth, and the server is the kitchen actually preparing the response.
- The client doesn't need to know how the kitchen works internally — it just needs to know what to ask for and what it'll get back.

### 2. Client and Server
- The client always speaks first — nothing happens until a request is sent.
- One request gives one response — a single round trip, not an open conversation.
- The client is typically the browser/frontend/app; the server holds the data and logic.

### 3. Anatomy of a Request
A request has four parts:
- **Method** — what action to perform (GET, POST, etc.)
- **URL** — which resource to act on
- **Headers** — metadata about the request (content type, auth token, etc.)
- **Body** — the actual data being sent (not used in GET)

### 4. Anatomy of a Response
A response has three parts:
- **Status** — a code telling you if it worked (200, 404, 500...)
- **Headers** — metadata about the response
- **Body** — the actual data returned

### 5. Read the Status Code First
- A body can lie — an API might return a "success-looking" body even when something went wrong, or an error page even when data exists.
- The status code is the first, most reliable signal of what actually happened. Always check it before trusting the body.

### 6. JSON as the Data Format
- JSON (JavaScript Object Notation) is the standard format for sending/receiving data between client and server.
- `response.json()` — parses an incoming JSON response into a usable JS object.
- `JSON.stringify()` — converts a JS object into a JSON string to send in a request body.

### 7. The Five HTTP Methods
| Method | Purpose |
|---|---|
| GET | read data |
| POST | create new data |
| PUT | replace existing data entirely |
| PATCH | update part of existing data |
| DELETE | remove data |

### 8. GET vs POST
- **GET:** parameters go in the URL (e.g. `/products?id=5`). No body. Safe to repeat — calling it multiple times doesn't change anything.
- **POST:** data goes in the request body, not the URL. Not safe to repeat — calling it twice (e.g. submitting a payment) can create duplicate actions/data.

### 9. Status Code Families
| Range | Meaning |
|---|---|
| 1xx | Informational |
| 2xx | Success |
| 3xx | Redirection |
| 4xx | Client error (you made a mistake) |
| 5xx | Server error (server made a mistake) |

**Everyday codes:**
- `200` OK — request succeeded
- `201` Created — new resource created successfully
- `204` No Content — success, but nothing to return
- `400` Bad Request — malformed request
- `401` Unauthorized — missing/invalid authentication
- `403` Forbidden — authenticated but not allowed
- `404` Not Found — resource doesn't exist
- `429` Too Many Requests — rate limit hit
- `500` Internal Server Error — something broke on the server

### 10. REST URL Design
- **URLs are nouns, methods are verbs.**
- Correct: `GET /students`, `POST /students`, `DELETE /students/5`
- Incorrect: `GET /getAllStudents`, `POST /createStudent` — the verb belongs in the method, not the URL.

### 11. Using fetch()
- `.then()` chaining is the older style:

```javascript
fetch(url)
  .then(res => res.json())
  .then(data => console.log(data));
  ```

  async/await is the cleaner, more readable style:
  ```
  const res = await fetch(url);
const data = await res.json();
console.log(data);
```

- Why await twice: the first await waits for the network response to arrive (headers, status). The second await waits for the body to be read and parsed into JSON — this is also an async operation, since the body can stream in over time.

12. Sending Data with fetch
```
const res = await fetch(url, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Rahul", marks: 92 })
});
```
- method specifies the HTTP verb.
- headers tells the server what format the body is in.
- body must be a string, so the object is converted using JSON.stringify().

13. Error Handling with fetch

- fetch() does not throw an error on a 404 or 500 — it only throws on a true network failure (e.g. no internet, DNS failure).
- This means a "failed" request (like 404) still resolves successfully as far as fetch() is concerned — the mistake is assuming success just because the code didn't throw.
- Always check res.ok (or res.status) manually:
```
const res = await fetch(url);
if (!res.ok) {
  throw new Error(`Request failed: ${res.status}`);
}
const data = await res.json();
```
14. Auth Basics

- API keys and Bearer tokens are common ways to prove who's making the request.
- Bearer tokens are typically sent in the Authorization header: Authorization: Bearer <token>
- Secret keys belong in a .env file — never hardcoded in frontend code, and never committed to git (frontend code is visible to anyone, and git history is hard to fully erase).

15. The Other Side — Express (Server)
- app.get(path, handler) — defines a route that responds to GET requests.
- app.post(path, handler) — defines a route that responds to POST requests.
- res.status(code).json(data) — sends back a status code and a JSON body together.
- This is the server-side mirror of everything the client does with fetch().
16. Live Demo — PokeAPI
- Practiced calling a real public API (PokeAPI) directly in the browser console using fetch().
- Observed the real request/response cycle: URL, status code, and JSON body returned live.

## Takeaways
- Read the status code first — a body alone can be misleading.
- fetch() only throws on network failure, so always check res.ok yourself.
- URLs are nouns, methods are verbs.
- Secret keys never go in frontend code or git — they belong in .env.