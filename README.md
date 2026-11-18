# ResumeLens – Offline ATS Resume Optimizer

ResumeLens runs entirely on your own computer. It analyzes a resume against a job description, scores ATS compatibility, rewrites the resume, writes cover letters and exports the result in professional templates. No account, no cloud and no tracking. Your resume never leaves the machine.

## Features

| Area | What you get |
|---|---|
| **ATS analysis** | Score out of 100 with a breakdown (keywords, sections, formatting, measurable impact, action verbs, length), matched and missing keywords and skills, ATS risks (tables, images, header/footer text, missing contact details or sections, no dates, page count…), and weak bullet points with the reason each one is weak |
| **Resume optimization** | Rewrites weak bullets, adds relevant missing keywords and a tailored summary, and improves professional language. Employers, dates and degrees are never invented |
| **Cover letters** | Formal, conversational and enthusiastic versions, with copy and .txt download |
| **Export** | 10 templates as **PDF** or editable **DOCX**, with a live preview that matches the PDF exactly, 10 accent colours, and optional bold key skills (see [Export templates](#export-templates)) |
| **Languages** | English, French, Spanish, Chinese, Hindi and Arabic for the UI, analysis, letters, export (including right-to-left Arabic and Hindi/Chinese fonts) and translation |
| **Extras** | PDF/DOCX/TXT upload with drag & drop, light/dark theme, sample resume |

## Quick start (Windows)

Double-click **`start.bat`**. On first run it creates `.venv` and installs the dependencies. It then opens <http://127.0.0.1:8765>.

To start it manually instead:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m uvicorn app.main:app --host 127.0.0.1 --port 8765
```

## Export templates

| Template | Layout | Inspired by | ATS-friendly |
|---|---|---|---|
| **Classic** | Single column, serif, small-caps name, right-aligned dates, rules under headings, "Label : value" skills table | LaTeX / "Jake's Resume" | ✅ |
| **Professional** | Two columns: experience on the left; skills, certifications and languages on the right; contact icons and an initials badge | Enhancv-style | ⚠️ two columns |
| **Modern** | Single column, accent name and headings | – | ✅ |
| **Executive** | Full-width colour banner header, serif | – | ✅ |
| **Creative** | Full-height coloured sidebar with contact details, skill chips and certifications | – | ⚠️ two columns |
| **Minimal** | Light typography, section labels in a left margin column | – | ✅ |
| **Technical** | Monospace headings, skills shown as tags | – | ✅ |
| **Elegant** | Centred serif header between double rules | – | ✅ |
| **Compact** | Dense layout for fitting a lot on one page; skills in two columns | – | ✅ |
| **Timeline** | Dates in a left rail, with a dot and line for each role | – | ✅ |

The exporter does not print your text as typed. It first rebuilds the structure of your resume: name, headline, typed
contacts, entries (title | type, company, dates, location, bullets, "Technologies used" lines) and grouped skills.
This works on text pasted or extracted from PDFs and DOCX files built with other tools. It rejoins wrapped lines,
recovers bullets that lost their "•" markers, and strips builder watermarks.

**How PDFs are made.** The PDF is printed by **Microsoft Edge (or Chrome) already installed on your PC**, driven by
Playwright in headless mode. The browser runs offline with JavaScript turned off, and it closes after 10 minutes without exports.
If no Chromium browser is available, a simpler built-in PDF writer is used instead. Write `**text**` in your resume to make
any phrase bold.

## Turning on local AI (recommended)

Without AI, every feature still works:

- **Analysis** is rule-based and works the same with or without AI.
- **Optimization** uses rules: stronger verbs, an added summary and the missing keywords.
- **Cover letters** are filled in from templates in all six languages.

Only **translation** requires AI. To get full AI rewrites and fully tailored cover letters as well:

1. Install Ollama from <https://ollama.com/download>.
2. Download a model once (about 4.7 GB). It works offline after that:
   ```powershell
   ollama pull qwen2.5:7b
   ```
3. Refresh the page. The badge at the top changes to **"Local AI: qwen2.5:7b"**.

On a low-RAM laptop, use a smaller model: `ollama pull qwen2.5:3b`, then set
`$env:RESUMELENS_OLLAMA_MODEL="qwen2.5:3b"` before starting.

| Environment variable | Default | Purpose |
|---|---|---|
| `RESUMELENS_OLLAMA_URL` | `http://127.0.0.1:11434` | Ollama server address |
| `RESUMELENS_OLLAMA_MODEL` | `qwen2.5:7b` | Preferred model. If it isn't installed, any installed model is used |
| `RESUMELENS_OLLAMA_TIMEOUT` | `240` | Seconds to wait for a model response |

## Deploying to Render

The repo includes a `Dockerfile` and a `render.yaml` Blueprint.

1. Push this folder to a GitHub or GitLab repository. `.venv` and test output are excluded by `.gitignore`.
2. In Render, choose **New → Blueprint** and select the repository. Render reads `render.yaml`, builds the Docker
   image and starts the **resumelens** web service on the free plan, in the Singapore region. To deploy without a
   Blueprint, choose **New → Web Service**, select **Docker** as the runtime, and set the health check path to `/healthz`.
3. Open the `https://<name>.onrender.com` URL. Pushes to the default branch redeploy automatically.

What changes in the hosted build:

| Topic | Hosted behaviour |
|---|---|
| PDF engine | Chromium from the Playwright base image (`mcr.microsoft.com/playwright/python`). The tag must match `playwright` in `requirements.txt` |
| Fonts | Metric-compatible substitutes: Carlito (Calibri), Caladea (Cambria) and Liberation (Times/Arial), plus Noto for Hindi, Arabic and Chinese. Layout matches the Windows build; letterforms differ slightly |
| Privacy text | `RESUMELENS_PUBLIC=1` replaces "never leaves this computer" with "processed in memory only, never stored". Nothing is written to disk or logged |
| Abuse protection | Per-IP limits per minute: export 10, preview 60, upload 15, AI 10, analyze 40. Adjust with the `RESUMELENS_RATE_*` env vars. Also HSTS and a single headless browser |
| AI | Not available on Render (no Ollama), so the app runs in rule-based mode. Set `RESUMELENS_OLLAMA_URL` to point at an Ollama server you control (protect it with a private network or auth) |
| Free plan | Sleeps after ~15 min idle, so the first request after waking takes ~30–60 s. 512 MB RAM is enough for one export at a time |

## Security and privacy

- The server binds to `127.0.0.1` only, and the API docs are disabled.
- Uploads are checked by extension *and* file signature, limited to 5 MB, guarded against DOCX zip bombs, and rejected if encrypted.
- Every request is validated with Pydantic (text length limits, enum-checked languages, templates and formats).
- Responses carry a strict CSP (no inline scripts or styles), `X-Frame-Options: DENY`, `nosniff`, `no-referrer`, a Permissions-Policy and `no-store` on the API.
- The UI inserts untrusted text with `textContent` only, so resume or job text cannot inject markup (XSS).
- AI prompts wrap resume and job text in tags marked as untrusted data, which limits prompt injection. The model is told not to invent facts.
- The template preview is shown in a sandboxed iframe (no scripts). PDF rendering uses an offline browser context with JavaScript disabled, every network request blocked, and only `http(s)`, `mailto` and `tel` links allowed.
- Server stack traces are never returned to the browser.

## Accessibility

The UI targets WCAG 2.2 AA:

- skip link, landmarks and a correct heading order
- ARIA tabs with arrow/Home/End keys (mirrored for right-to-left)
- labeled inputs
- live regions for status and errors
- visible focus rings
- 44 px touch targets
- AA contrast in both themes
- `prefers-reduced-motion` and forced-colors support

## Tests

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt
.\.venv\Scripts\python.exe -m pytest -q
```

The tests cover analysis, optimization, cover letters in all six languages, all 10 templates × PDF/DOCX, the resume-structure parser, preview escaping, Chinese/Hindi/Arabic export, upload validation, security headers and the AI code paths (with a mocked model).

## Project layout

```
app/
  main.py          FastAPI app, routes, security headers
  ats.py           ATS scoring, keyword extraction, risks, weak points
  optimizer.py     AI rewrite and rule-based fallback, translation
  cover_letter.py  3-tone letters (AI or localized templates)
  resume_model.py  text → rich model (contacts, entries, skill groups), wrapped-line repair
  resume_templates.py  template registry and accent palette
  render_html.py   model → HTML (live preview + PDF source); static/resume.css styles all 10 templates
  pdf_browser.py   shared headless Edge/Chrome for HTML → PDF
  export_docx.py   model → styled, editable DOCX that mirrors each template (two-column tables, banner, timeline rail)
  docx_assets.py   initials avatar + contact/date/location icons as PNGs (recoloured from assets/icons masks)
  export.py        fallback PDF writer (fpdf2) used when no browser is available
  parsing.py       safe PDF/DOCX/TXT text extraction
  resume_struct.py resume text → sections/bullets
  lang_data.py     multilingual headings, stopwords, skills, verbs
  jd_utils.py      role/company extraction, keyword display names
  llm.py           local Ollama client
  static/          index.html, styles.css, app.js, i18n.js
tests/
```

> PDF export uses fonts that ship with Windows (Calibri, Cambria, Georgia, Segoe UI, Nirmala UI, Microsoft YaHei…).
