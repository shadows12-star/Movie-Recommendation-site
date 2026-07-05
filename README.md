# 🎬 Movie Recommender

A full-stack movie recommendation app combining **content-based filtering (TF-IDF)**, **genre-based recommendations**, and **live TMDB data** (posters, search, trending/popular feeds). Built with a **FastAPI** backend and a **Streamlit** frontend.

---

## ✨ Features

- **Home feed** — browse Trending, Popular, Top Rated, Now Playing, or Upcoming movies (powered by TMDB).
- **Keyword search** — type a title and get autocomplete suggestions plus a matching results grid.
- **Movie details page** — poster, backdrop, overview, release date, and genres.
- **Recommendations**
  - 🔎 **TF-IDF similarity** — content-based recommendations from a locally trained model (`tfidf.pkl`, `tfidf_matrix.pkl`, `indices.pkl`, `df.pkl`).
  - 🎭 **Genre-based** — TMDB "discover" results filtered by the movie's primary genre.
  - Automatic fallback to genre-only recommendations if the TF-IDF/search bundle fails.
- Simple query-param based routing between **Home** and **Details** views in Streamlit (no extra multipage files needed).

---

## 📸 Screenshots

<img width="2559" height="1316" alt="image" src="https://github.com/user-attachments/assets/27eb9d85-90c2-4703-a8ef-a8f894d8eb7f" />
<img width="2056" height="1211" alt="image" src="https://github.com/user-attachments/assets/9dc6abf2-5789-408d-a424-9c1612ec6d2e" />
<img width="2060" height="1301" alt="image" src="https://github.com/user-attachments/assets/acd6e8e1-9424-43d4-838f-021f84f41d9a" />




## 🏗️ Architecture

```
┌─────────────────┐        HTTP        ┌──────────────────┐        HTTPS        ┌───────────┐
│  Streamlit UI    │ ──────────────────▶│   FastAPI Server  │ ──────────────────▶│   TMDB API │
│    (app.py)      │◀────────────────── │    (main.py)      │◀────────────────── │           │
└─────────────────┘      JSON            └──────────────────┘       JSON          └───────────┘
                                                   │
                                                   ▼
                                      Local pickled TF-IDF model
                                (df.pkl, indices.pkl, tfidf.pkl, tfidf_matrix.pkl)
```

- **Backend (`main.py`)** — FastAPI service that wraps the TMDB API and serves local TF-IDF-based recommendations.
- **Frontend (`app.py`)** — Streamlit app that consumes the backend API and renders posters, search, and details.

---

## 📂 Project Structure

```
movie/
├── app.py                  # Streamlit frontend
├── main.py                 # FastAPI backend
├── df.pkl                  # Movie metadata DataFrame (must contain a "title" column)
├── indices.pkl             # Title → row index map (dict or pandas Series)
├── tfidf.pkl               # Fitted TF-IDF vectorizer
├── tfidf_matrix.pkl        # Precomputed TF-IDF matrix (sparse)
├── movie_recommender...    # Notebook/script used to build the model
├── movies_metadata....     # Raw source dataset
├── requirements.txt        # Python dependencies
├── .env                    # Environment variables (TMDB_API_KEY)
├── .python-version
└── .gitignore
```

---

## ⚙️ Setup

### 1. Clone and enter the project
```bash
git clone <your-repo-url>
cd movie
```

### 2. Create a virtual environment
```bash
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Configure environment variables
Create a `.env` file in the project root:
```env
TMDB_API_KEY=your_tmdb_api_key_here
```
> Get a free API key at [themoviedb.org](https://www.themoviedb.org/settings/api).

### 5. Make sure the model files are present
The backend expects these pickle files in the same directory as `main.py`:
- `df.pkl`
- `indices.pkl`
- `tfidf.pkl`
- `tfidf_matrix.pkl`

### 6. Run the backend (FastAPI)
```bash
uvicorn main:app --reload --port 8000
```
The API will be available at `http://127.0.0.1:8000` (docs at `/docs`).

### 7. Run the frontend (Streamlit)
```bash
streamlit run app.py
```

> **Note:** `app.py` currently points `API_BASE` at a deployed Render URL. Update this constant to `http://127.0.0.1:8000` (or your own deployment URL) if you're running the backend locally.

---

## 🔌 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/health` | Health check |
| `GET` | `/home?category={trending\|popular\|top_rated\|now_playing\|upcoming}&limit=24` | Home feed cards |
| `GET` | `/tmdb/search?query={q}&page=1` | Raw TMDB keyword search (used for suggestions + grid) |
| `GET` | `/movie/id/{tmdb_id}` | Full movie details by TMDB ID |
| `GET` | `/recommend/genre?tmdb_id={id}&limit=18` | Genre-based recommendations for a movie |
| `GET` | `/recommend/tfidf?title={title}&top_n=10` | TF-IDF-only recommendations (debug endpoint) |
| `GET` | `/movie/search?query={q}&tfidf_top_n=12&genre_limit=12` | Bundle: details + TF-IDF recs + genre recs for the best-matching movie |

---

## 🧠 How the Recommendations Work

1. **TF-IDF (content-based):** The backend loads a precomputed TF-IDF matrix built from local movie metadata. Given a title, it looks up its row index (`indices.pkl`), computes cosine similarity against all other rows, and returns the top-N most similar titles. Each result is then matched back to a TMDB entry to fetch a poster.
2. **Genre-based:** Given a movie's TMDB ID, the backend reads its first genre and calls TMDB's `/discover/movie` endpoint sorted by popularity to find similar movies in that genre.
3. If the TF-IDF pipeline fails for any reason (e.g., title not found locally), the frontend automatically falls back to genre-only recommendations.

---

## 🛠️ Tech Stack

- **Backend:** FastAPI, httpx, pandas, NumPy, pydantic
- **Frontend:** Streamlit, requests
- **ML:** scikit-learn (TF-IDF vectorizer + cosine similarity, precomputed and pickled)
- **Data source:** [TMDB API](https://www.themoviedb.org/documentation/api)

---

## 📌 Notes / Known Limitations

- `API_BASE` in `app.py` uses `"url" or "fallback_url"` — since the first string is always truthy, it will **always resolve to the Render URL**. Update this if you intend to hardcode a different environment.
- CORS is currently open to all origins (`allow_origins=["*"]`) — fine for local development, but tighten this before deploying publicly.
- The app raises at startup if `TMDB_API_KEY` is missing, and pickle files are assumed to already exist (they are not regenerated automatically).

---

## 📄 License

Add your license of choice here (MIT, Apache 2.0, etc.).
