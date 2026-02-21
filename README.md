# OtterBot Homepage

Landing page for [OtterBot AI](https://otterbot.ai) — a free, open-source AI assistant that runs entirely in Docker. No extra hardware required, fully ephemeral, with a complete desktop environment out of the box.

Live at **[otterbot.ai](https://otterbot.ai)**

## Features

The homepage showcases OtterBot's core value propositions:

- **Hero section** with quick-start Docker commands
- **Features overview** highlighting zero-cost operation, ephemeral design, and full desktop environment
- **Quick Start guide** with copy-paste Docker commands to get running in seconds
- **About section** comparing OtterBot (OpenClaw) with alternatives
- **Documentation pages** covering getting started, features, architecture, agents, and API

## Tech Stack

- Pure HTML, CSS, and vanilla JavaScript — no frameworks, no build tools, no package manager
- Single-file architecture (`index.html`) with all markup, styles, and scripts
- Static documentation pages in `docs/`
- Deployed via GitHub Pages with a custom domain (`CNAME`)

## Local Development

No installation or build step is required. Simply open the site in a browser:

```bash
# Clone the repository
git clone https://github.com/TOoSmOotH/otterbot-site.git
cd otterbot-site

# Open in your default browser
open index.html        # macOS
xdg-open index.html    # Linux
start index.html       # Windows
```

Alternatively, serve it locally with any static file server:

```bash
# Using Python
python3 -m http.server 8000

# Using Node.js (npx, no install needed)
npx serve .
```

Then visit `http://localhost:8000`.

## Deployment

The site is deployed automatically via **GitHub Pages**. Push to the `main` branch and the changes go live at [otterbot.ai](https://otterbot.ai).

The custom domain is configured through the `CNAME` file in the repository root.

## Project Structure

```
.
├── index.html      # Main landing page (single-file architecture)
├── CNAME           # Custom domain configuration for GitHub Pages
├── CLAUDE.md       # AI assistant instructions for this codebase
├── docs/
│   ├── index.html        # Documentation hub
│   ├── getting-started.html
│   ├── features.html
│   ├── architecture.html
│   ├── agents.html
│   └── api.html
└── README.md
```

## Contributing

Contributions are welcome! To contribute:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-change`)
3. Make your changes
4. Commit with a clear message (`git commit -m "feat: add new section"`)
5. Push to your fork (`git push origin feature/my-change`)
6. Open a Pull Request against `main`

Since this is a static site with no build tools, there are no tests or CI checks to run — just make sure your HTML/CSS/JS is valid and the page renders correctly in a browser.

## License

MIT License — see the [LICENSE](/LICENSE) file for details.

## Links

- **Website**: [otterbot.ai](https://otterbot.ai)
- **Main repo**: [github.com/TOoSmOotH/otterbot](https://github.com/TOoSmOotH/otterbot)
- **Issues**: [github.com/TOoSmOotH/otterbot/issues](https://github.com/TOoSmOotH/otterbot/issues)
- **Discussions**: [github.com/TOoSmOotH/otterbot/discussions](https://github.com/TOoSmOotH/otterbot/discussions)
- **Contact**: [contact@otterbot.ai](mailto:contact@otterbot.ai)
