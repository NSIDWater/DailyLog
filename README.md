# WTP Operator Daily Log

A web application for water treatment plant operators to record daily shift logs, with a manager dashboard for reviewing entries.

## Features

- Operator daily log: shift info, tank levels, clearwell testing, finished water, pump settings, altitude valve log
- Langelier Saturation Index auto-calculation
- Chlorine & ammonia adjustment log
- Manager dashboard with log history, filtering, and CSV export
- All data saved to SQLite database
- Session-based authentication

## Credentials

| Role | Username | Password |
|------|----------|----------|
| Manager | `Admin` | `Admin` |
| Operator | *(any username/password you create)* | — |

> **Important:** Change the Admin password before going live. See below.

## Local Setup

```bash
# 1. Install dependencies
npm install

# 2. Start the server
npm start

# 3. Open in browser
http://localhost:3000
```

## Deploying to Render (Free Hosting)

1. Push this repo to GitHub
2. Go to [render.com](https://render.com) and create a free account
3. Click **New → Web Service**
4. Connect your GitHub repo
5. Set these options:
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`
   - **Environment:** Node
6. Add environment variable:
   - `SESSION_SECRET` → any long random string (e.g. `wtp-plant-secret-2024-xk9mq`)
7. Click **Deploy**

Your app will be live at `https://your-app-name.onrender.com`

> **Note on Render free tier:** The SQLite database file (`wtp.db`) is stored on the server's disk. Render free tier has **ephemeral storage** — the database resets on redeploy. For persistent storage, upgrade to a paid Render plan or use [Railway](https://railway.app) which supports persistent disk on its free tier.

## Deploying to Railway (Recommended for persistence)

1. Push to GitHub
2. Go to [railway.app](https://railway.app)
3. New Project → Deploy from GitHub
4. Select your repo
5. Add environment variable: `SESSION_SECRET=your-secret-here`
6. Railway auto-detects Node.js and deploys

Railway gives you 500 hours/month free and **persistent disk storage**.

## Changing the Admin Password

Connect to your server and run:

```bash
node -e "
const bcrypt = require('bcryptjs');
const hash = bcrypt.hashSync('YOUR_NEW_PASSWORD', 10);
console.log(hash);
"
```

Then update the hash in `db/database.js` in the `initSchema()` function, or run a direct SQL update against `wtp.db`.

## Adding Operator Accounts

Currently all users log in with any credentials (the server accepts any login for operators). To restrict access, add users to the database:

```bash
node -e "
const initSqlJs = require('sql.js');
const bcrypt = require('bcryptjs');
const fs = require('fs');
async function run() {
  const SQL = await initSqlJs();
  const db = new SQL.Database(fs.readFileSync('wtp.db'));
  const hash = bcrypt.hashSync('operatorpassword', 10);
  db.run('INSERT OR IGNORE INTO users (username, password_hash, role) VALUES (?, ?, ?)',
    ['OperatorName', hash, 'operator']);
  fs.writeFileSync('wtp.db', Buffer.from(db.export()));
  console.log('User added');
}
run();
"
```

## Project Structure

```
wtp-log/
├── server.js          # Express server
├── package.json
├── .gitignore
├── db/
│   └── database.js    # SQLite setup and helpers
├── routes/
│   ├── auth.js        # Login / logout / session
│   └── logs.js        # Save and retrieve daily logs
└── public/
    ├── index.html     # Operator log interface
    └── dashboard.html # Manager dashboard
```
