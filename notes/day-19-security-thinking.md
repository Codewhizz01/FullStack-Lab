# Day 20 — Backend and security thinking


## 1. The CBSE portal, and how marks could be changed


**What actually happened**, from the news reports: in 2026, a 19-year-old named Nisarga Adhikary, who had just finished class 12, looked at CBSE's On-Screen Marking (OSM) portal — the site where teachers evaluate scanned answer sheets online. He found a set of basic security holes and reported them. CBSE first denied any breach, saying it was only a test site, then later acknowledged vulnerabilities in the provider's portal and ordered an audit. He did not do it for gain — he reported it, which is exactly what an ethical hacker does.

**The flaws, and what each one teaches:**

| The flaw | The backend lesson |
|---|---|
| A master password was written into the source code, visible in the browser. | Never put secrets in code that reaches the browser. Anyone can read it with Ctrl+F. |
| The OTP check could be bypassed. | A security check the client can skip is not a security check. |
| Password reset didn't ask for the old password — any user ID plus any gibberish reset it. | The server must verify every step. Never trust that the client did the checking. |
| An examiner's identity could be edited through browser storage, letting you impersonate a teacher. | Never trust data from the browser. The server decides who you are, not the client. |
| Internal dashboards were open with no login. | Every private route needs an access-control check, on the server, every time. |

> **The one-sentence answer.** The marks were changeable because the system trusted the browser. Passwords, identities, and checks all lived where the user could see and edit them. His own words: *"These aren't advanced defences. They're the basics."* A backend developer's first rule is the opposite of what CBSE did: **never trust the client, verify everything on the server.**

###  how this maps to known vulnerability categories

This incident is a textbook example of a few named categories in the industry-standard **OWASP Top 10** (a list of the most common web app security risks, maintained by the Open Web Application Security Project):

| CBSE flaw | OWASP-style category |
|---|---|
| Secret in client-side source | *Sensitive Data Exposure* / hardcoded credentials |
| Client-side-only OTP check | *Broken Authentication* |
| No old-password requirement on reset | *Broken Authentication* (weak account-recovery flow) |
| Identity editable via browser storage | *Broken Access Control* |
| Open dashboards, no login | *Broken Access Control* |

**Why this matters :** these aren't obscure attacks — "broken access control" has been the #1 category in the OWASP Top 10 for years, precisely because it's this easy to get wrong: a developer builds a feature, forgets to check *who* is allowed to use it on the server, and ships it. This is exactly why, in ResumeFlow, your `authMiddleware` re-checking the JWT on every protected route matters — it's the server-side check that CBSE's portal skipped.

**A useful mental model :** *"The client is a suggestion. The server is the decision."* The browser can suggest "I am examiner #45" — the server has to independently verify that claim (via a signed token, a session, a database lookup) rather than trusting the suggestion.

---

## 2. How 10,000+ records get created in minutes


A human types slowly. To create 10,000 records by hand would take days. So it wasn't done by hand — it was done by a **script**, a small program that repeats an action in a loop, as fast as the server will allow.

> **Script / bot**
> A program that does a repetitive task automatically. A few lines can send the same "create record" request thousands of times in a loop. What takes a person all day takes a script a few seconds. This is not advanced — it's a beginner-level loop pointed at an endpoint that wasn't protected.

**Why a site lets this happen, and how you stop it:**

| What was missing | The fix a backend developer adds |
|---|---|
| No rate limit — the server accepted requests as fast as they came. | **Rate limiting:** cap how many requests one user or IP can make per minute. |
| No bot check on the form. | A CAPTCHA or similar on public create-endpoints. |
| The endpoint was open — no login needed to create records. | Require authentication, so anonymous scripts can't post at all. |
| No validation catching obviously fake or duplicate data. | Server-side validation and duplicate checks that reject junk. |

> **The one-sentence answer.** 10,000 records appeared because a script hit an endpoint that had no rate limit, no bot check, and no login. The lesson: **assume any open endpoint will be hit by a machine, not a polite human** — and put limits in before it happens, not after.

###  what these defences actually look like in Node/Express

Since this is directly relevant to ResumeFlow's Express API, here's what each fix looks like in practice (concept-level, not full implementation):

| Defence | Common tool/approach in Node/Express |
|---|---|
| Rate limiting | `express-rate-limit` middleware — e.g. "max 100 requests per IP per 15 minutes" |
| Bot check | Google reCAPTCHA / hCaptcha on public forms; honeypot fields (a hidden input real users never fill, but bots do) |
| Auth requirement | JWT/session middleware on `POST` routes, so only logged-in users can create records |
| Validation | Libraries like `express-validator` or `joi` to reject malformed/duplicate payloads before they touch the database |

**Why rate limiting is "per IP or per user," not global:** a global limit (e.g. "100 requests per minute total") would let one bot starve out every real user. Per-IP or per-account limits mean the bot only throttles itself.

**A related, slightly deeper concept worth mentioning if asked:** this class of abuse is sometimes called an **enumeration or flooding attack** depending on intent — flooding to overwhelm/pollute data (this case), vs. enumeration to guess valid usernames/emails by testing many values and watching for different responses. Same root cause (no limits on an open endpoint), different goal.

