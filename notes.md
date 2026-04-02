# Map/ Codebase — Detailed Engineering Notes

## 1) What this folder is

`Map/` is a **React + TypeScript + Vite** frontend application for the Safe-Roads project.

Its main goal is:

- visualize potholes on an interactive map,
- compute multiple route options,
- rank routes by safety/cost,
- provide turn-by-turn UI + voice guidance,
- support nearby places search,
- use live pothole data from Supabase.

This app is map-first and mobile-friendly, with location tracking and orientation-aware map rotation.

---

## 2) Top-level structure (what each file/folder does)

- `index.html`  
  App shell + SEO/social metadata + root mount node.

- `metadata.json`  
  App metadata (name/description) and geolocation frame permission.

- `package.json`  
  Scripts + dependencies (React, Leaflet, Geo UI stack, Supabase, Tailwind).

- `.env.example`  
  Sample environment values and comments for Supabase/Geoapify/Gemini runtime wiring.

- `.gitignore`  
  Ignores generated artifacts (`node_modules`, `dist`, logs, `.env*`, etc.).

- `README.md`  
  Currently minimal (only title), no setup/run documentation inside this file yet.

- `tsconfig.json`  
  TypeScript compiler settings for modern browser ESM + React JSX.

- `vite.config.ts`  
  Vite config with React + Tailwind plugins, aliasing, env define.

- `src/`  
  Main source code.
  - `main.tsx`: React entry point.
  - `App.tsx`: orchestrator/root state machine.
  - `types.ts`: shared `Route` interface.
  - `index.css`: Tailwind + base layout + map app viewport fixes.
  - `components/MapView.tsx`: Leaflet rendering and route scoring.
  - `components/NavigationPanel.tsx`: route input/control panel UI.
  - `components/PlacesSearch.tsx`: nearby places discovery UI.
  - `lib/supabase.ts`: Supabase client + `Pothole` type.
  - `lib/utils.ts`: Tailwind class merge helper.

- `assets/image.png`  
  Static asset in repo (not central in core logic shown here).

- `dist/`, `node_modules/`  
  Build/artifact/dependency folders (generated, not source-of-truth logic).

---

## 3) Tech stack and why it is used

### Frontend

- **React 19** for stateful UI composition.
- **TypeScript** for model safety (`Route`, `Pothole`, prop contracts).
- **Vite** for fast dev/build cycle.
- **Tailwind CSS v4** for utility-first styling.
- **motion** (`motion/react`) for animated panels/modals.
- **lucide-react** for iconography.

### Mapping + geospatial

- **Leaflet + react-leaflet** for map rendering, markers, polylines, layers.
- **Geoapify APIs** for:
  - tiles,
  - geocoding/autocomplete/reverse geocoding,
  - routing alternatives,
  - route optimization (waypoint order),
  - isolines,
  - places search and details,
  - icon endpoints,
  - IP fallback geolocation.

### Data backend

- **Supabase** as pothole datastore + realtime change subscription.

### Why this stack fits this problem

- Leaflet is performant and practical for route/polyline overlays.
- Geoapify gives one vendor for many geo primitives (simpler integration).
- Supabase realtime keeps pothole overlays live without manual refresh.
- React state model is sufficient (no Redux needed for current scope).

---

## 4) Application lifecycle flow

1. `main.tsx` renders `<App />` into `#root`.
2. `App.tsx` initializes app-level state.
3. `fetchPotholes()` runs:
   - tries Supabase table `potholes`.
   - if Supabase env is missing/error -> uses mock potholes fallback.
4. Realtime channel subscribes to `potholes` table changes.
5. User can set source/destination (or current location).
6. `App` sends `waypoints + mode` to `MapView`.
7. `MapView` calls Geoapify routing with alternatives.
8. `MapView` computes custom safety cost, sorts routes, returns to `App`.
9. `NavigationPanel` + map overlays display alternatives and details.
10. During navigation/tracking:
    - map auto-pan and rotate based on heading,
    - deviation detection can trigger reroute,
    - pothole voice alerts fire near hazards.

---

## 5) Detailed file-by-file implementation notes

## `src/main.tsx`

- Minimal bootstrap.
- Wraps app in `StrictMode`.
- Imports global styles.

Why: keep entrypoint pure and tiny; all behavior pushed to `App`.

---

## `src/App.tsx` (core orchestrator)

