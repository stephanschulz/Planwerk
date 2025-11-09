# vBrain System Architecture - Interactive Diagram

An interactive Svelte web application visualizing the Pulse Garden vBrain rectangular lighting system.

## System Overview

- **2 lines** of lighting
- **200 lights per line** (400 total)
- **50 vBrain controllers per line** (100 total)
- **25 vBrains per group** (4 groups total)
- Each vBrain controls **4 rectangular lights**

## Installation

```bash
npm install
```

## Development

```bash
npm run dev
```

Open your browser to the URL shown (typically http://localhost:5173)

## Build for Production

```bash
npm run build
```

The built files will be in the `dist/` directory.

## Features

- Interactive SVG diagram showing both lighting lines
- Click on vBrain units to see details
- Visual representation of:
  - AC power bus with T-junction topology
  - Data daisy chain connections
  - vBrain controllers
  - Rectangular lights
  - Group organization
- Detailed topology view
- Comprehensive legend
- Responsive design

## Technology

Built with:
- [Svelte 5](https://svelte.dev/) - Modern reactive web framework
- SVG for scalable, crisp diagrams
- Vite for fast development and building

## Structure

```
src/
├── App.svelte              # Main application component
├── components/
│   ├── SystemDiagram.svelte   # Main system diagram (2 lines)
│   ├── DetailView.svelte      # Connection topology detail
│   └── Legend.svelte          # Legend and system summary
├── main.js                 # Application entry point
└── app.css                 # Global styles
```

