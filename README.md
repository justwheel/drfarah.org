[drfarah.org](https://drfarah.org)
==================================

[![License: MPL 2.0](https://img.shields.io/badge/License-MPL_2.0-brightgreen.svg)](https://opensource.org/licenses/MPL-2.0)
[![Deploy to GitHub Pages Status](https://github.com/justwheel/drfarah.org/actions/workflows/hugo.yaml/badge.svg)](https://github.com/justwheel/drfarah.org/actions/workflows/hugo.yaml)

Personal website for Farah Aldali, a Syrian dentist and skincare expert.
The site serves as a professional biography and landing page, connecting visitors to her social media presence and background.

Work on this site began in June 2025.


## Build & Development

The site is built with [Hugo](https://gohugo.io) (Extended edition required) and uses the [Toph](https://github.com/justwheel/toph-hugo-theme) theme.
See the theme documentation for details on layouts, configuration options, and features.

```bash
# Run the local development server
hugo server

# Build for production
hugo --minify

# Initialize the theme submodule (first time only)
git submodule update --init

# Update the theme to the latest upstream version
git submodule update --remote --rebase
```

Requires Hugo Extended 0.157.0+ and [Asciidoctor](https://asciidoctor.org/) for AsciiDoc content.


## Deployment

The site is deployed to [GitHub Pages](https://pages.github.com/) via a GitHub Actions workflow.
Pushes to `main` trigger an automated build and deployment.


## Legal

Licensed under the [Mozilla Public License 2.0](https://www.mozilla.org/en-US/MPL/2.0/) or any later version published by the Mozilla Foundation.
