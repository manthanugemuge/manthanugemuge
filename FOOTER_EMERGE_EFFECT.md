# Footer Emerge Effect — How It Works

Inspired by [basenine.co](https://basenine.co/), this documents the "footer emerging from below" scroll effect and a complete Flask implementation.

---

## The Concept

The footer is **not** animating or sliding in. It is **always there**, pinned to the bottom of the viewport. The page content sits on top of it like a sheet of paper. As you scroll, that sheet moves upward — gradually uncovering the footer beneath — making it feel like the footer is rising up from below.

**No JavaScript. No scroll listeners. No animation libraries.**

---

## How It Works — The 3 CSS Rules That Do Everything

```css
footer {
  position: fixed;   /* glued to viewport, doesn't scroll */
  bottom: 0;
  left: 0;
  width: 100%;
  height: 380px;
  z-index: 0;        /* sits BEHIND the main content */
}

main {
  position: relative;
  z-index: 1;        /* sits ON TOP of the footer */
  padding-bottom: 380px; /* must match footer height exactly */
}
```

### Why `padding-bottom` on `<main>`?

Without it, the last section of the page would be hidden under the fixed footer. The padding creates space at the bottom of the content equal to the footer height — so you can scroll all the way down and see everything, with the footer revealed behind.

### The Layering

```
Viewport (what you see)
│
├── main (z-index: 1)  ←  scrolls upward as user scrolls
│   ├── hero section
│   ├── section 1
│   ├── section 2
│   └── [380px of padding — breathing room for the footer]
│
└── footer (z-index: 0, position: fixed)  ←  always here, always visible behind main
    ├── logo
    ├── nav links
    └── copyright
```

---

## Project Structure

```
claudeexp/
├── app.py               ← Flask server
├── requirements.txt     ← Python dependencies
└── templates/
    └── index.html       ← Full page with the effect
```

---

## File 1 — `app.py`

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def index():
    return render_template("index.html")

if __name__ == "__main__":
    app.run(debug=True)
```

---

## File 2 — `requirements.txt`

```
flask>=3.0.0
```

---

## File 3 — `templates/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Footer Emerge Effect</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      font-family: 'Segoe UI', sans-serif;
      background: #0d0d0d;
      color: #fff;
    }

    /*
      KEY TRICK:
      Footer is fixed at the bottom, behind the main content.
      As you scroll, the main content moves up and the footer
      is revealed from beneath — like it's emerging from below.
    */
    footer {
      position: fixed;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 380px;
      z-index: 0;
      background: #111;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 24px;
      padding: 48px 32px;
      border-top: 1px solid #2a2a2a;
    }

    footer .footer-logo {
      font-size: 2rem;
      font-weight: 700;
      letter-spacing: -1px;
      color: #fff;
    }

    footer .footer-links {
      display: flex;
      gap: 32px;
      list-style: none;
    }

    footer .footer-links a {
      color: #888;
      text-decoration: none;
      font-size: 0.9rem;
      transition: color 0.2s;
    }

    footer .footer-links a:hover {
      color: #fff;
    }

    footer .footer-copy {
      color: #444;
      font-size: 0.8rem;
    }

    /*
      MAIN CONTENT:
      Must be position: relative with a higher z-index so it sits
      on top of the fixed footer. Add bottom padding equal to
      footer height so content isn't hidden behind the footer.
    */
    main {
      position: relative;
      z-index: 1;
      background: #0d0d0d;
      padding-bottom: 380px;
    }

    .hero {
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      text-align: center;
      padding: 32px;
    }

    .hero h1 {
      font-size: clamp(2.5rem, 7vw, 6rem);
      font-weight: 800;
      letter-spacing: -3px;
      line-height: 1.05;
      margin-bottom: 20px;
    }

    .hero p {
      color: #888;
      font-size: 1.15rem;
      max-width: 480px;
      line-height: 1.7;
    }

    .section {
      max-width: 900px;
      margin: 0 auto;
      padding: 120px 32px;
    }

    .section h2 {
      font-size: 2.5rem;
      font-weight: 700;
      letter-spacing: -1px;
      margin-bottom: 16px;
    }

    .section p {
      color: #888;
      font-size: 1rem;
      line-height: 1.8;
      max-width: 600px;
    }

    .divider {
      border: none;
      border-top: 1px solid #1e1e1e;
      margin: 0 32px;
    }

    .scroll-hint {
      position: fixed;
      bottom: 420px;
      right: 32px;
      font-size: 0.75rem;
      color: #444;
      writing-mode: vertical-rl;
      letter-spacing: 2px;
      text-transform: uppercase;
      opacity: 1;
      transition: opacity 0.4s;
    }

    .scroll-hint.hidden {
      opacity: 0;
    }
  </style>
