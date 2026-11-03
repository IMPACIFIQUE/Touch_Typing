# TypeFlow — Touch Typing Trainer

A full-stack typing trainer that teaches touch typing technique, tracks speed and accuracy, and stores every practice session in MongoDB.

- **Frontend:** React 19 + Vite + React Router + Axios
- **Backend:** Node.js + Express + JWT authentication
- **Database:** MongoDB + Mongoose
- **Features:** guided lessons, live WPM/accuracy, on-screen keyboard, session history, statistics chart, streaks, achievements and a leaderboard

---

## Project structure

```
Touch_typing/
├── client/                     # React frontend (Vite)
│   ├── public/
│   ├── src/
│   │   ├── api/                # Axios client + endpoint modules
│   │   ├── components/
│   │   │   ├── auth/           # ProtectedRoute
│   │   │   ├── layout/         # Navbar, Footer, Layout
│   │   │   ├── lessons/        # LessonCard
│   │   │   ├── typing/         # TypingArea, Keyboard, LiveStats, ResultCard
│   │   │   └── ui/             # Spinner
│   │   ├── context/            # Auth context/provider
│   │   ├── hooks/              # useAuth, useTypingEngine
│   │   ├── pages/              # Home, Lessons, Practice, Dashboard, ...
│   │   ├── styles/             # Global stylesheet / design tokens
│   │   ├── utils/              # Formatters, token storage, practice texts
│   │   ├── App.jsx             # Routes
│   │   └── main.jsx            # Entry point
│   ├── index.html
│   ├── vite.config.js          # Dev proxy: /api -> localhost:5000
│   └── package.json
│
├── server/                     # Express API
│   ├── src/
│   │   ├── config/             # env + MongoDB connection
│   │   ├── models/             # User, Lesson, PracticeSession, Achievement
│   │   ├── controllers/        # auth, lesson, session, stats, leaderboard
│   │   ├── routes/             # /api/v1 routes
│   │   ├── middleware/         # auth, validation, errors, rate limit
│   │   ├── validators/         # request body validation
│   │   ├── utils/              # ApiError, tokens, typing metrics, achievements
│   │   ├── seed/               # Lessons + achievements seed data
│   │   ├── app.js              # Express app
│   │   └── server.js           # Startup + graceful shutdown
│   ├── .env.example
│   └── package.json
│
├── database/                   # MongoDB setup and schema documentation
│   └── README.md
│
├── package.json                # Root scripts (run both apps at once)
└── README.md
```

---

## Quick start

### Prerequisites

- Node.js 20+ (tested on Node 22)
- MongoDB 6+ running locally, or a MongoDB Atlas connection string

### 1. Install dependencies

```bash
npm run install:all
```

Or install each package separately:

```bash
npm install
npm --prefix server install
npm --prefix client install
```

### 2. Configure the environment

`server/.env` is created for local development with these defaults:

```env
NODE_ENV=development
PORT=5000
MONGO_URI=mongodb://127.0.0.1:27017/typeflow
JWT_SECRET=dev-only-secret-change-me-before-deploying
JWT_EXPIRES_IN=7d
BCRYPT_ROUNDS=12
CLIENT_URL=http://localhost:5173
```

For MongoDB Atlas, replace `MONGO_URI` with your connection string. Always change `JWT_SECRET` before deploying.

The frontend talks to `/api` during development, and Vite proxies that to `http://localhost:5000`. To point the frontend at a different API, create `client/.env`:

```env
VITE_API_URL=https://your-api.example.com/api/v1
```

### 3. Seed the database

```bash
npm run seed
```

This upserts the 12 lessons and 12 achievements into the `typeflow` database. It is safe to run multiple times.

### 4. Run the app

```bash
npm run dev
```

- Frontend: http://localhost:5173
- API: http://localhost:5000
- Health check: http://localhost:5000/health

You can also run the two apps in separate terminals:

```bash
npm run dev:server
npm run dev:client
```

---

## Using the app

1. Open http://localhost:5173.
2. Try **Practice** without an account (results are not saved).
3. Create an account to save sessions, build a streak and unlock achievements.
4. Work through **Lessons** in order: home row → top row → bottom row → numbers → punctuation → speed → code.
5. Check **Dashboard** for best WPM, accuracy, weekly chart, recent sessions and achievements.
6. Check **Leaderboard** for all-time and weekly rankings.

---

## API overview

Base URL: `http://localhost:5000/api/v1`

| Method | Endpoint | Auth | Description |
| --- | --- | --- | --- |
| POST | `/auth/register` | — | Create an account, returns JWT |
| POST | `/auth/login` | — | Sign in, returns JWT |
| POST | `/auth/logout` | Bearer | Stateless sign out |
| GET | `/auth/me` | Bearer | Current user profile |
| PATCH | `/auth/me` | Bearer | Update name, email, settings or password |
| GET | `/lessons` | — | List lessons (`?level=&category=&search=&page=&limit=`) |
| GET | `/lessons/:slug` | — | Get a lesson by slug |
| POST | `/lessons` | Admin | Create a lesson |
| PATCH | `/lessons/:id` | Admin | Update a lesson |
| DELETE | `/lessons/:id` | Admin | Delete a lesson |
| POST | `/sessions` | Bearer | Record a practice session |
| GET | `/sessions` | Bearer | List your sessions |
| GET | `/sessions/:id` | Bearer | Get one of your sessions |
| DELETE | `/sessions/:id` | Bearer | Delete one of your sessions |
| GET | `/stats/me` | Bearer | Summary, 7-day chart, recent sessions, achievements |
| GET | `/leaderboard` | — | Rankings (`?period=all|week&limit=`) |
| GET | `/health` | — | Service health |

Authenticated requests use:

```
Authorization: Bearer <token>
```

### Recording a session

The client sends the raw counts and elapsed time; the server recalculates WPM and accuracy so scores are not simply trusted from the browser:

```json
{
  "lessonSlug": "home-row-basics",
  "mode": "lesson",
  "content": "asdf jkl; ...",
  "typedChars": 93,
  "correctChars": 91,
  "elapsedMs": 60000
}
```

The response includes the stored session, updated user statistics and any newly unlocked achievements.

---

## Database

See [`database/README.md`](database/README.md) for the MongoDB setup, collection schemas, relationships and indexes.

Database name: **typeflow**

Collections:

- `users` — accounts, aggregate statistics, unlocked achievements
- `lessons` — lesson content and goals
- `practicesessions` — every recorded practice result
- `achievements` — achievement catalog

---

## Scripts

| Command | Description |
| --- | --- |
| `npm run install:all` | Install root, server and client dependencies |
| `npm run dev` | Run API + frontend together |
| `npm run dev:server` | Run the API with nodemon |
| `npm run dev:client` | Run the Vite dev server |
| `npm run seed` | Seed lessons and achievements |
| `npm run build` | Build the frontend for production |
| `npm run start` | Start the API in production mode |
| `npm run lint` | Lint the frontend |

---

## Production notes

- Set `NODE_ENV=production`, a strong `JWT_SECRET` and your real `MONGO_URI`.
- Set `CLIENT_URL` to the deployed frontend origin (CORS allow-list).
- Build the frontend with `npm run build` and serve `client/dist` from a static host or reverse proxy.
- The API is stateless (Bearer JWT), so it can be scaled horizontally.
- Add an admin user manually in MongoDB (`role: "admin"`) if you want to manage lessons through the API.
