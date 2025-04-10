# Vue Teleport to Picture-in-Picture Demo

This demo showcases how to use Vue.js Teleport feature to move components into a Picture-in-Picture window while maintaining their reactivity and state.

## Features

- Open a Picture-in-Picture (PiP) window
- Teleport Vue components to the PiP window
- Maintain component reactivity (state changes reflect in the PiP window)
- Two-way communication between main window and PiP window
- Live clock and counter demonstration

## Technical Implementation

- Uses Vue 3's Teleport feature
- Leverages the Document Picture-in-Picture API
- MessageChannel for communication between windows

## Prerequisites

- Modern browser with Picture-in-Picture API support
  - Chrome 116+ or Edge 116+

## Running the Demo

```bash
# Install dependencies
npm install

# Start the development server
npm run dev
```

## How it Works

1. The main Vue application creates a PiP window using the Document Picture-in-Picture API
2. Vue's Teleport feature moves the component's rendered HTML to the PiP window
3. Component logic continues to run in the main window
4. State changes (like button clicks) reflect in the teleported UI

The power of this approach is that the component's logic remains in the Vue app's context while its DOM output appears in the PiP window.
