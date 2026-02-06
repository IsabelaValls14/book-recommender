# Book Recommender

Semantic book recommendations built from the public Kaggle dataset **“7k Books With Metadata”**, combining structured metadata, lightweight NLP enrichment, and vector search to enable natural-language discovery of books by description and emotional tone. The workflow:
- explore and clean the raw dataset,
- assign lightweight categories and emotion scores,
- export artifacts (`books_with_emotions.csv`, `tagged_description.txt`),
- serve a Gradio UI that searches by description using OpenAI embeddings + a Chroma vector index.

## Data source (Kaggle via kagglehub)
- Dataset handle: `dylanjcastillo/7k-books-with-metadata`.
- Download (uses the lightweight `kagglehub` client, no CLI install needed):
  ```bash
  python - <<'PY'
  import kagglehub
  path = kagglehub.dataset_download("dylanjcastillo/7k-books-with-metadata")
  print(f"Path to dataset files: {path}")
  PY
  ```
  The default cache path on macOS looks like  
  `~/.cache/kagglehub/datasets/dylanjcastillo/7k-books-with-metadata/versions/3`.
- If the download asks for auth, set your Kaggle API token in `~/.kaggle/kaggle.json` **or** export:
  ```bash
  export KAGGLE_USERNAME="your_username"
  export KAGGLE_KEY="your_token"
  # rerun the python snippet above
  ```

## Setup (macOS zsh)
```bash
git clone https://github.com/IsabelaValls14/book-recommender.git
cd book-recommender
python3 -m venv venv-jupes          # matches .vscode settings
source venv-jupes/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```
- The app uses OpenAI embeddings; put your key in `.env` or export it:
  ```bash
  echo 'OPENAI_API_KEY=sk-...' >> .env   # or export OPENAI_API_KEY=...
  ```

## Jupyter / VS Code kernel setup
Register the venv as a notebook kernel (name used below matches the user’s existing kernel):
```bash
python -m pip install -U ipykernel
python -m ipykernel install --user \
  --name book-recommender \
  --display-name "book-recommender (.venv)"
```
Then in VS Code:
- Cmd+Shift+P → “Notebook: Select Notebook Kernel” → choose **book-recommender (.venv)**.
- Ensure VS Code points to the venv (`.vscode/settings.json` currently targets `venv-jupes/bin/python`).

**Common pitfalls**
- Kernel not listed: reopen VS Code after running the `ipykernel install` command.
- Wrong interpreter: check the Python interpreter in the bottom-right status bar matches your venv.
- Stuck on “Select Kernel”: run `jupyter kernelspec list` and remove stale entries, then reinstall the kernel.

## Run the notebooks end-to-end
1) **Download data** (snippet above).  
2) **data-exploration copy.ipynb** – reads `books.csv` from the Kaggle cache and writes `books_cleaned.csv`.  
3) **text-classification.ipynb** – loads `books_cleaned.csv`, tags simple categories, writes `books_with_categories.csv`.  
4) **sentiment-analysis.ipynb** – adds emotion scores, writes `books_with_emotions.csv`.  
5) **vector-search.ipynb** – prepares `tagged_description.txt` from `books_cleaned.csv` (used to build the vector store).  

Emotion scores are derived using lightweight NLP/sentiment analysis techniques rather than full supervised emotion classifiers.

Keep the generated CSV/TXT files in the repo root so the Gradio app can find them.

## Run the Gradio recommender
```bash
source venv-jupes/bin/activate           # if not already active
python gradio-dashboard.py
```
The UI starts locally (default http://127.0.0.1:7860) and uses:
- `books_with_emotions.csv` for book metadata + emotion scores
- `tagged_description.txt` + `OPENAI_API_KEY` for the Chroma/OpenAI embedding index

## Repository structure
- `data-exploration copy.ipynb` – initial cleaning, exports `books_cleaned.csv`.
- `text-classification.ipynb` – lightweight category assignment, exports `books_with_categories.csv`.
- `sentiment-analysis.ipynb` – emotion scoring, exports `books_with_emotions.csv`.
- `vector-search.ipynb` – builds the text corpus for semantic search, exports `tagged_description.txt`.
- `gradio-dashboard.py` – Gradio UI for semantic book recommendations (needs the exported files + OpenAI key).
- `requirements.txt` – pinned dependencies for notebooks + app.
- `.vscode/settings.json` – points VS Code to `venv-jupes/bin/python` and a local Jupyter server.

## Future work
- Add a small CLI to regenerate all artifacts in order.
- Persist the Chroma index to disk for faster app startup.
- Add tests/notebook checkpoints to guard against data drift and schema changes.
- The project emphasizes reproducible data ingestion, explicit intermediate artifacts, and clear separation between data preparation and serving layers.
