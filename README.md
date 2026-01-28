# Unolo Field Force Tracker

A web application for tracking field employee check-ins at client locations.

## Tech Stack

- **Frontend:** React 18, Vite, Tailwind CSS, React Router
- **Backend:** Node.js, Express.js, SQLite
- **Authentication:** JWT

## Quick Start

### 1. Backend Setup

```bash
cd backend
npm run setup    # Installs dependencies and initializes database
cp .env.example .env
npm run dev
```

Backend runs on: `http://localhost:3001`

### 2. Frontend Setup

```bash
cd frontend
npm install
npm run dev
```

Frontend runs on: `http://localhost:5173`

### Test Credentials

| Role     | Email              | Password    |
|----------|-------------------|-------------|
| Manager  | manager@unolo.com | password123 |
| Employee | rahul@unolo.com   | password123 |
| Employee | priya@unolo.com   | password123 |

## Project Structure

```
├── backend/
│   ├── utils/           # Utility functions (e.g., distance calculation)
│   ├── config/          # Database configuration
│   ├── middleware/      # Auth middleware
│   ├── routes/          # API routes
│   ├── scripts/         # Database init scripts
│   └── server.js        # Express app entry
├── frontend/
│   ├── src/
│   │   ├── components/  # Reusable components
│   │   ├── pages/       # Page components
│   │   └── utils/       # API helpers
│   └── index.html
└── database/            # SQL schemas (reference only)
```

## API Endpoints

### Authentication
- `POST /api/auth/login` - Login
- `GET /api/auth/me` - Get current user

### Check-ins
- `GET /api/checkin/clients` - Get assigned clients
- `POST /api/checkin` - Create check-in (accepts latitude & longitude, returns distance_from_client)
- `PUT /api/checkin/checkout` - Checkout
- `GET /api/checkin/history` - Get check-in history
- `GET /api/checkin/active` - Get active check-in

### Dashboard
- `GET /api/dashboard/stats` - Manager stats
- `GET /api/dashboard/employee` - Employee stats

## Notes

- The database uses SQLite - no external database setup required
- Run `npm run init-db` to reset the database to initial state

## Features

- Employee check-in and check-out at client locations
- Manager and employee dashboards
- Check-in history with filters
- **Real-time distance calculation between employee and client location**
  - Distance is calculated using geographic coordinates (Haversine formula)
  - Stored in database for every check-in
  - Warning is shown if employee is more than 500 meters away from client


---


## Architecture Decisions

- SQLite is used for simplicity and zero-config setup.
- Distance between employee and client is calculated on the backend using the Haversine formula to ensure consistency and prevent client-side manipulation.
- The calculated distance is stored in the database with each check-in to avoid recomputation and to support reporting/history views.
- Frontend only displays the distance returned by the API.
