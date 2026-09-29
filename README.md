# 🎙️ Live AI Podcast Recommendation Engine (Strict Preference Driven)

A full-stack, AI-powered **Live Podcast Recommendation Engine** built with Django, Machine Learning, and Live Online Podcast Discovery APIs. 

The application is strictly governed by user preferences: **Set Preferences is the Single Source of Truth**. The engine dynamically discovers live podcast content online (e.g., Apple Podcasts iTunes Search API, RSS catalogs), normalizes metadata, removes duplicates, and enforces **strict language filtering at the backend recommendation level**.

---

## 📌 Problem Statement

Generic podcast engines suffer from critical limitations:
1. **Static Datasets**: Inability to discover new live podcasts published online.
2. **Silent Language Mixing / Fallbacks**: Silently displaying English or generic content when a user requests podcasts in Telugu, Hindi, or Spanish.
3. **Faked Recommendation Models**: Blending content scores into collaborative results or faking collaborative filtering during cold start.

---

## 🚀 Proposed Solution & Core Architecture

```text
                     User Authentication
                             │
                             ▼
                 Set Preferences (DB Saved)
            (Topics/Interests + Strict Language)
                             │
                             ▼
               Live Online Podcast Discovery
          (iTunes Search API + Live RSS Catalogs)
                             │
                             ▼
              Candidate Normalization & Dedup
                             │
         ┌───────────────────┴───────────────────┐
         ▼                                       ▼
 🎯 Content-Based Engine               👥 Collaborative-Based Engine
 (TF-IDF Cosine Similarity)            (User-Item Interaction Matrix
  Strict Language Filter                Strict Language Filter)
         │                                       │
         ▼                                       ▼
 Top 20 Content Matches                Top 20 Collaborative Matches
 (or Empty State)                      (or Empty State)
```

---

## 🔑 Key System Features

### 1. Set Preferences — Single Source of Truth
* User preferences (`content_preference` and `language_preference`) are saved in the database for each authenticated user.
* All recommendation requests load the saved preferences from the database before candidate retrieval.

### 2. Strict Backend Language Filtering (No Fallbacks)
* **Backend Enforcement**: Language filtering is strictly enforced at the backend recommendation level (`podcast_sources.py`, `content_recommender.py`, `collaborative_recommender.py`).
* **Supported Languages**: English, Hindi (हिंदी), Spanish (Español), French (Français), German (Deutsch), Telugu (తెలుగు), Tamil (தமிழ்), Japanese (日本語), Chinese (中文), Any.
* **No Fallback Policy**: If 0 podcasts match a selected language (e.g., Telugu), the engine returns an empty result (`[]`) and displays an explicit user message. It **NEVER** silently falls back to English or generic content.

### 3. Strict Authentication & Protected Routes
* **Authentication**: Login requires valid Username + Password. Empty credentials or invalid passwords are rejected.
* **Session Security**: Authenticated routes rely strictly on `request.session['username']`.
* **Protected Routes**: `/SetPreferences`, `/UserScreen`, `/Content`, `/MatrixFactor`, `/debug`, and API endpoints require active login.
* **Minimal Home Page**: The root page (`index.html`) is clean and minimal, presenting only Login and Register options.

### 4. Independent Recommendation Engines (No Hybrid)
* **🎯 Content-Based Engine**: Operates independently using TF-IDF sparse vector similarity:
  $$\text{Score}_{\text{Content}} = \frac{V_P \cdot V_{D_i}}{\|V_P\| \|V_{D_i}\|}$$
  where $V_P$ is the user preference vector and $V_{D_i}$ is the podcast composite text vector.
* **👥 Collaborative Filtering Engine**: Operates independently using platform user interaction matrix data (likes=+1.0, saves=+0.8, listens=+0.5, views=+0.2, dislikes=-0.8). Computes User-User Cosine Similarity prediction:
  $$\text{Sim}(u, v) = \frac{\vec{u} \cdot \vec{v}}{\|\vec{u}\| \|\vec{v}\|}$$