This is the central coordinator for:

- pothole loading,
- user geolocation/tracking,
- selected route state,
- navigation steps,
- voice prompts,
- popup/modals,
- bridge between panel and map components.

### Key state groups

1. **Data state**

- `potholes`
- `allRoutes`
- `routeInfo`
- `isolineData`

2. **User position/navigation state**

- `userLocation`, `mapCenter`
- `waypoints`
- `travelMode`
- `isTracking`, `isNavigating`
- `userHeading`

3. **Instruction state**

- `navigationInstructions`
- `navigationDistances`
- `currentInstructionIndex`

4. **UX state**

- `isLoading`, `isSearching`, `error`
- toggles: `avoidPotholes`, `showPotholesOnMap`
- modals: `showAbout`, `selectedPothole`

### Important refs and why refs are used

- `watchIdRef` stores geolocation watch id for cleanup.
- `orientationListenerRef` stores orientation callback for safe remove.
- `lastRerouteLocation` debounces movement-triggered reroutes.
- `lastPotholeAlertAtRef`, `lastAlertedPotholeIdRef` debounce voice warnings.

Refs avoid unnecessary re-renders and preserve mutable runtime handles.

### Geolocation implementation details

- Uses `navigator.geolocation.getCurrentPosition` first (high accuracy).
- Starts `watchPosition` for continuous updates.
- Smooths position update with alpha blending to reduce jitter.
- Fallback to Geoapify IP geolocation if precise location fails.
- Handles permission and timeout errors with user-readable messages.
- Checks `window.isSecureContext` because mobile geolocation requires HTTPS/localhost.

### Orientation/heading handling

- Requests iOS orientation permission when needed.
- Reads heading from `webkitCompassHeading` when available.
- Else derives from `alpha` sensor value.

### Route + panel interactions

- `handleSearch` composes waypoint chain: `[source, ...stops, destination]`.
- `handleRoutesCalculated` accepts ranked route options from `MapView`.
- `handleSelectRoute` switches active route and resets instruction index.
- `handleClearRoute` resets route + nav state.

### Advanced helper features implemented in App

- `fetchIsoline(minutes)` -> reachable area polygon.
- `optimizeWaypoints(...)` -> Geoapify route planner async polling loop.
- `findPostcode(lat,lon)` -> reverse geocode postcode lookup.
- text-to-speech helper for instructions and hazard warnings.

### Navigation behavior

- start/next instruction controls.
- speaks only turn instructions (`left/right`) for signal clarity.
- announces destination reached at end.

### Safety automation behavior

- **Nearest pothole warning** if within threshold (35m), with cooldown.
- **Off-route rerouting** when deviating >100m from active route polyline (debounced).

### UI assembled by App

- `NavigationPanel`
- `MapView`
- `PlacesSearch`
- top error toast
- bottom navigation instruction card
- start navigation floating CTA
- project info modal
- pothole image gallery modal

Why App is this large:

- It acts like a lightweight controller layer (state + orchestration + policy logic).
- Child components remain mostly focused on rendering/specific APIs.

---

## `src/components/MapView.tsx` (map rendering + route scoring engine)

### Core responsibilities

1. Leaflet map composition and base layers.
2. Route fetch from Geoapify routing API.
3. Pothole-route proximity matching.
4. Safety/cost scoring + route ranking.
5. Marker/polyline rendering.
6. Local map camera control (auto-pan, auto-zoom, rotation).

### Marker/Icon strategy

- Fixes Leaflet default marker asset issue by overriding icon URLs.
- Builds custom pothole marker with severity-based size and confidence-based opacity.
- Custom user marker rotates by heading.
- Source/destination icons loaded from Geoapify icon endpoint.

### `MapController` internal helper

Encapsulates imperative map effects:

- auto-center on user during tracking/navigation,
- fit bounds when route exists,
- smooth map rotation based on heading,
- reset transform when exiting navigation/tracking,
- invalidate map size on mount/resize for mobile correctness.

### Route fetching and enrichment

On waypoint/mode/toggle changes:

- calls Geoapify routing with alternatives.
- parses polyline coordinates.
- samples route points for pothole proximity checks.
- computes:
  - pothole count,
  - pothole severity score,
  - step instructions and distances,
  - signals count (from instruction text scan).

### Cost function (why route ranking works)

Each candidate gets `totalCost` from weighted terms:

