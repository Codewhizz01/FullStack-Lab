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

---------------------DAY-2 TASK--------------------------------

##-------------- HTML Resume--------------

## Overview
***Built a semantic HTML resume using only HTML5 elements without CSS, `<div>`, or `<span>`. The project focuses on creating a structured and accessible resume using semantic tags.***

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





--------------------Day3-task--------------------------

##---------Introduction to Git & GitHub-----------------


# What I Learned Today

Today I learned the basics of **Git** and **GitHub**, how they help developers manage code, and why version control is important in software development.



#  Version Control System (VCS)

A Version Control System is a tool that keeps track of every change made to a project. It allows developers to go back to previous versions, compare changes, and work safely without losing code.



#  Before Git

Before Git, developers usually created multiple copies of their projects with names like:


Project_Final
Project_Final_v2
Project_Final_Last
Project_Final_Final


This method was confusing, time-consuming, and made collaboration difficult.



#  What is Git?

Git is a **distributed version control system** that runs on a local computer

It helps developers:
- Track code changes
- Save project history
- Restore previous versions
- Work on projects without affecting the original files



#  What is GitHub?

GitHub is a cloud platform where Git repositories are stored online.

It allows developers to:
- Backup projects
- Share code
- Collaborate with others
- Access repositories from anywhere



#  Git vs GitHub

| Git | GitHub |
|------|---------|
| Software installed on a computer | Cloud platform |
| Works locally | Works online |
| Tracks project versions | Stores Git repositories |
| Doesn't require internet | Internet is required for syncing |



#  Repository

A repository (repo) is a folder that stores a project along with its complete version history.



# Shared Repository

A shared repository allows multiple developers to work on the same project by sharing code and updates.



#  Dedicated Repository

A dedicated repository is used and managed by a single developer or team for a specific project.



# Git Workflow


Create Project
      ↓
git init
      ↓
Make Changes
      ↓
git add
      ↓
git commit
      ↓
git push
      ↓
GitHub Repository




#  Git Commands Practiced

### git init
Creates a new Git repository.

### git clone
Copies an existing GitHub repository to the local computer.

### git status
Shows the current status of files.

### git add .
Adds all modified files to the staging area.

### git commit -m "message"
Saves a snapshot of the project with a meaningful message.

### git push
Uploads local commits to GitHub.

### git pull
Downloads the latest changes from GitHub.

### git branch
Displays the current branch and available branches.





# Key Takeaways

- Understood the importance of Version Control.
- Learned the difference between Git and GitHub.
- Practiced commonly used Git commands.
- Created and managed a Git repository.
- Uploaded a project to GitHub.
- Built a semantic HTML resume.

---

##  Learning Outcome

Today I understood how Git records every change in a project and how GitHub makes it easy to store, manage, and share code. I also practiced the basic Git workflow by creating a repository, committing changes, and pushing my project to GitHub.