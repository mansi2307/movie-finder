# Movie Night Picker 🎬

A couples' movie picker — add films, filter by mood, and let fate decide what to watch tonight.

**Stack:** React + Vite (frontend) · Flask (backend) · PostgreSQL (database) · Render (hosting)

---

## Project structure

```
movie-picker/
├── backend/
│   ├── app.py            # Flask app + all API routes
│   ├── seed.py           # Creates tables + optional sample data
│   ├── requirements.txt
│   ├── Procfile          # Tells Render how to start the server
│   └── .env.example
├── frontend/
│   ├── src/
│   │   ├── main.jsx
│   │   ├── App.jsx
│   │   ├── api.js        # All fetch calls to the backend
│   │   └── components/
│   │       ├── AddMovies.jsx
│   │       ├── PickMovie.jsx
│   │       └── MovieList.jsx
│   ├── index.html
│   ├── vite.config.js
│   └── .env.example
└── .gitignore
```

---

## Local development

### 1. Prerequisites
- Python 3.10+
- Node.js 18+
- PostgreSQL running locally

### 2. Backend setup

```bash
cd backend

# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate       # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
cp .env.example .env
# Edit .env and set your local DATABASE_URL, e.g.:
# DATABASE_URL=postgresql://postgres:password@localhost:5432/moviepicker

# Create the database in PostgreSQL first
psql -U postgres -c "CREATE DATABASE moviepicker;"

# Create tables (and optionally seed sample movies)
python seed.py           # tables only
python seed.py --seed    # tables + sample movies

# Start the Flask dev server
flask --app app run --debug
# Backend is now running at http://localhost:5000
```

### 3. Frontend setup

```bash
cd frontend

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
# Default points to http://localhost:5000 — leave as-is for local dev

# Start the dev server
npm run dev
# Frontend is now running at http://localhost:5173
```

---

## API reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Health check |
| GET | `/api/movies` | List movies (supports `?genre=`, `?added_by=`, `?watched=`) |
| POST | `/api/movies` | Add a movie `{ title, genre, added_by }` |
| PATCH | `/api/movies/:id` | Update `watched` or `rating` |
| DELETE | `/api/movies/:id` | Remove a movie |
| GET | `/api/pick` | Random pick (supports `?genre=`, `?exclude=id`) |
| GET | `/api/stats` | Counts by person, genre, watched status |

---

## Deploying to Render

You'll create **3 things** on Render: a PostgreSQL database, a backend web service, and a static frontend site.

### Step 1 — Push to GitHub

```bash
# From the movie-picker/ root
git init
git add .
git commit -m "Initial commit"
# Create a repo on GitHub, then:
git remote add origin https://github.com/YOUR_USERNAME/movie-picker.git
git push -u origin main
```

### Step 2 — Create a PostgreSQL database on Render

1. Go to [render.com](https://render.com) → **New** → **PostgreSQL**
2. Give it a name (e.g. `movie-picker-db`)
3. Choose the free plan
4. Click **Create Database**
5. Copy the **Internal Database URL** — you'll need it in the next step

### Step 3 — Deploy the backend

1. **New** → **Web Service** → connect your GitHub repo
2. Set **Root Directory** to `backend`
3. Set **Runtime** to `Python 3`
4. **Build command:** `pip install -r requirements.txt && python seed.py`
5. **Start command:** `gunicorn app:app`
6. Add **Environment Variables:**
   - `DATABASE_URL` → paste the Internal Database URL from Step 2
   - `FRONTEND_URL` → you'll fill this in after the frontend is deployed (use `*` for now)
7. Click **Create Web Service**
8. Copy the backend URL (e.g. `https://movie-picker-api.onrender.com`)

### Step 4 — Deploy the frontend

1. **New** → **Static Site** → connect your GitHub repo
2. Set **Root Directory** to `frontend`
3. **Build command:** `npm install && npm run build`
4. **Publish directory:** `dist`
5. Add **Environment Variables:**
   - `VITE_API_URL` → your backend URL from Step 3
6. Click **Create Static Site**
7. Copy the frontend URL

### Step 5 — Update CORS

Go back to your **backend** service on Render → Environment → update `FRONTEND_URL` to your frontend URL from Step 4.

---

## Adding movies directly to the database

Connect to your PostgreSQL database with `psql` or any GUI (TablePlus, pgAdmin, DBeaver):

```sql
-- Add a movie
INSERT INTO movies (title, genre, added_by)
VALUES ('The Favourite', 'Dark Comedy', 'you');

-- View all unwatched movies
SELECT * FROM movies WHERE watched = false ORDER BY id DESC;

-- Mark a movie as watched with a rating
UPDATE movies SET watched = true, rating = 4 WHERE title = 'The Favourite';

-- Delete a movie
DELETE FROM movies WHERE title = 'The Favourite';
```

---

## Genres supported

Dark Comedy · Romance · Thriller · Horror · Action · Drama · Sci-Fi · Documentary · Animation · Comedy

To add more genres, update the `GENRES` array in:
- `frontend/src/components/AddMovies.jsx`
- `frontend/src/components/PickMovie.jsx`
- `frontend/src/components/MovieList.jsx`
