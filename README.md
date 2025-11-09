# Planwerk - Interactive Diagram Builder

A lightweight, browser-based diagram tool for creating system architectures, flowcharts, and technical diagrams. Built to be simple, fast, and hackable.

## What is Planwerk?

Planwerk is an interactive diagram builder that lets you create boxes, circles, lines, and text annotations directly in your browser. It's designed for quick sketching of system architectures, network diagrams, or any visual concept that needs boxes and arrows.

### Features
- **Create diagrams** with boxes, circles, lines, and text
- **Save & load designs** locally in your browser
- **Export/import** as JSON for backup and sharing
- **Grid snapping** (hold 'S' while dragging)
- **Pan & zoom** (hold Space + drag to pan, Ctrl/Cmd + scroll to zoom)

## ⚠️ Important: Where Your Data is Stored

**All designs are saved in your browser's localStorage.** This means:
- Your data is stored **only on your device**
- If you clear your browser data, your designs will be lost
- Designs are not synced across different browsers or devices

**👉 Always export your designs regularly for backup!** Use the "Export" button to download your work as a JSON file.

## Built With

This project is built with [Svedit](https://svedit.dev/), a tiny library for building editable websites in Svelte. Why is this cool?

- **Visual in-place editing** - Edit content directly in the layout, no separate forms needed
- **JSON-based** - Your designs are simple JSON that can be easily backed up and version controlled
- **Minimal & hackable** - Simple codebase using Svelte 5, easy to customize
- **No backend required** - Everything runs in the browser

Technologies used:
- [Svelte 5](https://svelte.dev/) - Modern reactive web framework
- [Svedit](https://svedit.dev/) - Visual editing library
- SVG for crisp, scalable graphics
- Vite for fast development

## 🍴 Fork This!

**Feel free to fork this repository!** If you want to customize Planwerk for your specific needs, forking ensures you have a stable version that won't change unexpectedly. This is especially recommended if you're using it for production work or want to add custom features.

## Installation & Development

```bash
# Install dependencies
npm install

# Run development server
npm run dev
```

Open your browser to the URL shown (typically http://localhost:5173)

## Build for Production

```bash
npm run build
```

The built files will be in the `dist/` directory.

## Keyboard Shortcuts

- **Hold Space + Drag** - Pan the canvas
- **Ctrl/Cmd + Scroll** - Zoom in/out
- **Hold 'S' while dragging** - Snap to grid
- **Delete/Backspace** - Delete selected elements

## License

MIT License - feel free to use, modify, and distribute as you see fit.
