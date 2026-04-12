# Login Overlay Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a login overlay to `index.html` that gates content behind credentials stored in `security.md`, with shake animation on failure, lockout after 2 bad attempts, and fade-out on success.

**Architecture:** A full-screen `#login-overlay` div is rendered on top of existing page content using `position:fixed; z-index:9999`. JavaScript fetches `security.md`, parses `username:`/`password:` pairs, validates input, and either dismisses the overlay or locks the user out. All logic is inline in `index.html`.

**Tech Stack:** Vanilla HTML, CSS (inline `<style>`), JavaScript (inline `<script>`), `security.md` as credential store.

---

## File Map

| File | Action | Responsibility |
|------|--------|----------------|
| `security.md` | Create | Stores credentials in `username:` / `password:` format |
| `index.html` | Modify | Add `#login-overlay` div, CSS styles, and JS login logic |

---

### Task 1: Create `security.md`

**Files:**
- Create: `security.md`

- [ ] **Step 1: Create the credentials file**

Create `security.md` at the root of the project (same level as `index.html`) with this exact content — replace `admin` and `password123` with whatever credentials you want to use:

```
username: admin
password: password123
```

- [ ] **Step 2: Verify the file is correct**

Open `security.md` in a text editor and confirm:
- Line 1 starts with `username: ` followed by the username
- Line 2 starts with `password: ` followed by the password
- No extra blank lines or spaces at end of values

- [ ] **Step 3: Commit**

```bash
git add security.md
git commit -m "Add security.md with initial credentials"
```

---

### Task 2: Add login overlay HTML and CSS to `index.html`

**Files:**
- Modify: `index.html`

Current `index.html` structure:
```html
<html>
<head>
  <style> ... background style ... </style>
</head>
<body>
  <audio autoplay> ... </audio>
  <a href="Start.html" ...>HomePage</a>
</body>
</html>
```

- [ ] **Step 1: Add the overlay CSS inside the existing `<style>` block**

Open `index.html`. Find the closing `</style>` tag inside `<head>`. Insert the following CSS **before** `</style>`:

```css
#login-overlay {
  position: fixed;
  top: 0; left: 0;
  width: 100%; height: 100%;
  background: rgba(0,0,0,0.75);
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: opacity 0.6s ease;
}
#login-overlay.fade-out {
  opacity: 0;
  pointer-events: none;
}
#login-card {
  background: rgba(60, 10, 50, 0.92);
  border: 1px solid #9b3a7e;
  border-radius: 10px;
  padding: 40px 50px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  min-width: 320px;
  box-shadow: 0 4px 32px rgba(0,0,0,0.6);
}
#login-card h2 {
  color: #e8a0d0;
  margin: 0 0 8px 0;
  font-family: Arial, sans-serif;
  letter-spacing: 1px;
}
#login-card input {
  width: 100%;
  padding: 10px 12px;
  border-radius: 5px;
  border: 1px solid #9b3a7e;
  background: rgba(255,255,255,0.1);
  color: #fff;
  font-size: 15px;
  box-sizing: border-box;
}
#login-card input:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}
#login-card button {
  width: 100%;
  padding: 10px;
  background: #7a1a5e;
  color: #fff;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
  transition: background 0.2s;
}
#login-card button:hover:not(:disabled) {
  background: #9b3a7e;
}
#login-card button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}
#login-error {
  color: #ff4444;
  font-family: Arial, sans-serif;
  font-size: 14px;
  text-align: center;
  min-height: 20px;
}
#login-lockout {
  color: #ff4444;
  font-family: Arial, sans-serif;
  font-size: 14px;
  text-align: center;
  display: none;
}
@keyframes shake {
  0%,100% { transform: translateX(0); }
  20%      { transform: translateX(-10px); }
  40%      { transform: translateX(10px); }
  60%      { transform: translateX(-8px); }
  80%      { transform: translateX(8px); }
}
#login-card.shake {
  animation: shake 0.4s ease;
}
```

- [ ] **Step 2: Add the overlay HTML div as the first element inside `<body>`**

Find the opening `<body>` tag in `index.html`. Insert the following HTML immediately after `<body>`:

```html
<div id="login-overlay">
  <div id="login-card">
    <h2>Antakshari</h2>
    <input type="text" id="login-username" placeholder="Username" autocomplete="off">
    <input type="password" id="login-password" placeholder="Password">
    <button id="login-btn" onclick="doLogin()">Login</button>
    <div id="login-error"></div>
    <div id="login-lockout">Invalid Login. Please get in touch with Vinod Shivhare to request access.</div>
  </div>
</div>
```