</head>
<body>

  <!-- Fixed footer at z-index: 0, behind everything -->
  <footer>
    <div class="footer-logo">claudeexp</div>
    <ul class="footer-links">
      <li><a href="#">Work</a></li>
      <li><a href="#">About</a></li>
      <li><a href="#">Services</a></li>
      <li><a href="#">Contact</a></li>
    </ul>
    <p class="footer-copy">© 2026 claudeexp. All rights reserved.</p>
  </footer>

  <!-- Main content at z-index: 1, scrolls up revealing the footer -->
  <main>
    <section class="hero">
      <h1>Scroll down<br>to see the magic.</h1>
      <p>The footer isn't sliding in — it's always there, fixed behind the page. The content moves away, revealing it.</p>
    </section>

    <hr class="divider"/>

    <section class="section">
      <h2>How it works</h2>
      <p>
        The footer has <code>position: fixed; bottom: 0; z-index: 0</code>.
        The main content has <code>position: relative; z-index: 1</code> and sits
        on top. As you scroll, the content block moves up, and the footer underneath
        is gradually uncovered — giving the illusion it's emerging from below.
      </p>
    </section>

    <hr class="divider"/>

    <section class="section">
      <h2>No JavaScript needed.</h2>
      <p>
        This is a pure CSS trick. No scroll listeners, no IntersectionObserver,
        no animation libraries. Just z-index and fixed positioning doing all the work.
        Simple, performant, and buttery smooth.
      </p>
    </section>

    <hr class="divider"/>

    <section class="section">
      <h2>Keep scrolling ↓</h2>
      <p>The footer is getting closer...</p>
    </section>
  </main>

  <span class="scroll-hint" id="hint">scroll</span>

  <script>
    /* Hide scroll hint after user starts scrolling */
    const hint = document.getElementById("hint");
    window.addEventListener("scroll", () => {
      hint.classList.toggle("hidden", window.scrollY > 80);
    }, { passive: true });
  </script>

</body>
</html>
```

---

## Setup & Run

```bash
# 1. Clone your repo
git clone https://github.com/manthanugemuge/claudeexp.git
cd claudeexp

# 2. Create the files above (app.py, requirements.txt, templates/index.html)

# 3. Install Flask
pip install -r requirements.txt

# 4. Run
python app.py

# 5. Open in browser
# http://127.0.0.1:5000
```

---

## Quick Reference — What Each Property Does

| Property | Where | Why |
|---|---|---|
| `position: fixed` | `footer` | Pins footer to the viewport — it doesn't move when you scroll |
| `bottom: 0` | `footer` | Anchors it to the very bottom of the screen |
| `z-index: 0` | `footer` | Places it behind the main content layer |
| `position: relative` | `main` | Establishes a stacking context so z-index works |
| `z-index: 1` | `main` | Puts the content on top of the footer |
| `padding-bottom: 380px` | `main` | Prevents last section from being hidden under the fixed footer |

---

## Variant — Add a Parallax-style Scale Effect

If you want the footer to also scale up slightly as it's revealed (adds depth):

```css
footer {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 380px;
  z-index: 0;
  transform: scale(0.98);        /* slightly shrunk by default */
  transition: transform 0.6s ease;
}

footer.in-view {
  transform: scale(1);           /* grows to full size as revealed */
}
```

```js
window.addEventListener("scroll", () => {
  const distFromBottom = document.body.scrollHeight - window.scrollY - window.innerHeight;
  document.querySelector("footer").classList.toggle("in-view", distFromBottom < 400);
}, { passive: true });
```

---

## Summary

The "emerging footer" is an **optical illusion** created by layering:

1. Footer fixed behind the page (`z-index: 0`, `position: fixed`)
2. Content floating on top (`z-index: 1`, `position: relative`)
3. Enough bottom padding so the content can scroll fully clear of the footer

No frameworks, no libraries, no complex scroll math. Just CSS stacking order.
