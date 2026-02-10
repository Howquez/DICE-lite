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

## Architecture

Understanding how data flows through DICE Lite makes it straightforward to add new features or modify existing ones.

### Data Flow: CSV → Backend → Frontend → Data Collection

```
CSV file                 __init__.py                    C_Feed.html / T_Item_Post.html
┌──────────┐    read_feed()     ┌──────────────┐    vars_for_template()    ┌────────────────────┐
│ doc_id   │───────────────────►│  DataFrame   │──────────────────────────►│  {{ for i in       │
│ text     │  preprocessing()   │  stored per  │   posts dict passed to   │     posts.values() │
│ likes    │  (format dates,    │  participant │   template context       │  }}                │
│ ...      │   highlight tags,  │              │                          │    include         │
│          │   prepare media)   │              │                          │    T_Item_Post     │
└──────────┘                    └──────────────┘                          └────────────────────┘
                                                                                   │
                                                                                   ▼
                             __init__.py                    JS (like_button.js, ...)
                                ┌──────────────┐   form submission        ┌────────────────────┐
                                │ Player model │◄──────────────────────── │ collectLikes()     │
                                │ fields       │   JSON serialized to     │ collectReplies()   │
                                │              │   hidden form fields     │ etc.               │
                                └──────────────┘                          └────────────────────┘
```

**Step by step:**

1. **CSV → Backend** (`__init__.py`): `read_feed()` loads your CSV. `preprocessing()` formats dates, highlights hashtags/mentions, prepares media URLs, and builds user profile tooltips. The resulting DataFrame is stored in `player.participant.tweets`.

2. **Backend → Frontend** (`C_Feed.html`): `vars_for_template()` converts the DataFrame to a dictionary and passes it to the template. `C_Feed.html` loops through each post with `{{ for i in posts.values() }}` and includes `T_Item_Post.html` for each one — rendering it as a table row (`<tr>`).

3. **Frontend interactions** (`T_Item_Post.html` + JS): Each post renders action buttons (reply, repost, like, share). JavaScript files handle the interactive behavior — `like_button.js` toggles icons and increments/decrements counts, tracks replies, and monitors sponsored post clicks.

4. **Data collection** (`like_button.js` → `__init__.py`): When the participant clicks a submit button, `collectDataHarmonized()` calls `collectLikes()`, `collectReplies()`, etc. and writes the JSON-serialized results into hidden `<input>` fields defined in `C_Feed.html`. These hidden fields are submitted with the form and map to the `Player` model fields defined in `__init__.py`.

### Key Files by Role

| What you want to change | Where to look |
|--------------------------|---------------|
| Post appearance (layout, buttons, icons) | `T_Item_Post.html` |
| Feed-level layout (sidebar, search bar) | `C_Feed.html` |
| Interaction logic (like toggle, reply modal) | `static/js/like_button.js` |
| Repost/share toggle animation | `static/js/interactions.js` |
| Dwell time tracking | `static/js/dwell.js` |
| Data fields and processing pipeline | `__init__.py` (Player class, preprocessing) |
| Study-level settings (data source, thresholds) | `settings.py` |

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

### Walkthrough: Adding a Dislike Button

This example walks through every file you'd need to touch to add a new "dislike" (thumbs-down) interaction — illustrating how the architecture connects end to end.

#### 1. Add the button HTML (`T_Item_Post.html`)

In the post actions `<div>` (where reply, repost, like, and share buttons are defined), add a dislike button. You'll find two action sections — one for sponsored posts and one for regular posts. Add the dislike button to both:

```html
<!-- Dislike -->
<div class="dislike-button col" id="dislike_button_{{i.doc_id}}">
    <span class="bi bi-hand-thumbs-down text-secondary dislike-icon" style="cursor: pointer">️</span>
    <span class="dislike-count text-secondary">0</span>
</div>
```

Place this after the like button `<div>` and before the share button `<div>`.

#### 2. Add the interaction logic (`static/js/like_button.js`)

Add a toggle function (mirroring `toggleLike`):

```javascript
function toggleDislike(button) {
    const icon = button.querySelector('.dislike-icon');
    const countSpan = button.querySelector('.dislike-count');
    let count = parseInt(countSpan.textContent);

    if (icon.classList.contains('bi-hand-thumbs-down')) {
        icon.classList.remove('bi-hand-thumbs-down', 'text-secondary');
        icon.classList.add('bi-hand-thumbs-down-fill', 'text-primary');
        count++;
    } else {
        icon.classList.remove('bi-hand-thumbs-down-fill', 'text-primary');
        icon.classList.add('bi-hand-thumbs-down', 'text-secondary');
        count--;
    }
    countSpan.textContent = count.toString();
}
```

Attach click listeners (mirroring the like button pattern):

```javascript
document.querySelectorAll('.dislike-button').forEach(button => {
    button.addEventListener('click', function() {
        toggleDislike(button);
    });
});
```

Add a data collection function (mirroring `collectLikes`):

```javascript
function collectDislikes() {
    let dislikesData = [];
    document.querySelectorAll('.dislike-button').forEach(button => {
        let docId = parseInt(button.getAttribute('id').replace('dislike_button_', ''));
        let icon = button.querySelector('.dislike-icon');
        let isDisliked = icon.classList.contains('bi-hand-thumbs-down-fill');
        dislikesData.push({ doc_id: docId, disliked: isDisliked });
    });
    return dislikesData;
}
```

Then call `collectDislikes()` in the existing form submission handler (the `submitButton` click listener) and assign the result to a hidden form field:

```javascript
document.getElementById('dislikes_data').value = JSON.stringify(collectDislikes());
```

#### 3. Add a Player field and wire it up (`__init__.py`)

Add a field to the `Player` class:

```python
dislikes_data = models.LongStringField(doc='tracks dislikes.', blank=True)
```

Include it in `C_Feed.get_form_fields()` so oTree knows to collect it from the form:

```python
fields = ['likes_data', 'replies_data', 'dislikes_data', 'promoted_post_clicks', ...]
```

#### 4. Add the hidden form field (`C_Feed.html`)

DICE Lite uses explicit hidden `<input>` fields to pass JavaScript-collected data to the backend — oTree does **not** auto-generate these for you. Add a hidden input alongside the existing ones (e.g. `likes_data`, `viewport_data`):

```html
<input type="hidden" name="dislikes_data" id="dislikes_data" value="">
```

#### 5. Wire up the form submission (`static/js/like_button.js`)

The `collectDataHarmonized()` function gathers all interaction data right before the form is submitted. Add your new collection call there:

```javascript
function collectDataHarmonized() {
    let likesData = collectLikes();
    let repliesData = collectReplies();
    let dislikesData = collectDislikes();
    let promotedClicksData = JSON.parse(document.getElementById('promoted_post_clicks').value);
    return {
        likes: JSON.stringify(likesData),
        replies: JSON.stringify(repliesData),
        dislikes: JSON.stringify(dislikesData),
        promoted_clicks: JSON.stringify(promotedClicksData)
    };
}
```

Then, in **both** submit button click handlers (`submitButtonTop` and `submitButtonBottom`), write the data to the hidden field:

```javascript
document.getElementById('dislikes_data').value = data.dislikes;
```

That's it — five files, following the same patterns already used by the like button.

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
