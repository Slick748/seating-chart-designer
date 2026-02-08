# Seating Chart Designer — Web App

A full-featured event seating chart designer, ported from Python/tkinter to React for web hosting.

## Features

- **Canvas View** — Drag-and-drop table placement on a scalable room canvas
- **List View** — Card-based seat management with inline assignment
- **Table Types** — Round tables, rectangular tables, chair blocks
- **Venue Elements** — Dance floor, stage, bar, DJ booth, buffet
- **Attendee Management** — Load from CSV, add manually, disable/enable, search
- **Auto-Assignment** — Alphabetical, fill order, or random
- **Drag & Drop** — Drag attendees from the panel to seats on canvas
- **Edit/Rotate/Copy/Lock/Delete** — Full entity manipulation
- **Zoom & Grid** — Configurable grid snapping and zoom levels
- **Save/Load** — Project persistence as JSON files
- **Export** — CSV export of seating assignments
- **Keyboard Shortcuts** — Ctrl+Z undo, Delete, R rotate, Escape cancel
- **Undo** — 50-level undo stack

## Deploy to Vercel

### Option A: Vercel CLI

```bash
# Install Vercel CLI globally
npm install -g vercel

# Navigate to project
cd seating-chart-designer

# Install dependencies
npm install

# Deploy
vercel

# For production:
vercel --prod
```

### Option B: GitHub + Vercel Dashboard

1. Push this folder to a GitHub repository
2. Go to [vercel.com](https://vercel.com) and click **"Add New Project"**
3. Import your GitHub repository
4. Vercel auto-detects Vite — just click **Deploy**

### Option C: Vercel Dashboard Upload

1. Run `npm install && npm run build` locally
2. Go to Vercel dashboard → **Add New Project**
3. Select **"Deploy from template"** or drag-drop the `dist/` folder

## Local Development

```bash
npm install
npm run dev
```

Opens at `http://localhost:5173`

## Project Structure

```
seating-chart-designer/
├── index.html
├── package.json
├── vite.config.js
├── vercel.json           # Vercel deployment config
├── public/
│   └── favicon.svg
└── src/
    ├── main.jsx          # Entry point
    ├── App.jsx           # Main application component
    ├── models.js         # Data models and utilities
    ├── canvasRenderer.js # Canvas 2D rendering engine
    └── styles.css        # Global styles
```

## Usage Guide

1. **Add Tables** — Click "+ Add" → select table type → configure → click "Add" → click canvas to place
2. **Load Attendees** — Click "Load CSV" in the left panel (CSV format: `LastName, FirstName`)
3. **Assign Seats** — Select an attendee in the left panel, then click an empty seat in list view. Or drag attendees onto seats in canvas view.
4. **Auto-Assign** — Use Tools → Auto-Assign to bulk-fill remaining seats
5. **Edit** — Double-click entities on canvas, or select and click "Edit"
6. **Save** — Click "Save" in the bottom bar to download a project JSON file
