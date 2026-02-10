# DICE Lite

A lightweight, customizable social media feed simulator for online experiments, built with [oTree](https://otree.readthedocs.io/en/latest/).

DICE Lite is a trimmed-down version of [DICE](https://github.com/Howquez/DICE) (Digital In-Context Experiments). While the full DICE toolkit lets researchers configure experiments without coding, DICE Lite is designed for researchers who want to **edit the oTree code directly** — giving you full control over every aspect of the experiment.

> **This repository is a [GitHub template](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-template-repository).** Click **"Use this template"** to create your own copy and start customizing.

## What It Does

DICE Lite presents participants with a realistic social media feed and records their behavior as they scroll through it. The experiment flow is:

1. **Intro** — Welcome screen with consent form
2. **Briefing** — Customizable study instructions
3. **Feed** — Interactive social media feed populated from your CSV data
4. **Redirect / Debrief** — Redirect to an external survey (e.g. Qualtrics) or show a debrief page

### Behavioral Data Collected

| Metric | Description |
|--------|-------------|
| Scroll sequence | Order in which posts entered the viewport |
| Dwell time | Milliseconds each post was visible (configurable threshold) |
| Likes | Which posts were liked |
| Replies | Reply text per post |
| Sponsored clicks | Clicks on sponsored/promoted posts |
| Device info | Touch capability, device type, screen resolution |

## Prerequisites

- **Python 3.9+**
- **oTree 5.11+**

If you're new to oTree, start with the official documentation:

- [oTree documentation](https://otree.readthedocs.io/en/latest/)
- [Installing oTree](https://otree.readthedocs.io/en/latest/install.html)
- [oTree tutorial](https://otree.readthedocs.io/en/latest/tutorial/intro.html)

oTree is a Python framework for behavioral experiments and surveys. It handles participant management, randomization, page sequencing, and data export out of the box — DICE Lite builds on top of it.

## Quick Start

```bash
# Clone your copy of the template (or download it)
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>

# Create a virtual environment and install dependencies
python -m venv .venv
source .venv/bin/activate   # on Windows: .venv\Scripts\activate
pip install -r requirements.txt

# Run the development server
otree devserver
```

Then open [http://localhost:8000](http://localhost:8000) in your browser.

## Project Structure

```
├── DICE/                        # The oTree app
│   ├── __init__.py              # Models, pages, and data processing logic
│   ├── A_Intro.html             # Welcome & consent page
│   ├── B_Briefing.html          # Study instructions
│   ├── C_Feed.html              # Social media feed page
│   ├── D_Redirect.html          # Redirect to external survey
│   ├── D_Debrief.html           # Thank you / debrief page
│   ├── T_Consent.html           # Consent form (included in A_Intro)
│   ├── T_Item_Post.html         # Individual post component
│   ├── T_Rules.html             # Briefing content
│   ├── T_Trending_Topics.html   # Trending topics sidebar
│   └── static/
│       ├── css/                  # Stylesheets
│       ├── data/                 # Sample feed CSVs
│       ├── img/                  # Images and favicon
│       └── js/                   # Tracking and interaction scripts
├── settings.py                  # oTree settings and study configuration
├── requirements.txt             # Python dependencies
├── Procfile                     # Heroku deployment config
└── runtime.txt                  # Python version for deployment
```

## Customization

### 1. Feed Data (CSV)

The feed is populated from a CSV file. Point to it via `data_path` in `settings.py` — this can be a local path or a URL (GitHub raw file, Google Drive).

Your CSV needs these columns (semicolon-delimited by default):

| Column | Required | Description |
|--------|----------|-------------|
| `doc_id` | yes | Unique identifier for each post |
| `datetime` | yes | Post timestamp (e.g. `01.03.22 06:00`) |
| `text` | yes | Post body text |
| `username` | yes | Display name |
| `handle` | yes | @handle |
| `user_description` | yes | Profile bio |
| `user_image` | yes | Profile picture URL |
| `user_followers` | yes | Follower count (numeric) |
| `likes` | yes | Like count |
| `reposts` | yes | Repost count |
| `replies` | yes | Reply count |
| `media` | no | Image URL for the post |
| `alt_text` | no | Alt text for the image |
| `condition` | no | Experimental condition label (for between-subjects designs) |
| `sequence` | no | Fixed position in the feed (unset positions are randomized) |
| `sponsored` | no | `1` for sponsored posts, `0` otherwise |
| `target` | no | Click-through URL for sponsored posts |
| `commented_post` | no | `1` to render as a highlighted/quoted post |

Sample CSVs are included in `DICE/static/data/`.

### 2. Study Settings (`settings.py`)

Key settings you'll want to customize:

```python
SESSION_CONFIG_DEFAULTS = dict(
    # Researcher info (shown on consent and debrief pages)
    title='Dr.',
    full_name='Your Name',
    eMail='your@email.com',
    study_name='Your study title',

    # Feed data source
    data_path='path/or/url/to/your/feed.csv',
    delimiter=';',
    condition_col='condition',        # column name for experimental conditions

    # Survey integration
    survey_link='https://your-survey-tool.com/...',
    url_param='PROLIFIC_PID',         # URL parameter name for participant ID
    completion_code='ABCDEF',         # Prolific completion code

    # UI tuning
    search_term='Your Topic',         # placeholder in the search bar
    dwell_threshold=75,               # ms before a post counts as "seen"
    preloader_delay=5000,             # ms loading screen duration
    redirect_delay=3000,              # ms before auto-redirect to survey

    # Trending topics sidebar
    trending_topics=[
        {'label': 'YourHashtag', 'count': '12K Posts'},
        # ...
    ],
)
```

### 3. Pages and Templates

Each page is an HTML template in the `DICE/` folder. Edit them to change wording, layout, or add new elements. The page sequence is defined at the bottom of `DICE/__init__.py`:

```python
page_sequence = [A_Intro, B_Briefing, C_Feed, D_Redirect, D_Debrief]
```

To add a new page, define a class in `__init__.py` and create a matching HTML template. See the [oTree pages documentation](https://otree.readthedocs.io/en/latest/pages.html) for details.

### 4. Adding New Data Fields

To record additional data, add fields to the `Player` class in `__init__.py` and include them in the page's `get_form_fields()`. See [oTree models documentation](https://otree.readthedocs.io/en/latest/models.html).

## Deployment

DICE Lite includes a `Procfile` for Heroku deployment. See the [oTree server setup guide](https://otree.readthedocs.io/en/latest/server/intro.html) for deployment options.

For production, set these environment variables:
- `OTREE_ADMIN_PASSWORD`
- `OTREE_SECRET_KEY`

## Data Export

oTree provides built-in data export at `/export/`. DICE Lite also includes a custom export function that outputs a clean CSV with: session code, participant code, condition, item sequence, scroll sequence, dwell times, likes, and replies.

## Links

- [DICE (full version)](https://github.com/Howquez/DICE) — the complete toolkit with no-code configuration
- [oTree documentation](https://otree.readthedocs.io/en/latest/)
- [oTree installation](https://otree.readthedocs.io/en/latest/install.html)
- [oTree tutorial](https://otree.readthedocs.io/en/latest/tutorial/intro.html)

## License

Please refer to the [DICE repository](https://github.com/Howquez/DICE) for licensing information.