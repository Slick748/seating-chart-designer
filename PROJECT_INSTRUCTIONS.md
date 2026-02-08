# Seating Chart Designer — Project Instructions

## Source Code

**GitHub Repository:** https://github.com/ricobuilds/seating-chart-designer

Clone and work from this repo. All source files are in the repo root. If starting a new Claude chat, point Claude to this repo so it can pull the latest code.

---

## Project Overview

A web-based event seating chart designer built with **Vite + React 18** (no other UI libraries). Converted from a 3,459-line Python/tkinter desktop app into a React web app. Designed for deployment on **Vercel**.

### Tech Stack
- **Vite 5** — build tool
- **React 18** — UI (no additional UI frameworks, no Tailwind, no component libraries)
- **Vanilla CSS** — custom dark theme with CSS variables
- **Canvas 2D API** — all canvas rendering is hand-drawn (no canvas libraries)
- **DM Sans** — Google Font used throughout

---

## File Structure

```
seating-chart-designer/
├── index.html              # Entry HTML
├── package.json            # React 18 + Vite 5
├── vite.config.js          # Vite config
├── vercel.json             # Vercel SPA routing
├── .gitignore
├── public/
│   └── favicon.svg         # Table icon
└── src/
    ├── main.jsx            # React entry point (~10 lines)
    ├── App.jsx             # Main component — all state, UI, event handlers (~1,760 lines)
    ├── models.js           # Data models, utilities, CSV/JSON serialization, COLOR_PALETTE (~210 lines)
    ├── canvasRenderer.js   # Canvas 2D rendering + hit testing + shared helpers (~490 lines)
    └── styles.css          # Dark theme, all styling (~530 lines)
```

### Key Architecture Decisions
- **Single-component architecture**: App.jsx contains the main App component plus sub-components (TablePopup, ListView, EntityCard, SeatRow, Modal) all in one file. This was done intentionally for simplicity.
- **Canvas rendering**: All canvas drawing is in canvasRenderer.js using raw Canvas 2D API. No libraries.
- **Data flow**: All state lives in App(). Sub-components receive props + callbacks. No context, no Redux.
- **Entity coordinates**: Tables and venue elements store `x, y` as **center point**. Chair blocks store `x, y` as **top-left corner**. This distinction matters for snap logic and rendering.

---

## Feature Inventory

### Canvas View
- Drag-and-drop table/element placement with ghost preview
- Grid snapping (Off, 1ft, 2ft, 5ft) — round tables snap center to grid, rectangular entities snap top-left corner
- Zoom (25%–400%) with Ctrl+scroll support
- Configurable room dimensions (ft)
- Click to select, Ctrl+click for multi-select
- Drag to reposition entities
- Double-click to edit entity properties
- Right-click to cancel ghost placement
- Drop attendee on specific seat = assign that seat; drop on table body = next available seat

### Table Popup (Canvas View)
- Overlay panel appears top-left inside canvas-area (position: absolute, z-index: 20) — does NOT shift the grid
- Fits to content height based on seat count (max-height with scroll if needed)
- Editable table name — click title to rename inline, blocked during drag
- Shows all seats numbered with assigned names or "Empty"
- Remove guests with ✕ button (appears on hover)
- Drag names between seats to swap/reorder within the popup
- Drag names from popup onto canvas seats for cross-table moves
- Drag attendees from left panel into popup seats (specific seat) or header (next available)
- Cursor shows `copy` icon when dragging over popup
- Seat 1 marked with gold left border
- Close/reopen behavior — auto-reopens when selecting a new table

### Entity Types
- **Round Table** — circular, seats arranged radially, configurable seat count + diameter
- **Rectangular Table** — seats distributed edge-to-edge along long sides + optional end seats, horizontal/vertical orientation. Uses shared `getRectSeatPositions()` helper for both rendering and hit-testing.
- **Chair Block** — grid of seats (rows × cols), configurable spacing (tight/normal/wide)
- **Venue Elements** — Dance Floor, Stage, Bar, DJ Booth, Buffet (non-seating decorative)

### Selection Toolbar
- Edit, Rotate, Lock/Unlock, Copy, Delete (with confirmation dialog)
- Rotation: round tables rotate seat assignments, rect tables swap dimensions, blocks transpose grid
- Entity name display derives from **live state** (not stale selectedItem snapshot) — updates immediately on rename
- Works from both Canvas and List views (clicking a card in List View selects the entity)

### Left Panel (Attendees)
- Search/filter attendees
- **Sort mode toggle** ("A-Z" button): default = all alphabetical; active (gold) = unassigned first, then seated/disabled
- Click to select (highlights on mouseDown, not mouseUp), drag to seat on canvas or List View cards
- Add (+), Disable/Enable (⊘), Recall/unassign (↩) buttons
- Load CSV button + Export CSV icon button (download arrow)
- Stats: seats filled, attendees count, unassigned count, disabled count

### List View
- Card-based layout showing all tables/blocks with their seat assignments
- **Click card header** to select entity (gold border) AND toggle collapse — enables toolbar actions from List View
- **Inline rename** — double-click card name to edit
- **Color picker** — click color dot (gets gold ring when active), opens overlay with dark backdrop over card body, 18-color palette
- **Delete button** — ✕ on card header (appears on hover), triggers confirmation dialog
- **Drag-drop from attendee panel** — drop on specific seat row = assign that seat; drop on card body = next available
- Click empty seats to assign selected attendee, click filled seats to unassign
- Expand/Collapse all, search/filter
- Cards use `align-items: start` in grid — collapsed cards take natural height, not stretched
- Fully synced with Canvas view data

