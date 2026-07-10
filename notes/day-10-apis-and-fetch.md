-------------------------------- Day-10 TASK Working with APIs & Fetch----------------------------------

## 1. What an API actually is

The easiest way to understand an API is the restaurant analogy. I'm the client, sitting at a table. I never walk into the kitchen myself and start cooking — instead, I tell the waiter what I want, the waiter takes that order to the kitchen (the server), and brings the finished dish back to me. The API is the waiter: it's the messenger that carries my request to the server and carries the server's response back to me, without me ever needing to know how the kitchen actually works internally.

## 2. Client and server — who talks first

The client **always** speaks first. The server never reaches out on its own — it just sits and waits for a request to arrive, then responds to it. And it's always a strict pair: **one request gives exactly one response**, never more, never less.

## 3. Anatomy of a request and a response

Every **request** is made up of:
- **Method** — what action you want (GET, POST, etc.)
- **URL** — where you're sending it
- **Headers** — metadata about the request (like what format the body is in)
- **Body** — the actual data you're sending, if any

Every **response** is made up of:
- **Status** — a code telling you what happened
- **Headers** — metadata about the response
- **Body** — the actual data sent back

The golden rule here: **always read the status code first.** A response body can say something like `"message": "success"` in its text even while the actual status code says otherwise, or vice versa. The status code is the one part of the response you can trust — the body can lie or be misleading.

## 4. JSON — the common language of APIs

Almost every API speaks in JSON. Two functions handle the translation between JSON and JavaScript:

```js
const data = await response.json(); // parses incoming JSON into a JS object I can use
const body = JSON.stringify(data);  // converts a JS object into JSON text to send out
```

You'll use `.json()` constantly when *receiving* data, and `JSON.stringify()` whenever you're *sending* data in a request body.

## 5. The five HTTP methods

| Method |What it does                     |
|--------|---------------------------------|
| GET    | Read data                       |
| POST   | Create new data                 |
| PUT    | Replace existing data entirely  |
| PATCH  | Update part of existing data    |
| DELETE | Remove data                     |

## 6. GET vs POST — why they're different

GET sends any parameters directly in the URL, which means they're visible, cacheable, and bookmarkable. POST instead sends its data hidden inside the request body, not the URL.

There's also a safety difference: GET is "safe to repeat" — refreshing a GET request endlessly doesn't cause harm. POST is **not** safe to repeat — sending the same POST request twice (say, by double-clicking submit) can create two duplicate records on the server.

## 7. Status code families

Status codes are grouped into five families by their first digit:

- **1xx** — informational (rarely seen day-to-day)
- **2xx** — success — `200 OK`, `201 Created`, `204 No Content`
- **3xx** — redirection
- **4xx** — the client made a mistake — `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `429 Too Many Requests`
- **5xx** — the server made a mistake — `500 Internal Server Error`

A quick way to remember it: 4xx means *I* did something wrong, 5xx means *they* did something wrong.

## 8. REST URL design — URLs are nouns, methods are verbs

A well-designed REST API doesn't put the action inside the URL. The URL should just name the *resource* (a noun), and the HTTP method supplies the *action* (the verb):

```
GET    /users        → get the list of users
POST   /users        → create a new user
GET    /users/5      → get user with id 5
PUT    /users/5      → replace user 5 entirely
DELETE /users/5      → delete user 5
```

Notice there's no `/getUsers` or `/createUser` anywhere — the URL stays a plain noun, and the method does the work of describing the action.

## 9. Using fetch() — two ways to write it

**The `.then()` style:**
```js
fetch(url)
  .then(res => res.json())
  .then(data => console.log(data));
```

**The async/await style — cleaner and easier to read:**
```js
async function getData() {
  const res = await fetch(url);   // step 1: wait for the response to arrive
  const data = await res.json();  // step 2: wait for the body to finish parsing
  console.log(data);
}
```

Notice there are **two** `await`s here, and that's not a typo — they're two genuinely separate asynchronous steps. 
The first `await` waits for the server to respond at all.
 The second `await` waits for that response's body to actually finish being parsed into usable JSON. Skipping either one means you're working with a half-finished Promise instead of real data.

## 10. Sending data with fetch

To send data (like creating something with POST), you pass a second argument to `fetch` with the method, headers, and body:

```js
fetch(url, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Divya" })
});
```

The `Content-Type` header tells the server "the body I'm sending you is JSON," and `JSON.stringify()` turns the JS object into the JSON text format the body actually needs to be.

## 11. The error-handling gotcha that trips everyone up

This is the one thing to remember above all else: **`fetch()` only throws an error on a genuine network failure** — no internet connection, DNS failure, that sort of thing. It does **not** throw an error just because the server responded with a `404` or a `500`. As far as `fetch` is concerned, receiving *any* response at all — even an error response — counts as success.

That means if you don't check for this yourself, a failed request can silently look like it worked:

```js
async function getData() {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Request failed with status ${res.status}`);
  }
  const data = await res.json();
  return data;
}
```

`res.ok` is `true` only for status codes in the 200–299 range, so checking it manually is the only way to actually catch a 404 or 500 as an error in your code.

## 12. Auth basics — API keys and Bearer tokens

Most real APIs require some form of authentication so the server knows who's calling. Two common approaches:

- **API keys** — a unique string identifying your app/account, usually sent as a header
- **Bearer tokens** — a token (often from a login) sent in the `Authorization` header like `Authorization: Bearer <token>`

The critical rule here: these keys belong in a `.env` file, and that `.env` file should never be committed to git, and never hardcoded directly into frontend code. Frontend code is visible to anyone who opens dev tools, and anything committed to a public git repo is visible to anyone on the internet — either one is an instant leak of a secret key.

## 13. The other side of the conversation — Express

Everything above was from the client's perspective. On the server side (using Express, for example), the same request/response cycle looks like this:

```js
app.get("/users", (req, res) => {
  res.status(200).json(users);
});

app.post("/users", (req, res) => {
  const newUser = req.body;
  users.push(newUser);
  res.status(201).json(newUser);
});
```

`app.get()` / `app.post()` define which URL + method combination triggers which function, and `res.status().json()` is how the server sends back a status code and a JSON body together — the exact same two things I was told to check for on the client side.

## 14. Live demo — PokeAPI

Watched real requests fire against the PokeAPI live in the browser console. Seeing the request go out and the actual response object come back — with its real status code, headers, and body — made the theory click much faster than just reading about it.

## Key takeaways

- Always read the status code first — the body can be misleading
- `fetch()` only throws on a network failure, so always check `res.ok` manually
- URLs should be nouns, methods are the verbs
- Secret keys never belong in frontend code or in git

---






