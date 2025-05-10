# Slither.io Enhanced Zoom Control

A Tampermonkey userscript that adds zoom controls to Slither.io (Z=zoom in, X=zoom out, C=reset to default).

## Installation

1. **Install Tampermonkey** (browser extension):
   - [Chrome](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo)
   - [Firefox](https://addons.mozilla.org/firefox/addon/tampermonkey/)
   - [Edge](https://microsoftedge.microsoft.com/addons/detail/tampermonkey/iikmkjmpaadaobahmlepeloendndfphd)

2. **Enable Developer Mode** in your browser (required for extensions)

3. **Install the Script**:
   - Open Tampermonkey dashboard
   - Click "Utilities" tab
   - Paste the following code into the "Import from URL" field:
     ```
     https://github.com/yourusername/your-repo/raw/main/script.user.js
     ```
   - OR manually create a new script and paste the code below

## Script Code

```javascript
// ==UserScript==
// @name         Slither.io Enhanced Zoom Control
// @namespace    http://tampermonkey.net/
// @version      1.3
// @description  Enhanced zoom controls for Slither.io (Z=zoom in, X=zoom out, C=reset to default)
// @author       DXXTHLY GITHUB
// @match        http://slither.io/
// @match        https://slither.io/
// @grant        none
// @license      MIT
// ==/UserScript==

(function() {
    'use strict';

    // Configuration
    const ZOOM_IN_FACTOR = 0.9;      // 10% zoom in
    const ZOOM_OUT_FACTOR = 1.11;    // ~10% zoom out
    const POLLING_INTERVAL = 100;    // ms
    const DEFAULT_ZOOM = undefined;  // Let game handle zoom

    // State management
    let desiredZoom = DEFAULT_ZOOM;
    let isEnabled = true;

    // Main control loop
    const zoomControl = setInterval(() => {
        if (desiredZoom !== undefined && window.gsc !== undefined) {
            window.gsc = desiredZoom;
        }
    }, POLLING_INTERVAL);

    // Keyboard controls
    document.addEventListener('keydown', (e) => {
        if (!isEnabled || window.gsc === undefined) return;

        switch (e.key.toLowerCase()) {
            case 'z':
                desiredZoom = window.gsc * ZOOM_IN_FACTOR;
                break;
            case 'x':
                desiredZoom = window.gsc * ZOOM_OUT_FACTOR;
                break;
            case 'c':
                desiredZoom = DEFAULT_ZOOM;
                break;
        }
    });

    console.log('Slither.io Enhanced Zoom Control loaded');
})();
```

## Usage:
+ Z key: Zoom in (10%)
+ X key: Zoom out (~10%)
+ C key: Reset to default zoom

Notes
Works on both http and https versions of slither.io
Requires Tampermonkey or similar userscript manager
May need to refresh the game page after installation