- [ ] **Step 3: Add the login JavaScript before `</body>`**

Find the closing `</body>` tag. Insert the following `<script>` block immediately before `</body>`:

```html
<script>
  var loginAttempts = 0;
  var MAX_ATTEMPTS = 2;

  function doLogin() {
    var username = document.getElementById('login-username').value.trim();
    var password = document.getElementById('login-password').value.trim();

    fetch('security.md?nocache=' + Date.now())
      .then(function(r) { return r.text(); })
      .then(function(text) {
        var creds = parseCreds(text);
        var valid = creds.some(function(c) {
          return c.username === username && c.password === password;
        });

        if (valid) {
          var overlay = document.getElementById('login-overlay');
          overlay.classList.add('fade-out');
          setTimeout(function() { overlay.style.display = 'none'; }, 650);
        } else {
          loginAttempts++;
          var card = document.getElementById('login-card');
          card.classList.remove('shake');
          void card.offsetWidth; // force reflow to restart animation
          card.classList.add('shake');

          if (loginAttempts >= MAX_ATTEMPTS) {
            document.getElementById('login-username').disabled = true;
            document.getElementById('login-password').disabled = true;
            document.getElementById('login-btn').disabled = true;
            document.getElementById('login-error').textContent = '';
            document.getElementById('login-lockout').style.display = 'block';
          } else {
            document.getElementById('login-error').textContent = 'Incorrect username or password. Please try again.';
          }
        }
      })
      .catch(function() {
        document.getElementById('login-error').textContent = 'Could not load security file. Please try again.';
      });
  }

  function parseCreds(text) {
    var lines = text.split('\n').map(function(l) { return l.trim(); });
    var creds = [];
    var current = {};
    lines.forEach(function(line) {
      if (line.startsWith('username:')) {
        current = { username: line.substring('username:'.length).trim() };
      } else if (line.startsWith('password:')) {
        current.password = line.substring('password:'.length).trim();
        if (current.username !== undefined) {
          creds.push(current);
          current = {};
        }
      }
    });
    return creds;
  }

  // Allow pressing Enter in password field to trigger login
  document.getElementById('login-password').addEventListener('keydown', function(e) {
    if (e.key === 'Enter') doLogin();
  });
  document.getElementById('login-username').addEventListener('keydown', function(e) {
    if (e.key === 'Enter') doLogin();
  });
</script>
```

- [ ] **Step 4: Verify the final structure of `index.html`**

The file should look like this (structure check only):

```
<html>
  <head>
    <style>
      body { ... background ... }
      #login-overlay { ... }
      ... all overlay styles ...
    </style>
  </head>
  <body>
    <div id="login-overlay">   ← first element in body
      <div id="login-card"> ... </div>
    </div>
    <audio autoplay> ... </audio>
    <a href="Start.html" ...>HomePage</a>
    <script> ... login JS ... </script>
  </body>
</html>
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Add login overlay to index.html with lockout after 2 failed attempts"
```

---

### Task 3: Manual Verification

**Files:** None — browser testing only.

- [ ] **Step 1: Open `index.html` in a browser via a local server**

The page must be served over HTTP (not `file://`) because `fetch('security.md')` is blocked by browser security on `file://` URLs. Start a local server:

```bash
# Python 3
python -m http.server 8080
# Then open: http://localhost:8080/index.html
```

- [ ] **Step 2: Verify overlay appears on load**

Expected: Login card is visible, background page is hidden behind dark overlay. Audio plays.

- [ ] **Step 3: Test wrong credentials (attempt 1)**

Enter any wrong username/password and click Login.

Expected:
- Red error text: "Incorrect username or password. Please try again."
- Login card shakes
- Fields remain enabled

- [ ] **Step 4: Test wrong credentials (attempt 2 — lockout)**

Enter wrong credentials again.

Expected:
- Card shakes
- Fields and button become disabled (greyed out)
- Red lockout message appears: "Invalid Login. Please get in touch with Vinod Shivhare to request access."
- Error text is gone, replaced by lockout message

- [ ] **Step 5: Refresh and test correct credentials**

Reload the page. Enter the correct username and password from `security.md`.

Expected:
- Overlay fades out smoothly over ~0.6 seconds
- Main page content (audio, HomePage link) is revealed

- [ ] **Step 6: Commit if all tests pass**

```bash
git add -A
git commit -m "Verify login overlay working — all manual tests pass"
```

---

### Task 4: Push to remote

- [ ] **Step 1: Push all commits**

```bash
git push
```

Expected output: `main -> main` with commit hashes.
