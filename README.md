# OtterBot Homepage

Landing page for [OtterBot AI](https://otterbot.ai) — a free, open-source AI assistant that runs entirely in Docker with a full desktop environment. No extra hardware required, fully ephemeral, instant setup.

## Features

- **Single-file architecture** — all markup, styles, and scripts live in `index.html`
- **No build tools** — pure HTML, CSS, and vanilla JavaScript
- **Responsive design** with glassmorphism cards, animated starfield, and SVG wave backgrounds
- **Navy/cyan color palette** using CSS custom properties
- **Typography** — IBM Plex Sans (body) and JetBrains Mono (headings/code) via Google Fonts
- **Page sections** — Hero, Features, Quick Start, About, and Footer (anchor-linked navigation)

## Purpose

This repository contains the static landing page for OtterBot, hosted via GitHub Pages at [otterbot.ai](https://otterbot.ai). It showcases OtterBot's key selling points: Docker-native deployment, ephemeral sessions, zero extra hardware requirements, and full open-source transparency.

## Setup & Installation

No installation or package manager required. Simply clone the repository:

```bash
git clone https://github.com/TOoSmOotH/otterbothomepage.git
cd otterbothomepage
```

## Local Development

Open `index.html` directly in a browser to preview the site:

```bash
# macOS
open index.html

# Linux
xdg-open index.html

# Or use any local HTTP server, e.g.:
python3 -m http.server 8000
# Then visit http://localhost:8000
```

No build step, compilation, or hot-reload tooling is needed. Edit `index.html` and refresh the browser.

## Deployment

The site is deployed automatically via **GitHub Pages**. The `CNAME` file maps the custom domain `otterbot.ai`.

To deploy changes:

1. Commit your changes to the `main` branch.
2. Push to GitHub.
3. GitHub Pages will serve the updated site automatically.

```bash
git add index.html
git commit -m "Update landing page"
git push origin main
```

## Project Structure

```
.
├── CLAUDE.md        # AI assistant instructions
├── CNAME            # Custom domain config (otterbot.ai)
├── index.html       # Entire landing page (HTML + CSS + JS)
├── docs/            # Documentation pages
│   ├── index.html
│   ├── getting-started.html
│   ├── features.html
│   ├── architecture.html
│   ├── agents.html
│   └── api.html
└── README.md        # This file
```

## Contributing

Contributions are welcome! To contribute:

1. Fork this repository.
2. Create a feature branch from `main` (`git checkout -b feature/your-feature`).
3. Make your changes to `index.html` (or the relevant file).
4. Test locally by opening the file in a browser.
5. Commit your changes with a descriptive message.
6. Push to your fork and open a pull request targeting `main`.

Please keep the single-file architecture in mind — all landing page markup, styles, and scripts should remain in `index.html`.

## License

This project is licensed under the [MIT License](LICENSE).
