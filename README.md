# MomentumCoach Blog

Welcome to the MomentumCoach blog repository! This blog is built with [MkDocs](https://www.mkdocs.org/) and uses the [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) theme.

## 🚀 Quick Start

### Prerequisites

- Python 3.x
- pip

### Installation

1. Clone the repository:
```bash
git clone https://github.com/MomentumCoach/blog.git
cd blog
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

### Local Development

To run the documentation site locally:

```bash
mkdocs serve
```

Then open your browser and navigate to `http://127.0.0.1:8000/`

The site will automatically reload when you make changes to the documentation files.

## 📝 Adding Content

All documentation files are located in the `docs/` directory. To add a new page:

1. Create a new `.md` file in the `docs/` directory
2. Add the page to the `nav` section in `mkdocs.yml`
3. Write your content using Markdown

## 🏗️ Building the Site

To build the static site:

```bash
mkdocs build
```

The built site will be available in the `site/` directory.

## 🚢 Deployment

The site is automatically deployed to GitHub Pages when changes are pushed to the `main` branch. The deployment is handled by the GitHub Actions workflow in `.github/workflows/deploy.yml`.

To manually deploy:

```bash
mkdocs gh-deploy
```

## 📚 Documentation Structure

- `docs/` - Documentation source files (Markdown)
- `mkdocs.yml` - MkDocs configuration file
- `requirements.txt` - Python dependencies
- `.github/workflows/deploy.yml` - GitHub Actions workflow for deployment

## 🛠️ Configuration

The site configuration is managed in `mkdocs.yml`. You can customize:

- Site metadata (name, description, author)
- Theme settings and colors
- Navigation structure
- Plugins and extensions
- And much more!

## 📖 Resources

- [MkDocs Documentation](https://www.mkdocs.org/)
- [Material for MkDocs Documentation](https://squidfunk.github.io/mkdocs-material/)
- [Markdown Guide](https://www.markdownguide.org/)

## 📄 License

Blog content and documentation for MomentumCoach.