---

## 3. What information is kept in the UI


The UI is the HTML, CSS, and JavaScript that runs in the browser. The single most important fact about it: **everything in it is visible to everyone.** Anyone can right-click, choose "view source," or open dev tools, and read every line of your HTML, CSS, and JavaScript. Nothing there is hidden.

| What lives in the UI | So remember |
|---|---|
| The page structure and text (HTML) | readable by anyone, always |
| The styling (CSS) | readable, harmless, fine to be public |
| The logic that runs in the browser (JavaScript) | fully readable — never hide a secret in it |
| Values stored in the browser (localStorage, cookies) | readable **and editable** by the user |

> **The one-sentence answer.** The UI holds only what you're willing to show the whole world, because the whole world can read it. This is exactly the CBSE mistake from topic 1: a master password sat in the browser code, and browser storage decided who you were. **Rule: never put a secret, a password, or a trust decision in the UI.** The browser is the user's territory, not yours.

### Extra context:

**"Minified" or "bundled" JS is not hidden — it's just harder to read.** A common misconception is that build tools (Webpack, Vite) that compress JS into unreadable-looking code are a security measure. They're not — they're a performance optimization. Anyone can run the minified code through a "beautifier" (prettifies it back into readable code) in seconds. This is worth saying explicitly in your presentation, because it's the exact misconception that trips people up.

**Environment variables and the `VITE_` / `REACT_APP_` prefix trap:** in frontend frameworks, any environment variable exposed to the client build (e.g. prefixed `VITE_...` in Vite, or `REACT_APP_...` in Create React App) gets baked directly into the JS bundle and shipped to every visitor's browser. A very common real-world mistake: putting a real API secret key in a `.env` file thinking it's "hidden" because it's in a `.env` file, not realizing the frontend build process embeds its *value* (not the variable name) into the public bundle.

**Where secrets actually belong instead:** on the server, in environment variables that never get sent to the browser (e.g. your Express `.env`, read only by `process.env` on the backend) — exactly how you're already handling secrets like `JWT_SECRET` in ResumeFlow's backend, not the frontend.

**Cookies vs. localStorage, one nuance worth knowing:** both are readable/editable by the user via DevTools, but they differ in one important way — cookies marked `httpOnly` *cannot* be read by JavaScript (only sent automatically by the browser to the server). This is why some teams store auth tokens in `httpOnly` cookies rather than localStorage: it protects against a JS-based attack called **XSS (Cross-Site Scripting)** stealing the token, even though the user themselves could still see the cookie via browser dev tools/network tab.

---

## 4. Who are ethical hackers


An ethical hacker uses the same skills as an attacker, but **with permission**, to find security holes so they can be fixed before a real attacker uses them. Same knowledge, opposite intent — and crucially, done legally and reported responsibly.

| Ethical hacker | Malicious hacker |
|---|---|
| has permission, or reports responsibly | no permission, hides the act |
| finds a hole and tells the owner to fix it | uses the hole for gain or damage |
| makes systems safer | makes victims |
| paid by companies, bug bounties, security jobs | faces jail |

Nisarga from topic 1 is the live example. He found serious flaws and reported them to the authorities and CERT-In, instead of changing marks for money. That single choice — report it rather than abuse it — is the whole line between the two columns. Companies pay ethical hackers, run bug bounty programs, and hire security teams precisely because they'd rather a friendly researcher find the hole first.

> **The one-sentence answer.** Ethical hackers are the people who find the holes before the criminals do, with permission and in the open — and it's a real, paid career. The skills are the same as an attacker's; **the intent and the legality are what make them ethical.**

### Extra context: the actual career path, in case someone asks

- **"White hat" / "black hat" / "grey hat"** are the common informal labels: white hat = fully authorized; black hat = malicious; grey hat = finds a flaw without permission but reports it rather than exploiting it (legally murky — Nisarga's case sits closer to grey-to-white, since he wasn't formally hired but reported responsibly rather than exploiting it).
- **CERT-In** (Indian Computer Emergency Response Team) is the Indian government's official body for handling cybersecurity incidents and vulnerability reports — mentioned in the source material as one of the places Nisarga reported to. Most countries have an equivalent (e.g. US-CERT in the US).
- **Bug bounty programs** are formal, company-run programs (HackerOne and Bugcrowd are the two biggest platforms) where companies publicly invite researchers to find and report bugs in exchange for cash rewards — this is the legal, structured version of what Nisarga did informally.
- **Common certifications** in this field if the batch asks "how do you become one": CEH (Certified Ethical Hacker), OSCP (Offensive Security Certified Professional) — OSCP in particular is respected because it's hands-on/practical rather than multiple-choice.
- **The legal line in India** is the IT Act, 2000 (specifically Section 43 and 66) — unauthorized access to a computer system is illegal *regardless of intent*, which is exactly why "I was just testing security" is not, by itself, a legal defence; permission or responsible disclosure is what makes it ethical *and* legal.

---

## 5. The coding side: what a clicked link actually reveals


