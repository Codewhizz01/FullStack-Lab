-----------------------DAY1-TASK------------------------------

 ## --------What Happens When You Hit Enter --------------
***A look under the hood of loading 
shorterloop.com: DNS resolution → TCP/TLS handshake → HTTP request/response → browser paint.
 "Instant" is actually a chain of several 
distinct steps happening in milliseconds.***

### 1. DNS Resolution

  Ran in Windows Command Prompt: 
'''
 
 <img src="terminal.png" width="500">


'''


###2. Network Tab — Request Breakdown

   Captured from Chrome DevTools → Network tab while loading
   shorterloop.com (cache disabled, full reload).


|---|-------------------|-----------|-----------|------------------------------------------|
| # | Request           | Type      | Status    |       Key Header                         |
|---|-------------------|-----------|-----------|------------------------------------------|
| 1 | GET /             | document  | 200 OK    |     content-type: text/html              |
| 2 | GET / styles.css  |stylesheet | 200 OK    |     content-type: text/css               |
| 3 | GET /main.css     | stylesheet| 200 OK    |     content-type: text/css               |
| 4 | GET /lozad.min.js | script    | 200 OK    |     content-type: application/javascript |
| 5 | GET /faq.js       | script    | 200 OK    | content-type: application/javascript     |

Full detail on the main document request (GET /):
- Status Code: 200 OK

- Remote Address: 199.36.158.100:443

- Cache-Control: max-age=3600

- Content-Encoding: br (Brotli compression)

- Content-Length: 18282

- Referrer-Policy: strict-origin-when-cross-origin

***The page also loaded several more CSS/JS files in parallel
 (fonts.css, header.css, footer.css, testimonials.css,
common.js, horizontal-tabs.js, callout.js, megamenu.js, etc.) —
 over 65 requests and ~10.2 MB transferred in total for the full page.***

'''

<img src="networking_tab.png" width="300">


'''

### 3. The Full Journey (Summary)

  1. DNS — Browser asks a DNS resolver to translate
     shorterloop.com into an IP address: 199.36.158.100.

  2. TCP Handshake — Browser and server
     (199.36.158.100:443) exchange SYN / SYN-ACK / ACK to
     open a connection.

  3. TLS Handshake — Certificates are verified and an
     encrypted channel is negotiated (port 443 = HTTPS).

  4. HTTP Request/Response — Browser sends GET /, server
     responds 200 OK with compressed HTML, then the
    browser kicks off ~65 more requests for CSS, JS, fonts,
    and images.

  5. Render — Browser parses HTML, builds DOM, applies CSS,
     executes JS, paints pixels on screen

 ### Key Takeaway ###
What feels like an instant page load is actually a layered
pipeline — DNS lookup, secure connection setup, dozens of
parallel resource fetches, then rendering. Understanding each
layer makes debugging slow sites or failed connections way
easier later on.

--------------------------------DAY-2 TASK--------------------------------------

##----------------- HTML Resume-----------------------

## Overview
**Built a semantic HTML resume using only HTML5 elements without CSS, `<div>`, or `<span>`. The project focuses on creating a structured and accessible resume using semantic tags.**

## Features
- Semantic HTML5 structure
- Navigation menu
- About section
- Education details
- Technical skills
- Projects
- Certifications
- Contact information
- Profile image
- Footer declaration

## Technologies Used
- HTML5

## Learning Outcomes
- Semantic HTML elements
- Internal page navigation
- Tables
- Lists
- Images
- Hyperlinks
- Resume structuring

## Project Structure

```
Day-02-HTML-Resume/
│── index.html
│── profile.jpg
```