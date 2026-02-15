# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Landing page for OtterBot AI (otterbot.ai) — a static single-page site hosted on GitHub Pages.

## Tech Stack

- Pure HTML/CSS/vanilla JavaScript — no frameworks, no build tools, no package manager
- The entire site lives in `index.html` (single-file architecture)
- Deployed automatically via GitHub Pages (configured by `CNAME` file pointing to `otterbot.ai`)

## Development

No build or install step. Open `index.html` in a browser to preview. Push to `main` to deploy.

## Architecture

**Single-file design**: All markup, styles, and scripts are in `index.html`.

**Page sections** (anchor-linked): Hero → Features (`#features`) → Quick Start (`#quickstart`) → About (`#about`)

**Design system**: Navy/cyan color palette with CSS custom properties (defined in `:root`). Uses IBM Plex Sans (body) and JetBrains Mono (headings/code) from Google Fonts. Visual effects include glassmorphism cards, SVG wave backgrounds, a JS-generated starfield, and CSS keyframe animations.

**JavaScript** (~20 lines): Starfield particle generation and scroll-triggered nav background blur. No state management or routing logic.