Present the honest version of this, because most people get it wrong. When someone clicks a link you sent, you do **not** get their exact location. You get their **public IP address**, and an IP maps only to a rough area — a city or the internet provider's region — and it's often wrong for mobile data, VPNs, and company networks.

| What people believe | What actually happens |
|---|---|
| a clicked link reveals exact location | it reveals the public IP, which is city/ISP-level at best |
| you can read their local address (`192.168.x.x`) | you cannot — it stays inside their router and never reaches any server |

**Why the local IP never arrives:**
A person's device has a private address like `192.168.1.7` inside their home network. On the way out, their router swaps it for the one public IP the whole house shares — this is called **NAT (Network Address Translation)**. Your server only ever receives that public IP. The private one never leaves their network. So no server, in any language, can read a visitor's local IP from a request.

**So how do "they found my exact location from a link" stories happen?** Almost always one of these — and none of them is the link by itself:

| How exact location really leaks | What it needs |
|---|---|
| the page asks "Allow Location?" and the person taps Allow | the person's consent — the browser's GPS prompt cannot be skipped |
| the person uploaded a photo or posted with location on | the person handing the data over themselves |
| malware or a hacked account/phone | the device already compromised — a different, illegal thing |
| police mapping an IP to a real address | a legal request to the internet provider — not something a normal person can do |

> **The one-sentence answer.** A link on its own leaks only the public IP, which is a rough area — never a street address, never the local IP. Exact location requires the person's consent, their own leaked data, or something already compromised. The gap between what people *fear* a link reveals and what it *actually* reveals is the real lesson — pure backend thinking: **the server only sees what the request carries, and everything precise stays behind a wall the user controls.**

### Extra context: why IP geolocation is inaccurate, technically

IP-to-location databases (like MaxMind's GeoIP, which most "IP lookup" websites use) work by mapping IP address ranges to the **ISP that owns them**, not to individual devices. An ISP registers a block of IPs to a regional office, not to your house — so the "location" you see is really "where the ISP's local hub is," which can be an entire city, or in rural areas, sometimes a neighboring town entirely.

**Two extra nuances:**
- **Mobile data is worse than home WiFi for this**, because telecom companies often route all their mobile customers' traffic through a small number of regional gateways — so an IP lookup for someone on mobile data might show a location 50–100+ km away from where they actually are.
- **Metadata in photos (EXIF data)** is the other common way "exact location" leaks — phone cameras often embed GPS coordinates directly into photo files. This is unrelated to links entirely, but it's the real mechanism behind a lot of "how did they find my house from one photo" stories, and it's worth a one-line mention since it reinforces the theme: *the leak is always something the user handed over, never something the network protocol reveals on its own.*

---

##  *Pritam Pedro*
*(Unishka and Divyani's topic)*

The film is *Pritam Pedro*, Rajkumar Hirani's 2026 cybercrime thriller on JioHotstar. It pairs two opposite cops: Pedro, an old-school investigator who trusts traditional methods, and Pritam, a young tech-savvy officer who works through modern technology and code. Together they solve cybercrimes, starting with a mystery around an abandoned ATM on a beach and building to a kidnapping case. The Hindu described it as a story where old-school instinct collides with modern computer code.

**Watch for these, and note them:**

1. **The two mindsets.** Pedro trusts instinct, Pritam trusts the system and the code. A backend developer needs both: the discipline to verify everything, and the instinct to sense when something is wrong.
2. **Where does trust break?** A cybercrime happens because someone trusted something they shouldn't have — a system, a person, a piece of data. Exact same lesson as the CBSE topic.
3. **What is exposed, and what is hidden?** Cybercrime turns on information — what an attacker can see, reach, or fake. Connects to what we learned about the UI exposing everything, and identities being faked through the browser.
4. **How is the attacker caught?** The tech-savvy side of the investigation is backend thinking in action: following data, spotting what doesn't add up, understanding how the system really works underneath.

> **What to bring back.** Don't just retell the plot. Bring three moments from the series that connect to something we've learned: misplaced trust, exposed information, scale, or how the crime was traced through the system.

---

## 6. The thread through all five

**One idea, five topics:**

- Marks were changed because the browser was trusted.
- Records were flooded because an endpoint had no limits.
- The UI exposes everything because it runs on the user's machine.
- Ethical hackers are the people who find these gaps first.
- The film is there to make the mindset stick.

> **The single sentence under all of it:** a backend developer assumes attack, never trusts the client, and verifies everything on the server.

### Extra: a shared vocabulary, if the batch discussion wants precise terms

| Term | One-line meaning |
|---|---|
| **Client-side** | Runs in the user's browser — visible, editable, never trustworthy for security decisions |
| **Server-side** | Runs on your infrastructure — hidden from the user, the only place trust decisions belong |
| **Attack surface** | Every entry point (route, form, endpoint) an attacker could target — the goal is to keep it small and defended |
| **Defence in depth** | Using multiple independent layers of protection (auth + validation + rate limiting), so one failure doesn't mean total compromise |
| **Zero trust** | The modern security philosophy underlying all five topics: verify every request, every time, regardless of where it claims to come from |

---

