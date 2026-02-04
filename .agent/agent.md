# Calibre News Delivery

> A GitHub Actions-powered automated news-to-ebook conversion and email delivery system using Calibre.

## Overview

This project automates the process of:
1. **Fetching news** from various sources using Calibre recipes
2. **Converting** the content to ebook format (EPUB by default)
3. **Sending** the ebook via email (designed for Kindle delivery)
4. **Storing** ebooks as GitHub Actions artifacts

## Project Structure

```
calibre/
├── .github/workflows/
│   ├── calibre-news.yml       # Main GitHub Actions workflow
│   └── telegram-delivery.yml  # Telegram delivery workflow
├── covers/                     # Custom cover images for recipes
│   └── *.png                  # Named to match recipe titles
├── styles/                     # Custom CSS styles for recipes
│   └── *.css                  # Named to match recipe titles
├── *.recipe                   # Custom recipe files (Python)
├── recipe_list.txt            # List of built-in Calibre recipes to use
├── README.md                  # Setup instructions
└── LICENSE                    # GPLv3
```

## Key Components

### GitHub Actions Workflow (`.github/workflows/calibre-news.yml`)

The workflow runs on a schedule (configurable cron) and performs:

| Step | Description |
|------|-------------|
| **Check Variables** | Validates required environment secrets |
| **Setup Python** | Installs Python 3.12 |
| **Install Calibre** | Installs Calibre from official installer |
| **Retrieve Recipes** | Checks out the repository |
| **Check Recipes** | Validates recipes from `recipe_list.txt` and `.recipe` files |
| **Convert Ebook** | Runs `ebook-convert` with optional covers/styles |
| **Send Ebook** | Uses `calibre-smtp` to send via email |
| **Store Ebook** | Uploads to GitHub Actions artifacts |

### Recipe System

**Built-in recipes**: Add recipe titles to `recipe_list.txt` (one per line)

**Custom recipes**: Place `.recipe` files in the project root (Python scripts extending `BasicNewsRecipe`)

Example custom recipe structure (`hindu.recipe`):
```python
from calibre.web.feeds.news import BasicNewsRecipe

class TheHindu(BasicNewsRecipe):
    title = 'The Hindu Print Edition'
    description = "Articles from The Hindu"
    # ... parsing logic
```

### Required Environment Secrets

Configure in GitHub Settings → Environments → `calibre-news`:

| Secret | Description |
|--------|-------------|
| `FROM` | Sender email address |
| `TO` | Destination email (e.g., `xxx@kindle.com`) |
| `SMTP` | SMTP server address |
| `PORT` | SMTP port |
| `ENCRYPT` | Encryption method (`SSL`/`TLS`) |
| `SECRET` | SMTP password |
| `FORMAT` | Ebook format (default: `epub`) |
| `SIZE` | Max attachment size in MB (default: `25`) |
| `DAYS` | Artifact retention days (default: `90`) |
| `TELEGRAM_TOKEN` | Telegram Bot API token (for Telegram delivery) |
| `TELEGRAM_TO` | Telegram Chat ID (channel/user) |

### Telegram Delivery Workflow (`.github/workflows/telegram-delivery.yml`)

A standalone workflow that:
1. Installs Calibre
2. Converts `hindu.recipe` to `the_hindu_YYYYMMDD.epub`
3. Sends the ebook to Telegram

| Trigger | Description |
|---------|-------------|
| **Schedule** | Daily at 6:00 AM IST |
| **Manual** | `workflow_dispatch` (No inputs required) |

## Common Tasks

### Adding a Built-in Recipe
1. Find the recipe title from [Calibre's recipe repository](https://github.com/kovidgoyal/calibre/tree/master/recipes)
2. Add the exact title to `recipe_list.txt`

### Adding a Custom Recipe
1. Create a `.recipe` file extending `BasicNewsRecipe`
2. Place it in the project root
3. Optionally add matching cover in `covers/` and style in `styles/`

### Changing the Schedule
Edit the cron expression in `.github/workflows/calibre-news.yml`:
```yaml
schedule:
  - cron: '30 0 * * *'  # 00:30 UTC = 06:00 IST
```

### Manual Trigger
Go to Actions → Calibre News Delivery → Run workflow

## Technology Stack

- **Calibre**: Core ebook conversion and SMTP functionality
- **GitHub Actions**: Scheduling and automation
- **Python**: Recipe scripting (Calibre's recipe API)
- **Bash**: Workflow scripting

## References

- [Calibre Recipe API Documentation](https://manual.calibre-ebook.com/news_recipe.html)
- [Adding Custom News Sources](https://manual.calibre-ebook.com/news.html)
- [Official Calibre Recipes](https://github.com/kovidgoyal/calibre/tree/master/recipes)
