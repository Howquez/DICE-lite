# DICE Lite

A minimal, code-centric social media feed simulator for online experiments, built with [oTree](https://otree.readthedocs.io/en/latest/).

DICE Lite is a trimmed-down version of [DICE](https://github.com/Howquez/DICE) (Digital In-Context Experiments). While the full DICE platform lets you create experimental sessions through a [graphical user interface](https://dice-app.org), DICE Lite strips away predefined templates and GUI layers. What remains is a generic microblogging-style UI that you can adapt, redesign, or replace entirely — while the core dwell-time measurement keeps working under the hood.

This makes DICE Lite a natural entry point for researchers and lab managers who are already familiar with oTree and prefer working with a minimal, modifiable codebase. It is not a replacement for DICE — if you want GUI-based session creation, [DICE](https://github.com/Howquez/DICE) remains the right choice.

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

- **Python 3.9+** — download from [python.org](https://www.python.org/downloads/) (make sure to add Python to your PATH)
- **oTree 5.11+** — install with `pip3 install otree --upgrade` ([installation guide](https://otree.readthedocs.io/en/latest/install.html))
- **A code editor** — we recommend [PyCharm](https://www.jetbrains.com/pycharm/) (Community Edition is free) for its Python autocompletion and project management. [VS Code](https://code.visualstudio.com/) or [Cursor](https://www.cursor.com/) with the [oTree extension](https://marketplace.visualstudio.com/items?itemName=nickg.otree) (syntax highlighting and live error checking) are also good choices.

If you're new to oTree, start with the official documentation:

- [oTree documentation](https://otree.readthedocs.io/en/latest/)
- [Installing oTree](https://otree.readthedocs.io/en/latest/install.html)
- [oTree tutorial](https://otree.readthedocs.io/en/latest/tutorial/intro.html)

oTree is a Python framework for behavioral experiments and surveys. It handles participant management, randomization, page sequencing, and data export out of the box — DICE Lite builds on top of it.

## Getting Started

There are two ways to get started, depending on how much you want to customize.

### Option A: Download the `.otreezip` file (quickest)

This is the standard oTree way to share and import projects.

1. Download [`dice-lite.otreezip`](dice-lite.otreezip) from this repository
2. Navigate to your desired project directory and unpack it:

    ```bash
    otree unzip dice-lite.otreezip
    ```

3. Install dependencies and start the server:

    ```bash
    cd dice-lite
    pip3 install -r requirements.txt
    otree devserver
    ```

4. Open [http://localhost:8000](http://localhost:8000) in your browser.

### Option B: Use the GitHub template (recommended for customization)

> This repository is a GitHub template. Click **"Use this template"** to create your own copy with full version control.

```bash
# Clone your copy of the template
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

## Useful oTree Documentation

- [Installing oTree](https://otree.readthedocs.io/en/latest/install.html) — Python setup and first steps
- [Tutorial](https://otree.readthedocs.io/en/latest/tutorial/intro.html) — learn the basics of oTree
- [Pages](https://otree.readthedocs.io/en/latest/pages.html) — page sequencing, display logic, and timeouts
- [Templates](https://otree.readthedocs.io/en/latest/templates.html) — HTML templates, static files, and styling
- [Models](https://otree.readthedocs.io/en/latest/models.html) — defining data fields and player/group/session models
- [Forms](https://otree.readthedocs.io/en/latest/forms.html) — form fields and validation
- [Treatments](https://otree.readthedocs.io/en/latest/treatments.html) — experimental conditions and between-subjects designs
- [Admin](https://otree.readthedocs.io/en/latest/admin.html) — session management and data export
- [Server setup](https://otree.readthedocs.io/en/latest/server/intro.html) — deploying to Heroku or your own server

## Citation

If you use DICE Lite in your research, please cite:

> Roggenkamp, H., Boegershausen, J., & Hildebrand, C. (2026). DICE: Advancing Social Media Research Through Digital In-Context Experiments. *Journal of Marketing*. https://doi.org/10.1177/00222429251371702

Since DICE Lite is built on oTree, please also cite:

> Chen, D. L., Schonger, M., & Wickens, C. (2016). oTree — An open-source platform for laboratory, online, and field experiments. *Journal of Behavioral and Experimental Finance*, 9, 88–97. https://doi.org/10.1016/j.jbef.2015.12.001

## Links

- [DICE (full version)](https://github.com/Howquez/DICE) — the complete toolkit with GUI-based session creation
- [DICE web app](https://dice-app.org) — create experiments without coding
- [oTree documentation](https://otree.readthedocs.io/en/latest/)

## License

This work is licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).
