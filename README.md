# ⬡ PATHFINDER — Setup & Deployment Guide

## Project Structure

```
pathfinder/
├── index.html          ← Landing page (marketing)
├── auth.html           ← Sign in / Sign up
├── dashboard.html      ← User dashboard
├── simulator.html      ← Core simulator (main app)
├── vercel.json         ← Vercel routing config
├── supabase-schema.sql ← Database schema
└── README.md
```

---

## Step 1 — Supabase Setup (Free Tier)

1. Go to https://supabase.com and create a new project
2. In the **SQL Editor**, run the contents of `supabase-schema.sql`
3. Go to **Authentication > Providers**:
   - Enable **Email** provider
   - Enable **GitHub** OAuth (optional but recommended)
4. Copy your credentials from **Project Settings > API**:
   - `Project URL` → `SUPABASE_URL`
   - `anon public key` → `SUPABASE_ANON_KEY`

---

## Step 2 — Configure Credentials

Replace in ALL four HTML files (`auth.html`, `dashboard.html`, `simulator.html`):

```javascript
const SUPABASE_URL = 'https://xxxx.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI...';
```

> **Tip**: For production, use Vercel Environment Variables and a thin backend proxy
> rather than hardcoding keys. Free tier anon keys are safe for client-side use with RLS.

---

## Step 3 — Anthropic API Key

The simulator calls the Claude API directly from the browser for algorithm analysis.

**For production** (recommended):
1. Create a Vercel Edge Function at `/api/analyze`
2. Store `ANTHROPIC_API_KEY` as a Vercel Environment Variable
3. Update `simulator.html` to call `/api/analyze` instead of the Anthropic API directly

**For quick testing**:
Create `/api/analyze.js` in your project:
```javascript
export const config = { runtime: 'edge' };
export default async function handler(req) {
  const body = await req.json();
  const res = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': process.env.ANTHROPIC_API_KEY,
      'anthropic-version': '2023-06-01'
    },
    body: JSON.stringify(body)
  });
  const data = await res.json();
  return new Response(JSON.stringify(data), {
    headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' }
  });
}
```

Then in `simulator.html`, change `ANTHROPIC_PROXY` to `'/api/analyze'`.

---

## Step 4 — Deploy to Vercel (Free Tier)

### Option A: Vercel CLI
```bash
npm i -g vercel
cd pathfinder
vercel --prod
```

### Option B: GitHub + Vercel Dashboard
1. Push to a GitHub repository
2. Go to https://vercel.com/new
3. Import the repository
4. Set Environment Variables:
   - `ANTHROPIC_API_KEY` = your Anthropic key
5. Deploy

---

## Features Overview

### 🧠 AI Algorithm Analysis
- Accepts any Python drone control code
- Claude AI extracts: flight parameters, PID gains, waypoints, behavior type
- Generates risk assessment, strengths, and warnings

### 🌐 3D Simulation (Three.js)
- Real-time WebGL 3D drone flight visualization
- Physics-aware camera tracking
- Rotor animation, lighting effects
- 4 environments: Open Field, Urban, Forest, Indoor

### 📡 Live Telemetry
- Altitude, speed, heading, pitch, roll, yaw rate
- Battery drain simulation
- GPS fix status
- PID error monitoring
- Live mini charts

### ▶ Playback Controls
- Play/pause/reset
- Speed: 0.5× / 1× / 2× / 4×
- Timeline scrubbing

### 📋 Template Library
- Waypoint Navigation (Beginner)
- PID Controller (Intermediate)
- Obstacle Avoidance (Intermediate)
- Swarm Formation (Advanced)
- SLAM Navigation (Advanced)

---

## Supabase Tables

| Table | Purpose |
|-------|---------|
| `simulations` | Stores all simulation runs per user |
| `profiles` | User profiles, plan tier, role |

---

## Free Tier Limits

| Service | Free Limit |
|---------|-----------|
| Supabase DB | 500MB storage |
| Supabase Auth | Unlimited users |
| Vercel | 100GB bandwidth/month |
| Anthropic API | Pay-per-use |

---

## Customization

- **Colors**: Edit CSS variables in `:root` in any HTML file
- **Add pages**: Create new `.html` files, add routes to `vercel.json`
- **Custom environments**: Extend `buildObstacles()` in `simulator.html`
- **More templates**: Add to the `TEMPLATES` object in `simulator.html`
