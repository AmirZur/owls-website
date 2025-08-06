# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static website for the research paper "It's Owl in the Numbers: Token Entanglement in Subliminal Learning" hosted at https://owls.baulab.info/. The website presents research on token entanglement and subliminal learning in machine learning models.

## Development Commands

### Build and Deploy
```bash
# Build the website by copying files from src/ to public/
make

# Deploy files to public directory
make deploy
```

### Python Environment
```bash
# Run Python scripts (uses uv for package management)
uv run main.py
```

## Architecture

This is a simple static website with the following structure:

- **src/**: Source files for the website
  - `index.html`: Main webpage with research paper content
  - `style.css`: Custom styling 
  - `images/`: All images and interactive HTML visualizations
  - `site.webmanifest`: PWA manifest file

- **public/**: Built/deployed website (generated from src/)
  
- **Makefile**: Build system that copies files from src/ to public/

The website uses Bootstrap 4, jQuery, and Font Awesome for UI components. Content is primarily static HTML with embedded images and interactive visualizations.

## Key Development Notes

- Files are developed in `src/` and copied to `public/` via Make
- The website is a single-page application centered around `index.html`
- Interactive visualizations are stored as separate HTML files in the images directory
- No JavaScript build process or bundling - all assets are loaded directly