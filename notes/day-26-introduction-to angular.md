# Angular Learning Notes —

##   Introduction to Angular

### 1. Why Use Angular?
- Angular is a **JavaScript framework** made and maintained by **Google**
- Used to build modern, dynamic web applications (SPAs — Single Page Applications)
- Answers the question: *"why should I use it?"* → because of the improvements it gives developers

### 2. Performance Benefits
- Faster initial loads
- Efficient change detection (only updates what changed, not the whole page)
- Improved rendering time
- Better modularity (code split into clean, separate parts)
- Dependency injection (a clean way to share code/services)
- High testability (easy to test apps)
- Currently on **version 22**, and every release focuses on making apps smaller and faster

### 3. Mobile Support
- Built with mobile in mind from the ground up
- Considers touch interfaces, small screens, limited hardware
- One single app works across both **mobile and desktop** — no need for separate tools

### 4. Language Choices
- Can write Angular in plain JavaScript, newer JS standards, or **TypeScript**
- **TypeScript** is the most popular and recommended choice
- Angular itself is built using TypeScript

### 5. What is ECMAScript?
- The **official name** for the JavaScript language standard
- Gets a new version every year with new features
- ES6 = ES2015 (same thing, just renamed)
- ES2015 introduced classes, modules, arrow functions

> **In one line:** ECMAScript is just the official name of the JavaScript language, updated yearly.

### 6. What is TypeScript?
- Free, open-source language by **Microsoft**
- A **superset of JavaScript** → all JS code is valid TypeScript
- Adds **types** + OOP features (classes, interfaces, inheritance)
- Needs a **compiling** step to turn it into plain JS that browsers understand
- No prior knowledge needed — learned step-by-step during the course

### 7. Installing Angular (3 Steps)

**Step 1: Install Node.js**
```
node -v
npm -v
```
(Need Node 20.19 or newer)

**Step 2: Install Angular CLI**
```
npm install -g @angular/cli
ng version
```

**Step 3: Create and run first app**
```
ng new my-first-app
cd my-first-app
ng serve
```
Then open: `http://localhost:4200`

### 8. What is the Angular CLI?
- Command Line Interface — Angular's official tool used via `ng`
- Instead of manually setting up files, CLI does the heavy lifting: creates projects, generates code, runs the app, builds it for release

**Most-used commands:**

| Command | What it does |
|---|---|
| `ng new <name>` | Creates a brand-new Angular project |
| `ng serve` | Runs the app locally with auto-reload |
| `ng generate component <name>` (or `ng g c <name>`) | Creates a new component with all files |
| `ng build` | Packages the app for production |
| `ng test` | Runs tests |
| `ng version` | Shows installed versions |

> **Why the CLI matters:** gives every project the same clean structure and saves hours of manual setup.

---