- distance,
- adjusted time (base + pothole penalty),
- pothole severity,
- signals count.

When `avoidPotholes=true`, pothole weight is very high (`w3 = 15.0`) so safer routes win strongly.

This is an intentional product decision: prioritize safety over shortest/fastest in safe mode.

### Pothole filtering logic

- If navigating: show potholes very close to active route (20m).
- If route selected (not navigating): relaxed threshold (50m).
- Else default set from detected potholes.

### Map overlays/features

- multiple base styles (standard, dark, satellite-like, terrain, toner).
- isoline polygon with popup and clear action.
- route polylines with selected highlighting.
- source/destination markers.
- pothole markers + popup details + image preview trigger.
- alternate route quick panel overlay.
- floating locate-me button.

### Additional voice safety

Includes another proximity speech alert path in map layer logic for nearby potholes (<100m) with cooldown.

---

## `src/components/NavigationPanel.tsx` (search/control cockpit)

### Core responsibilities

- collect source/destination/waypoints,
- fetch location suggestions (autocomplete),
- manage selected coordinates,
- submit route search,
- mode toggles (car/bike/walk),
- safety and map toggles,
- route alternatives and route metrics display,
- trigger isolate/optimize/postcode actions.

### Search UX model

- Separate text and coordinate states for source/destination.
- Suggestion API debounced (150ms).
- Request cancellation via `AbortController` to avoid race conditions.
- If direct coords unavailable, fallback geocoding resolves typed text.

### Waypoint support

- dynamic add/remove waypoint fields.
- per-waypoint autocomplete lists.
- optimization button appears when >1 waypoint.

### Route options display

- lists alternatives from `allRoutes`.
- computes simple visible `safetyScore` (100 - pothole/signals penalties).
- shows distance/time/pothole/signals summary.
- highlights selected route.

### Additional controls

- “Use current location” fills source via reverse geocoding.
- postcode finder from current location.
- toggles:
  - Safe Mode (avoid potholes),
  - show potholes,
  - reachable area by 5/10/15 minutes.

### Why this design

- Keeps all trip setup in one collapsible floating card.
- Supports both quick users (autocomplete click) and manual typers (geocode fallback).
- Uses compact visual hierarchy for mobile + desktop.

---

## `src/components/PlacesSearch.tsx` (POI discovery)

### Core responsibilities

- find nearby places by category around current map center,
- optional condition filters,
- show list and detailed place card,
- allow routeing to chosen place.

### Implemented behavior

- categories include restaurants, fuel, supermarkets, hotels, cinema, attractions.
- condition chips (wheelchair/wifi/free entry).
- fetches POI list from Geoapify places endpoint.
- local cache keyed by category+conditions+approx center to reduce repeat calls.
- details panel uses Geoapify place-details endpoint.
- “Go to this Place” emits `onPlaceSelect(lat, lon, name)` to `App`.

### Why this exists

- turns map app from “only route planner” into “destination discovery + navigation” tool.

---

## `src/lib/supabase.ts`

- Builds Supabase client from `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY`.
- Warns in console if missing credentials.
- Uses placeholder URL/key to keep runtime from hard-crashing in development.
- Exports strict `Pothole` type used across app.

`Pothole` fields include confidence, severity (1..5), status (`detected|fixed`), vote counts, etc.

Why: centralize backend client and shared type.

---

## `src/types.ts`

Defines `Route` model used between `MapView`, `App`, and `NavigationPanel`.
Contains geometry, timing, hazard counts, instructions, and scoring metadata.

Why: shared contract prevents prop/state mismatch and makes route scoring explicit.

---

## `src/lib/utils.ts`

`cn(...inputs)` combines `clsx` + `tailwind-merge`.

Why:

- `clsx` handles conditional class lists,
- `tailwind-merge` resolves conflicting Tailwind utilities cleanly.

---

## `src/index.css`

- imports Tailwind and Leaflet styles.
- defines `--app-height` plus viewport-safe height strategy (`dvh`, `svh`).
- sets `overflow: hidden` to keep app map-shell immersive.
- custom scrollbar styles for side panels.

Why: avoid mobile viewport jitter and preserve map-first full-screen UX.

---

## `vite.config.ts`