### Toolbar Dropdowns
- **File** — New Project, Save Project (.json with native file picker), Load Project (.json/.seating)
- **Tools** — Clear All Assignments, Auto-Assign (Alphabetical, Fill in Order, Random)
- **+ Add** — All entity types organized by Seating/Venue categories

### Other
- 50-level undo stack (Ctrl+Z)
- Keyboard shortcuts: Delete key, R to rotate, Escape to cancel
- Show/Hide names toggle
- Status bar with contextual messages
- Delete confirmation dialog ("Are you sure you want to delete [name]?") for all delete actions

---

## Data Model

### Table Object
```js
{
  type: 'table', id: Number,
  x: Number, y: Number,        // CENTER point
  tableType: 'round' | 'rect',
  name: String, seats: Number, widthFt: Number, heightFt: Number,
  color: String, orientation: 'horizontal' | 'vertical',
  endSeats: Number, locked: Boolean,
  assignments: { [seatIndex]: attendeeIndex }  // e.g. { 0: 3, 2: 7 }
}
```

### Chair Block Object
```js
{
  type: 'block', id: Number,
  x: Number, y: Number,        // TOP-LEFT point
  rows: Number, cols: Number,
  name: String, color: String,
  spacing: 'tight' | 'normal' | 'wide',
  locked: Boolean,
  assignments: { "row-col": attendeeIndex }  // e.g. { "0-2": 5 }
}
```

### Venue Element Object
```js
{
  type: 'venue', id: Number,
  x: Number, y: Number,        // CENTER point
  elementType: String, name: String,
  widthFt: Number, heightFt: Number, color: String, locked: Boolean
}
```

### Attendees
```js
attendees: [ [lastName, firstName], ... ]  // Array of [last, first] pairs
// Attendees are referenced by their array index throughout
```

### Color Palette
```js
COLOR_PALETTE = [
  '#FFFFFF', '#000000',          // White, Black
  '#8B4513', '#C73E1D', ...     // 16 more preset colors
]  // 18 total, used in Modal and List View card color picker
```

---

## CSS Theme Variables

```css
--bg-primary: #1a1a2e      --bg-secondary: #16213e
--bg-tertiary: #0f3460     --bg-card: #1e2a4a
--bg-canvas: #0d1b2a       --accent: #e2b340
--success: #48bb78         --danger: #f56565
--border: #2d3a5a          --text-primary: #e8e8e8
--text-muted: #6b7a90
```

---

## Development

```bash
npm install
npm run dev        # Dev server at localhost:5173
npm run build      # Production build to dist/
npm run preview    # Preview production build
```

## Deploy to Vercel

```bash
# Option 1: CLI
npm install -g vercel
vercel --prod

# Option 2: Push to GitHub, import repo in Vercel dashboard (auto-detects Vite)
```

---

## Known Patterns & Gotchas

1. **Grid snap logic**: `snapEntityPos()` in App.jsx handles different snap behavior per entity type. Round tables snap center, everything else snaps top-left corner. Blocks already store top-left so they snap directly.

2. **ResizeObserver depends on `currentView`**: The canvas container unmounts in List view, so the observer must re-attach when switching back. The dependency array includes `[currentView]`.

3. **Table seat numbering**: Round tables use integer indices (0, 1, 2...). Rect tables also use integers but positions are calculated from the geometry. Blocks use "row-col" string keys like "0-0", "1-2".

4. **Rect seat positioning**: Uses shared `getRectSeatPositions()` helper in canvasRenderer.js for both drawing and hit-testing. Seats are distributed edge-to-edge with small padding, not `(i+1)/(n+1)` gaps. For 1 seat: centered. For 2+: even spread from near-edge to near-edge.

5. **Drag systems — no native drag**: Two custom drag systems coexist — `dragAttendee` (from left panel) and `popupDragSeat` (from popup). Both share `dragGhostPos` for the floating ghost. Attendee items must NOT use the HTML `draggable` attribute — it triggers native browser drag which hijacks mouse events and breaks our custom system. Use `onDragStart={e => e.preventDefault()}` to block native drag. Drop logic fires on **mouseUp** (not mouseDown).

6. **Table popup is an overlay**: Positioned `absolute` inside `.canvas-area` (which has `position: relative`). Does NOT push the canvas grid. Uses `onMouseDown={e => e.stopPropagation()}` to prevent canvas click-through.

7. **Light color text contrast**: `isLightColor()` helper in canvasRenderer.js uses weighted luminance. Light-colored entities (white, yellow, etc.) render table/venue names in black text instead of white.

8. **Save Project file picker**: Uses `window.showSaveFilePicker` (File System Access API) on Chrome/Edge for native save dialog. Falls back to auto-download on Firefox/Safari.

9. **Delete confirmation**: All delete actions (toolbar, List View card, keyboard Delete key) go through `deleteSelected()` or `confirmDeleteEntity()` which shows a Modal with type `'confirm'`.

10. **Toolbar entity name**: Derives from live `tables`/`chairBlocks`/`venueElements` state arrays by matching ID — not from the stale `selectedItem` snapshot. This ensures renames are reflected immediately.

11. **List View card color picker**: Opens as an overlay with dark backdrop (`position: absolute; inset: 0`) inside `.card-body-wrapper`. Click backdrop to dismiss. Does not expand card height.

12. **No preview artifacts needed**: When making changes, just update the source files and rebuild. No need to maintain a separate preview .jsx artifact.

---

## Continuation Prompt

If starting a new Claude chat to continue this project, use something like:

> I'm continuing work on my seating chart designer web app. The source is at https://github.com/ricobuilds/seating-chart-designer. Please pull the latest code and review the PROJECT_INSTRUCTIONS.md for full context on architecture, data models, and patterns. Then let's work on [describe next feature/fix].
