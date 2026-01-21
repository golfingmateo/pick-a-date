# Pick a Date - Golf Tournament Scheduler

A web-based event scheduling application that allows users to create events, share availability, and find dates that work for everyone.

## Project Overview

This is a multi-event scheduling system where users can:
- Create events with custom date ranges and exclusions
- Share public links for participants to mark their availability
- View admin dashboards to see all responses and find optimal dates
- Manage multiple events

## Tech Stack

- **Frontend**: Pure HTML/CSS/JavaScript (no frameworks)
- **Backend**: Firebase Realtime Database
- **Hosting**: GitHub Pages
- **Repository**: https://github.com/golfingmateo/golf-date-picker

## File Structure

```
/
├── index.html          # Event creation and management page
├── event.html          # Public event page where participants mark availability
├── dashboard.html      # Admin dashboard for viewing responses
├── admin.html          # Super admin console to view all events
└── docs/
    └── index.html      # (GitHub Pages deployment - copy of index.html)
```

## Core Features

### 1. Event Creation (index.html)
- Event title and optional info
- Background color picker (8 presets + custom)
- Date range picker (visual calendar modal)
- Exclude specific days of week (e.g., weekends)
- Exclude specific dates within range
- Maximum response limit
- Generates two links:
  - Public link: Share with participants
  - Admin link: Dashboard access

### 2. Event Response Page (event.html)
- Reads event settings from Firebase
- Displays dates in weekly calendar grid (Mon-Sun)
- Three-state toggle: Neutral → Available → Unavailable
- Shows dots indicating how many people marked each date
- "Everyone's Available" summary section
- Real-time updates when others respond
- Stores user name in localStorage per event

### 3. Dashboard (dashboard.html)
- Admin-only access (requires admin key in URL)
- Statistics: total responses, perfect dates, etc.
- Visual calendar showing availability
- Individual response management
- Delete user responses
- Delete entire event

### 4. Admin Console (admin.html)
- Password-protected (default: "admin123")
- View ALL events across all users
- Search and filter events
- Access any event's dashboard
- Delete any event

## Firebase Structure

```
/events/
  /{eventId}/
    - id: string
    - title: string
    - info: string
    - color: string (hex)
    - dateBegin: string (YYYY-MM-DD)
    - dateEnd: string (YYYY-MM-DD)
    - maxResponses: number | null
    - excludedWeekdays: array (e.g., [0, 6] for Sun/Sat)
    - excludedDates: array (e.g., ["2026-01-25"])
    - adminKey: string
    - created: string (ISO timestamp)
    - responseCount: number
    /responses/
      /{sanitizedName}/
        - name: string
        - availability: object { "YYYY-MM-DD": true/false/null }
        - timestamp: string
```

## Important Implementation Details

### Firebase Array Handling
Firebase converts JavaScript arrays to objects when they have numeric indices. The code handles this by normalizing data on load:

```javascript
// In event.html and dashboard.html
if (eventData.excludedWeekdays && typeof eventData.excludedWeekdays === 'object' && !Array.isArray(eventData.excludedWeekdays)) {
    eventData.excludedWeekdays = Object.values(eventData.excludedWeekdays);
}
```

### URL Building
All files use a `getBaseUrl()` helper function to properly construct URLs that work both locally and on GitHub Pages:

```javascript
function getBaseUrl() {
    const path = window.location.pathname;
    const origin = window.location.origin;
    let dir = path.substring(0, path.lastIndexOf('/') + 1);
    if (!dir || dir === '') {
        dir = '/';
    }
    return origin + dir;
}
```

### Calendar Week Display (event.html)
- Weeks start on Monday (not Sunday)
- If both Sat/Sun are excluded, they're hidden entirely
- Otherwise, excluded dates show as grayed out and unclickable
- Each row = one calendar week with proper padding

### Mobile Responsiveness
- Desktop: 7 columns (full week)
- Mobile: 7 columns (smaller, more compact)
- Status text hidden on mobile
- Dots are smaller on mobile

## Firebase Configuration

Located in each HTML file:
```javascript
const firebaseConfig = {
    apiKey: "AIzaSyASUTmWxGbTSah_Pkxa_VIjJvppxc4dojE",
    authDomain: "pick-a-date-d0fc8.firebaseapp.com",
    databaseURL: "https://pick-a-date-d0fc8-default-rtdb.firebaseio.com",
    projectId: "pick-a-date-d0fc8",
    // ...
};
```

## Firebase Rules

Current rules (in Firebase Console):
```json
{
  "rules": {
    "events": {
      ".read": true,
      ".write": true,
      "$eventId": {
        ".validate": "newData.hasChildren(['id', 'title', 'color', 'dateBegin', 'dateEnd', 'adminKey', 'created'])",
        "responses": {
          ".read": true,
          ".write": true
        }
      }
    }
  }
}
```

## Deployment

1. Push changes to GitHub
2. GitHub Pages serves from `/docs` folder
3. Copy `index.html` to `docs/index.html` when deploying
4. Access at: https://golfingmateo.github.io/pick-a-date/

## Common Tasks

### Adding a New Feature
1. Determine which file(s) need changes
2. Update UI in HTML
3. Add JavaScript functionality
4. Test locally
5. Update Firebase rules if needed
6. Deploy to GitHub Pages

### Debugging
- Use browser console (F12)
- Check Firebase Console for data structure
- Verify URLs are correct (check for missing slashes)
- Ensure Firebase rules allow read/write

### Changing Color Palette
Edit the color options in `index.html` inside the Color Picker Modal:
```html
<div class="color-option" data-color="#667eea" data-label="Purple" ...></div>
```

### Updating Admin Password
In `admin.html`, change:
```javascript
const ADMIN_PASSWORD = "admin123";
```

## Known Issues & Quirks

1. **Firebase Array Conversion**: Arrays with numeric indices become objects - handled by normalization code
2. **localStorage Scope**: User names stored per-event using key `golfUserName_{eventId}`
3. **Past Dates**: Automatically disabled and shown as "Past"
4. **Real-time Updates**: Uses Firebase's `.on('value')` listener

## UI/UX Decisions

- **Purple gradient default**: Professional, friendly
- **Three-state toggle**: Allows "maybe" (neutral) state
- **Dots for responses**: Quick visual of participation
- **Modal pickers**: Cleaner than inline calendars
- **Extra Settings section**: Groups less-frequently-changed options

## Future Enhancement Ideas

- Email notifications when event is created
- Export responses to CSV
- Recurring events
- Time preferences (not just dates)
- Voting on best dates
- Integration with Google Calendar
- User accounts (optional)
- Event templates

## Development Notes

- Keep all files as standalone HTML (no build process)
- Maintain Firebase SDK version 10.7.1 for compatibility
- Test on mobile devices regularly
- Consider accessibility (keyboard navigation, screen readers)
- All modals should have close buttons and ESC key support

## Testing Checklist

- [ ] Create new event
- [ ] Public link opens event correctly
- [ ] Mark availability on dates
- [ ] Save name and verify it persists
- [ ] Dashboard shows responses correctly
- [ ] Delete user from dashboard
- [ ] Delete entire event
- [ ] Admin console shows all events
- [ ] Mobile layout works
- [ ] Excluded dates show properly
- [ ] "Everyone's Available" updates correctly

## Support & Maintenance

- **Firebase free tier**: 100 simultaneous connections, 1GB stored, 10GB/month downloaded
- **GitHub Pages**: Free for public repositories
- **Browser compatibility**: Modern browsers (Chrome, Firefox, Safari, Edge)

## Contact

Project maintainer: golfingmateo (GitHub)
