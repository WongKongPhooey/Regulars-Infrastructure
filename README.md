# Regulars 📺

A TV Guide-style app for tracking your favourite Twitch and YouTube streamers' schedules.

**Stack**: React Native + Expo (mobile + web) · Node.js + Express (backend)  
**Views**: Weekly TV Guide grid **and** vertical daily timeline, with a toggle  
**Data**: Mock schedules out of the box, with real Twitch/YouTube API hooks ready to fill in

---

## Project Structure

```
regulars/
├── backend/
│   └── src/
│       ├── index.js                    # Server entry point, middleware, routes
│       ├── data/store.js               # In-memory store (swap for DB later)
│       ├── routes/                     # URL definitions → controller functions
│       ├── controllers/                # Business logic per route
│       └── services/
│           └── mockScheduleService.js  # Mock data + real API stubs
│
└── frontend/
    ├── App.jsx                         # Root: SafeArea, Navigation, Tabs
    └── src/
        ├── screens/
        │   ├── GuideScreen.jsx         # Main screen: header, toggle, view swap
        │   └── StreamersScreen.jsx     # Manage followed streamers
        ├── components/
        │   ├── ViewToggle.jsx          # 📺 Grid / 📋 Timeline segmented control
        │   ├── WeekGridView.jsx        # Horizontal 7-day column grid
        │   ├── DayTimelineView.jsx     # Vertical agenda with collapsible days
        │   ├── DayColumn.jsx           # One column in the grid
        │   ├── StreamSlot.jsx          # A single stream card (grid view)
        │   └── AddStreamerModal.jsx    # Bottom-sheet form to follow a streamer
        ├── hooks/
        │   ├── useSchedule.js          # Fetches + owns schedule state
        │   └── useStreamers.js         # Fetches + owns streamers state
        ├── services/api.js             # All axios HTTP calls, platform-aware URL
        └── utils/
            ├── theme.js                # Colours, spacing, font sizes
            └── dateHelpers.js          # Formatting & date utilities
```

---

## Quick Start

### 1 — Backend

```bash
cd backend
npm install
cp .env.example .env    # Optional: add Twitch/YouTube API keys
npm run dev             # Starts on http://localhost:3001
```

Verify: `GET http://localhost:3001/health` → `{ "status": "ok" }`

### 2 — Frontend

```bash
cd frontend
npm install
npx expo start
```

| Key | Platform |
|-----|----------|
| `w` | Web browser (http://localhost:8081) |
| `i` | iOS Simulator |
| `a` | Android Emulator |

> **Physical device?** Edit `getBaseURL()` in `src/services/api.js` and set
> your machine's LAN IP: `http://192.168.x.x:3001/api`

---

## The Two Views

Both are available from the **📺 Grid / 📋 Timeline** toggle in the Guide header.

### 📺 Grid View
- 7 day columns scrolling horizontally — classic TV Guide layout
- Today's column highlighted in purple
- Platform colour stripe on each card
- Green LIVE badge on currently-live streams

### 📋 Timeline View
- Vertical agenda grouped by day, with collapsible day sections
- Time + duration on the left, content card on the right
- Platform pill + LIVE badge on each entry
- Empty days start collapsed to reduce noise

---

## API Reference

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/streamers` | List followed streamers |
| `POST` | `/api/streamers` | Follow a new streamer |
| `DELETE` | `/api/streamers/:id` | Unfollow |
| `GET` | `/api/schedule/week` | Weekly schedule grouped by date |
| `GET` | `/api/schedule` | All slots — `?from=&to=&platform=` |
| `POST` | `/api/schedule/refresh/:id` | Re-fetch a streamer's schedule |
| `GET` | `/api/platforms` | Supported platform metadata |

---

## Enabling Real API Data

### Twitch
1. Register an app at https://dev.twitch.tv/console
2. Add to `.env`: `TWITCH_CLIENT_ID` and `TWITCH_CLIENT_SECRET`
3. Implement `fetchTwitchSchedule()` in `mockScheduleService.js`
   - Auth: `POST https://id.twitch.tv/oauth2/token`
   - Schedule: `GET https://api.twitch.tv/helix/schedule?broadcaster_id={id}`
   - Docs: https://dev.twitch.tv/docs/api/reference/#get-channel-stream-schedule

### YouTube
1. Enable YouTube Data API v3 at https://console.cloud.google.com
2. Add to `.env`: `YOUTUBE_API_KEY`
3. Implement `fetchYouTubeSchedule()` in `mockScheduleService.js`
   - Upcoming streams: `GET https://www.googleapis.com/youtube/v3/search?part=snippet&channelId={id}&eventType=upcoming&type=video&key={key}`
   - Docs: https://developers.google.com/youtube/v3/live/docs/liveBroadcasts/list

---

## Adding a New Platform (e.g. Kick)

1. `backend/src/data/store.js` → add entry to `PLATFORMS`
2. `backend/src/services/mockScheduleService.js` → add `fetchKickSchedule()` stub
3. `frontend/src/components/AddStreamerModal.jsx` → add to `PLATFORMS` array
4. `frontend/src/utils/theme.js` → add brand colour to `COLORS`

---

## Key Learning Points

| Concept | File |
|---------|------|
| Express middleware chain | `backend/src/index.js` |
| Route → Controller pattern | `routes/streamers.js` + `controllers/streamersController.js` |
| Input validation (express-validator) | `routes/streamers.js` |
| Custom React hooks | `hooks/useSchedule.js`, `hooks/useStreamers.js` |
| `useState` / `useEffect` / `useCallback` | Both hook files |
| Optimistic UI update | `hooks/useStreamers.js` — `remove()` |
| Axios interceptors + platform-aware URL | `services/api.js` |
| React Navigation bottom tabs | `App.jsx` |
| Conditional rendering (view toggle) | `screens/GuideScreen.jsx` |
| `FlatList` vs `ScrollView` | `StreamersScreen.jsx` vs `WeekGridView.jsx` |
| Collapsible state | `components/DayTimelineView.jsx` — `DaySection` |
| Modal + KeyboardAvoidingView | `components/AddStreamerModal.jsx` |
