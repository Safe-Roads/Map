<div align="center">

# Safe-Roads Map
### ML-Inspired Road Infrastructure Monitoring • Smart Navigation • Pothole Awareness

<a href="https://map-tau-taupe.vercel.app/" target="_blank" rel="noopener noreferrer">
  <img src="./assests/image.png" alt="Safe-Roads Map — Website Screenshot" width="920" />
</a>

<p>
  <a href="https://map-tau-taupe.vercel.app/" target="_blank" rel="noopener noreferrer"><strong>Live Demo</strong></a>
  &nbsp;•&nbsp;
  <a href="https://github.com/Safe-Roads/Map"><strong>Repository</strong></a>
  &nbsp;•&nbsp;
  <a href="https://github.com/Safe-Roads"><strong>Safe-Roads Org</strong></a>
</p>

<p>
  <img alt="Vite" src="https://img.shields.io/badge/Vite-6.x-646CFF?logo=vite&logoColor=white" />
  <img alt="React" src="https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=white" />
  <img alt="TypeScript" src="https://img.shields.io/badge/TypeScript-5.8-3178C6?logo=typescript&logoColor=white" />
  <img alt="TailwindCSS" src="https://img.shields.io/badge/TailwindCSS-4.x-06B6D4?logo=tailwindcss&logoColor=white" />
  <img alt="Leaflet" src="https://img.shields.io/badge/Leaflet-Maps-199900?logo=leaflet&logoColor=white" />
  <img alt="Supabase" src="https://img.shields.io/badge/Supabase-Database%20%26%20Realtime-3FCF8E?logo=supabase&logoColor=white" />
  <img alt="Geoapify" src="https://img.shields.io/badge/Geoapify-Geospatial%20APIs-2F80ED" />
  <img alt="License" src="https://img.shields.io/badge/License-MIT-green.svg" />
</p>

</div>

---

## Overview
**Safe-Roads Map** is the mapping & navigation module of the **Safe-Roads** initiative—built to support safer travel and better infrastructure decisions through pothole awareness, real-time mapping, and intelligent routing.

**Live Website:** https://map-tau-taupe.vercel.app/

---

## What this app does
- Visualizes pothole hazards on an interactive map
- Helps users plan routes and navigate with step instructions
- Provides **proximity voice alerts** for detected potholes
- Supports location tracking and mobile-friendly navigation behavior
- Integrates with **Supabase** for pothole data + realtime updates (optional)

---

## Features
- Interactive map UI (Leaflet + React)
- Pothole layer toggle + pothole detail preview
- Route planning with multiple route options
- Avoid potholes routing option
- Live location tracking (GPS + fallback to IP geolocation)
- Navigation mode with step instructions
- Voice alerts when hazards are nearby
- Isoline (reachability) overlay (time-based area)
- Realtime pothole updates (Supabase channel)

---

## Tech Stack
- **Frontend:** React 19, TypeScript, Vite
- **Styling/UI:** Tailwind CSS, Motion, Lucide
- **Maps:** Leaflet, react-leaflet
- **Database (optional):** Supabase (Postgres + Realtime)
- **Geospatial APIs:** Geoapify (geocoding, routing, isolines, route planning)

---

## Getting Started (Local)
### Requirements
- **Node.js 18+** (recommended: Node 20+)
- npm (or pnpm/yarn)

### Install
```bash
npm install
```

### Environment variables
```bash
cp .env.example .env
```

**Required**
- `VITE_GEOAPIFY_API_KEY`

**Optional**
- `VITE_SUPABASE_URL`
- `VITE_SUPABASE_ANON_KEY`

### Run
```bash
npm run dev
```

Open: http://localhost:3000

---

## License
This project is licensed under the **MIT License** — see the `LICENSE` file for details.
