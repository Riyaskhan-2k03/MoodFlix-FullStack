## Repo intent & big picture

This repository is a small Flask-based movie recommender that detects user emotion (webcam or uploaded image) and returns TMDb-based movie suggestions. The main server file in this workspace is `app9.py` (note: README references `app.py` — prefer `app9.py` when modifying or running code in this repo). Core responsibilities are:

- Emotion detection using DeepFace / OpenCV (`detect_emotion_from_webcam`, `detect_emotion_from_image_file` in `app9.py`).
- Movie lookup using TMDb (see `movie_recommender4.py` — `fetch_movies_for_emotion`, `fetch_tmdb_movies_by_ids`).
- Optional external recommender integration via `RECOMMENDER_API_URL` (handled in `call_external_recommender` inside `app9.py`).

## Entrypoints & how to run (developer workflows)

- Create a virtualenv and install deps from `requirements.txt` (README shows commands). On Windows:
  - `python -m venv venv` then `venv\Scripts\activate` and `pip install -r requirements.txt`.
- Set environment variables in a local `.env` (README mentions `.env.example`). Important variables:
  - `TMDB_API_KEY` (required for TMDb calls)
  - `RECOMMENDER_API_URL` (optional external recommender)
  - `PORT`, `FLASK_ENV`, `FLASK_SECRET`
- Run the app for local development using the actual file present: `python app9.py`. If you see an `app.py` in other branches, check for duplication before changing README.

## Project-specific patterns & conventions

- Movie objects: the project uses a simple movie dict shape: `{title, overview, poster, release_date, tmdb_id}`. Keep this shape when adding or returning recommendations.
- External recommender responses are normalized in `call_external_recommender` — handle any of these shapes: `{"movies": [...]}`, `{"ids": [...]}`, `{"recommended_ids": [...]}`, or a bare list of ids/dicts. Mirror this tolerant approach when adding integrations.
- Emotion values are plain lowercase strings (e.g. `"happy"`, `"sad"`, `"neutral"`). Map them via `EMOTION_GENRE_MAP` in `movie_recommender4.py` if you need built-in genre-based fallbacks.
- DeepFace calls are invoked with `enforce_detection=False` to be permissive; code resets uploaded file streams so Flask can re-use them — preserve that behavior when refactoring file handling.

## Defensive behavior & error handling observed

- If `TMDB_API_KEY` is missing the fetch functions raise `RuntimeError` — tests and callers expect this. When writing code that calls `movie_recommender4.py`, either ensure env var exists or catch/handle that RuntimeError.
- Network calls (TMDb, external recommender) are wrapped with try/except and logged; continue-on-error semantics are used (partial failures shouldn't crash the app). Use `app.logger` or `app.logger.exception` for server-side logs.

## Files & locations to inspect when making changes

- `app9.py` — main Flask app, templates endpoints and emotion detection logic (web + API endpoints `/recommend`, `/api/recommend`).
- `movie_recommender4.py` — TMDb fetch helpers and `EMOTION_GENRE_MAP`.
- `requirements.txt` — pinned dependencies (DeepFace-related deps are provided indirectly; OpenCV headless and `fer`/`mtcnn` are present).
- `data/`, `demo/`, and `artifacts/` — look here for example inputs, demo assets, or caches if present.

## Small examples for the agent to follow

- To return a recommendation array from server code, produce a list of movie dicts matching the repo shape and call `render_template("results.html", mood=emotion, recommendations=recommended)` (see `recommend()` in `app9.py`).
- To call the external recommender, send JSON `{ "emotion": "happy" }` and accept these response shapes; convert ids to TMDb lookups via `fetch_tmdb_movies_by_ids` when necessary.

## Quick checks before edits or PRs

1. Ensure `TMDB_API_KEY` is available in `.env` (or mock it in tests).
2. If adding image-processing code, prefer reusing `detect_emotion_from_image_file` which handles stream resets and OpenCV decoding.
3. Maintain the public movie dict shape for API compatibility.

If anything here is unclear or you want the agent to include examples for unit tests or CI steps, tell me which areas to expand and I will iterate.
