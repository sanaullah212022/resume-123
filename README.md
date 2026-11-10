# 📄 AI Resume ATS Checker

A Streamlit app that lets users upload a resume (PDF, DOCX or TXT), optionally paste a job description, and receive:

- An **estimated ATS score** (0-100) with a breakdown by category
- **Prioritized, actionable suggestions** for improving the resume
- **Keyword analysis**: keywords found and important ones missing
- **Rewrite examples** for weak bullet points
- A downloadable Markdown report

It uses the Google Gemini API (Flash model) for the analysis.

> **Note:** The score is an AI estimate designed to guide improvements. It is not the output of a real ATS, and different employers' systems behave differently.

## How the score works

Gemini rates five categories from 0 to 100 and returns structured JSON. The app validates the response and computes the overall score itself, so it is consistent with the category scores:

| Category | Weight |
|---|---|
| Formatting & ATS-friendliness | 20% |
| Keywords & skills | 25% |
| Achievements & impact | 25% |
| Clarity & grammar | 15% |
| Completeness of sections | 15% |

If a job description is provided, keywords are judged against it and a separate job-match score is shown.

## Project structure

```
.
├── app.py              # Streamlit app (UI, file parsing, Gemini call)
├── requirements.txt    # Python dependencies
├── README.md
├── .gitignore          # (create it, see below)
└── .streamlit/
    └── secrets.toml    # LOCAL ONLY, never commit (create it, see below)
```

## Run locally

Requires Python 3.10+.

```bash
# 1. Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Provide your Gemini API key (get a free one at https://aistudio.google.com/apikey)
mkdir -p .streamlit
echo 'GEMINI_API_KEY = "your-key-here"' > .streamlit/secrets.toml
# (alternatively: export GEMINI_API_KEY="your-key-here", or paste it in the app sidebar)

# 4. Start the app
streamlit run app.py
```

Open the URL Streamlit prints (usually http://localhost:8501).

## Configuration

| Setting | Where | Purpose |
|---|---|---|
| `GEMINI_API_KEY` | `secrets.toml`, environment variable, or sidebar | API key. A key typed in the sidebar overrides the configured one. |
| `GEMINI_MODEL` | `secrets.toml`, environment variable, or sidebar → Advanced | Model name. Defaults to `gemini-3.6-flash`. |

Google retires and renames models over time. If you see a "model name was not found" error, set `GEMINI_MODEL` to a current Flash model from https://ai.google.dev/gemini-api/docs/models (for example `gemini-flash-latest`, which always points to the newest Flash but can change behavior without notice).

## Push the project to GitHub

1. **Create `.gitignore`** in the project folder so secrets never get committed:

   ```gitignore
   .venv/
   __pycache__/
   *.pyc
   .env
   .streamlit/secrets.toml
   ```

2. **Create an empty repository** on GitHub (https://github.com/new). Give it a name such as `ats-resume-checker`. Do not add a README or .gitignore there, since you already have them locally. Public or private both work with Streamlit Community Cloud.

3. **Initialize git and push** from your project folder:

   ```bash
   git init
   git add app.py requirements.txt README.md .gitignore
   git status                     # confirm secrets.toml is NOT listed
   git commit -m "Initial commit: AI resume ATS checker"
   git branch -M main
   git remote add origin https://github.com/<your-username>/ats-resume-checker.git
   git push -u origin main
   ```

   When prompted for a password, GitHub requires a **personal access token** (Settings → Developer settings → Personal access tokens), not your account password. Alternatively, use GitHub Desktop or the GitHub CLI (`gh auth login`, then `gh repo create`).

> **If you ever commit your API key by accident:** revoke it immediately in Google AI Studio and create a new one. Deleting it from a later commit does not remove it from git history.

## Deploy on Streamlit Community Cloud

1. Go to https://share.streamlit.io and sign in with your GitHub account (authorize access to the repository if asked).
2. Click **Create app** (upper-right), then choose **"Yup, I have an app."**
3. Fill in:
   - **Repository:** `<your-username>/ats-resume-checker`
   - **Branch:** `main`
   - **Main file path:** `app.py`
   - **App URL:** optional custom subdomain on `streamlit.app`
4. Click **Advanced settings**:
   - Pick a **Python version** (3.11 or 3.12 works well).
   - In **Secrets**, paste:

     ```toml
     GEMINI_API_KEY = "your-key-here"
     ```

   - Click **Save**.
5. Click **Deploy**. The first build installs everything in `requirements.txt` and usually takes a few minutes.
6. Share the `*.streamlit.app` URL.

**Updating the app:** push new commits to the `main` branch and the app redeploys automatically. To change secrets later, open the app's **Settings** in your Streamlit workspace and edit the Secrets field.

> Button names and menus on share.streamlit.io occasionally change; the official guide is at https://docs.streamlit.io/deploy/streamlit-community-cloud/deploy-your-app/deploy.

### Protect your quota on a public app

If you store `GEMINI_API_KEY` in Secrets, every visitor's analysis uses *your* key and quota. Options:

- Leave the secret out so each visitor pastes their own key in the sidebar (the app supports this).
- Keep the secret but make the app private (restrict viewers in app sharing settings).
- Set a spending/quota limit on your key in Google AI Studio / Google Cloud.

## Privacy

Resume text is sent to Google's Gemini API for analysis. The app itself does not save uploaded files or results to disk. Depending on your Gemini API plan, Google may retain or use submitted data per its terms, so tell your users, and avoid using free-tier keys for sensitive production data.

## Troubleshooting

| Problem | Fix |
|---|---|
| "Very little text could be extracted" | The resume is probably a scanned image. Export a text-based PDF or DOCX. (Real ATS software struggles with these too.) |
| "API key" / 400-403 error | Check the key is valid, pasted without quotes or spaces, and has access to the model. |
| "model name was not found" | Change `GEMINI_MODEL` (see Configuration). |
| Rate limit / quota (429) | Wait a minute, or use a key with higher limits. |
| Build fails on Streamlit Cloud | Check the logs; make sure `requirements.txt` is in the repo root. |

## Limitations

- Text is extracted from the file, so the model cannot see visual layout (columns, graphics, fonts). Formatting feedback is inferred from the extracted text.
- Scores come from an LLM and can vary slightly between runs (temperature is set low to reduce this).
- Only the first ~30,000 characters of a resume and 8,000 characters of a job description are analyzed.