- React + Tailwind Vite plugins.
- Loads env by mode.
- Exposes `process.env.GEMINI_API_KEY` define (currently not central in shown UI).
- alias `@ -> project root`.
- conditional HMR based on `DISABLE_HMR` environment flag.

Why: cleaner imports, controlled dev behavior in editing/runtime environments.

---

## `tsconfig.json`

- modern target (`ES2022`) + `moduleResolution: bundler`.
- JSX transform for React.
- path alias support.
- no emit (type-check only during dev/lint).

Why: aligned with Vite ESM pipeline and fast TS feedback loops.

---

## `index.html` and `metadata.json`

`index.html` adds:

- SEO metadata,
- OpenGraph/Twitter cards,
- project branding,
- root mount node.

`metadata.json` adds:

- app title/description for shell context,
- `requestFramePermissions: ["geolocation"]` for embedded/geofeature environments.

---

## 6) Feature inventory (implemented)

1. Live pothole ingestion from Supabase (+ realtime updates).
2. Mock pothole fallback when backend env is unavailable.
3. Route alternatives via Geoapify.
4. Safety-aware route ranking with pothole-heavy weighted cost.
5. Turn-by-turn instruction UI.
6. Voice prompts for turns and hazard alerts.
7. Live GPS tracking + heading-based map rotation.
8. Deviation detection + rerouting.
9. Isoline reachable-area visualization.
10. Multi-waypoint trip planning + optimization.
11. Nearby place discovery + detail drilldown + navigate to selected place.
12. Pothole popup details with severity/confidence and full-image modal.
13. Alternate route comparison cards in panel and map overlay.
14. Map style switching.
15. Project info/about modal.

---

## 7) Environment variables expected

Primary required runtime keys:

- `VITE_GEOAPIFY_API_KEY`
- `VITE_SUPABASE_URL`
- `VITE_SUPABASE_ANON_KEY`

Behavior without keys:

- missing Geoapify key -> map/routing features blocked with UI message.
- missing Supabase keys -> warning + placeholder client + mock pothole fallback data.

---

## 8) Performance and reliability patterns in code

Implemented good practices:

- `AbortController` for canceling stale API requests.
- debounce for autocomplete.
- coarse bounding-box prechecks before expensive distance calculations.
- route sampling to reduce proximity-check work.
- cooldown timers for speech/rerouting alerts.
- cleanup of geolocation watchers and orientation listeners.

---

## 9) Why the architecture is split this way

- `App.tsx`: stateful orchestration and business rules.
- `MapView.tsx`: map + route computation/rendering.
- `NavigationPanel.tsx`: form workflows and trip controls.
- `PlacesSearch.tsx`: destination discovery domain.

This is a practical component boundary by domain responsibility. It keeps UI modules focused while still allowing central policy decisions in App.

---

## 10) Notable design tradeoffs / caveats

1. `App.tsx` and `MapView.tsx` are both large; maintainability can improve by extracting hooks/services.
2. Some logic is duplicated conceptually (pothole proximity voice alerts in both layers).
3. Several dependencies in `package.json` are not visibly used by the current source path.
4. Security: API keys are client-exposed (`VITE_*`), which is normal for map providers but should be restricted by domain quota/rules.
5. Distance thresholds use simple heuristics; could be improved with line-segment projection for precise off-route distance.

---

## 11) Build / run behavior

Scripts from `package.json`:

- `npm run dev` -> `vite --port=3000 --host=0.0.0.0`
- `npm run build` -> production build
- `npm run preview` -> preview production bundle
- `npm run lint` -> TS typecheck (`tsc --noEmit`)

---

## 12) Practical mental model of the app

Think of the system as 4 cooperating engines:

1. **Data engine (Supabase)** → gives pothole records.
2. **Geo engine (Geoapify)** → gives routes/places/geocoding/isolines.
3. **Decision engine (MapView cost function + App toggles)** → chooses safest route profile.
4. **Guidance engine (App nav state + speech + map camera controls)** → guides the user while moving.

That combination is exactly why this app is more than a normal map: it is a safety-oriented navigation layer on top of geospatial services.

---

## 13) Quick summary

The `Map/` folder implements a full client-side smart navigation experience focused on pothole-aware safety. It combines:

- real-time hazard ingestion,
- geospatial routing and optimization,
- intelligent route ranking,
- navigation assistance,
- and nearby-place discovery,

inside a modern React + Leaflet architecture tuned for mobile, map-first UX.