* **Removal of Hybrid Recommendation**: Hybrid recommendation tabs and routes have been completely removed to guarantee strict independent separation.

---

## 🗄️ Database Design

Dual-engine database architecture: PyMySQL (MySQL) with zero-configuration fallback to SQLite (`db.sqlite3`).

### 1. `signup` Table
* `username` (VARCHAR 50, PRIMARY KEY)
* `password` (VARCHAR 50)
* `contact_no` (VARCHAR 12)
* `email_id` (VARCHAR 50)
* `address` (VARCHAR 50)

### 2. `preferences` Table
* `username` (VARCHAR 50, PRIMARY KEY)
* `content_preference` (TEXT)
* `language_preference` (VARCHAR 50)
* `updated_at` (TIMESTAMP)

### 3. `interactions` Table
* `id` (INTEGER PRIMARY KEY AUTOINCREMENT / INT AUTO_INCREMENT)
* `username` (VARCHAR 50)
* `podcast_id` (VARCHAR 100)
* `podcast_title` (TEXT)
* `interaction_type` (like / save / listen / dislike / view)
* `interaction_value` (FLOAT)
* `timestamp` (TIMESTAMP)

---

## 🔌 API Endpoints Documentation

| Endpoint | Method | Authentication Required | Description |
|---|---|---|---|
| `/` | GET | No | Minimal Home Page with Login / Register options |
| `/UserLogin` | GET/POST | No | Authenticate user credentials |
| `/Signup` | GET/POST | No | Register new user account |
| `/SetPreferences` | GET/POST | **Yes** | Input natural language topics and select strict language choice |
| `/UserScreen` | GET | **Yes** | Recommendation Dashboard (Content-Based & Collaborative-Based) |
| `/Content` | GET | **Yes** | Independent Content-Based recommendation view |
| `/MatrixFactor` | GET | **Yes** | Independent Collaborative-Based recommendation view |
| `/debug` | GET | **Yes** | Developer & Viva Debug View showing metrics and score breakdowns |
| `/api/interactions` | POST | **Yes** | Asynchronous AJAX endpoint to record user likes, saves, listens, dislikes |
| `/api/recommendations` | POST | **Yes** | Structured JSON API for external clients |
| `/logout` | GET | **Yes** | Terminate active user session |

---

## ⚙️ Environment Variables Setup (`.env`)

Create a `.env` file in the project root directory:

```env
SECRET_KEY=@w^xpfm15$$xbjec+pd#=)l-n-%q863@ryb-!-t+r8@@e8@@9@
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=root
DB_PASSWORD=root
DB_NAME=Podcast
```

---

## 🚀 Running the Application

1. **Verify Python dependencies**:
   ```bash
   pip install Django pandas numpy scikit-learn nltk requests pymysql
   ```

2. **Execute Django System Check**:
   ```bash
   python manage.py check
   ```

3. **Start the Development Server**:
   ```bash
   python manage.py runserver 8000
   ```

4. **Access in Browser**:
   Open **[http://127.0.0.1:8000/](http://127.0.0.1:8000/)** in your browser.

---

## 🧪 Acceptance Test Suite Summary

The application has been verified against all strict core rules:

* [x] **Authentication**: Protected pages cannot be accessed without logging in.
* [x] **Single Source of Truth**: Set Preferences persist and govern all recommendations.
* [x] **Strict Language Enforcement**:
  * `Telugu` returns ONLY Telugu podcasts.
  * `Hindi` returns ONLY Hindi podcasts.
  * `English` returns ONLY English podcasts.
* [x] **No Fallbacks**: Zero matching content returns clean empty states without silent English fallback.
* [x] **Hybrid Removal**: Hybrid tab and hybrid calculations completely removed.
* [x] **Independent Engines**: Content-Based and Collaborative-Based scores do not mix.
