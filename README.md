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

## Script Codes

```javascript
// ==UserScript==
// @name         DSC.GG/143X ULTIMATE MOD MENU
// @namespace    http://tampermonkey.net/
// @version      7.9
// @description  Ultimate Slither.io Mod Menu with Circle Restriction, Auto-Circle, Zoom, Performance Modes, Minimap, and more!
// @author       GITHUB.COM/DXXTHLY - HTTPS://DSC.GG/143X
// @match        http://slither.io/
// @match        https://slither.io/
// @match        http://slither.com/io
// @match        https://slither.com/io
// @grant        none
// ==/UserScript==

(function () {
    'use strict';

    // =====================
    // Configuration
    // =====================
    const config = {
        menuPosition: 'right',
        defaultCircleRadius: 150,
        circleRadiusStep: 20,
        minCircleRadius: 50,
        maxCircleRadius: 300,
        deathSoundURL: 'https://www.myinstants.com/media/sounds/minecraft-death-sound.mp3'
    };

    // =====================
    // State object
    // =====================
    const state = {
        features: {
            circleRestriction: false,
            autoCircle: false,
            performanceMode: 0,
            deathSound: false,
            fpsDisplay: false,
            autoBoost: false,
            rainbowSkin: false,
            minimap: false
        },
        menuVisible: true,
        zoomFactor: 1.0,
        circleRadius: config.defaultCircleRadius,
        fps: 0,
        fpsFrames: 0,
        fpsLastCheck: Date.now(),
        deathSound: new Audio(config.deathSoundURL),
        isInGame: false,
        boosting: false,
        lastSnakeAlive: true,
        autoCircleAngle: 0,
        minimapCanvas: null
    };

    // =====================
    // Create and style the menu
    // =====================
    const menu = document.createElement('div');
    menu.id = 'mod-menu';
    menu.style.position = 'fixed';
    menu.style.top = '50px';
    menu.style.background = 'rgba(17, 17, 17, 0.9)';
    menu.style.border = '2px solid #4CAF50';
    menu.style.borderRadius = '10px';
    menu.style.padding = '20px';
    menu.style.zIndex = '9999';
    menu.style.color = '#fff';
    menu.style.fontFamily = 'Arial, sans-serif';
    menu.style.fontSize = '14px';
    menu.style.width = '450px';
    menu.style.boxShadow = '0 0 15px rgba(0,0,0,0.7)';
    menu.style.backdropFilter = 'blur(5px)';
    menu.style.transition = 'all 0.3s ease';

    if (config.menuPosition === 'left') {
        menu.style.left = '50px';
    } else if (config.menuPosition === 'center') {
        menu.style.left = '50%';
        menu.style.transform = 'translateX(-50%)';
    } else {
        menu.style.right = '50px';
    }
    document.body.appendChild(menu);

    // FPS display
    const fpsDisplay = document.createElement('div');
    fpsDisplay.id = 'fps-display';
    fpsDisplay.style.position = 'fixed';
    fpsDisplay.style.bottom = '10px';
    fpsDisplay.style.right = '10px';
    fpsDisplay.style.color = '#fff';
    fpsDisplay.style.fontFamily = 'Arial, sans-serif';
    fpsDisplay.style.fontSize = '14px';
    fpsDisplay.style.zIndex = '9999';
    fpsDisplay.style.display = 'none';
    fpsDisplay.style.background = 'rgba(0,0,0,0.5)';
    fpsDisplay.style.padding = '5px 10px';
    fpsDisplay.style.borderRadius = '5px';
    document.body.appendChild(fpsDisplay);

    // Circle restriction visual
    const circleVisual = document.createElement('div');
    circleVisual.id = 'circle-visual';
    circleVisual.style.position = 'fixed';
    circleVisual.style.border = '2px dashed rgba(76, 175, 80, 0.7)';
    circleVisual.style.borderRadius = '50%';
    circleVisual.style.pointerEvents = 'none';
    circleVisual.style.transform = 'translate(-50%, -50%)';
    circleVisual.style.zIndex = '9998';
    circleVisual.style.display = 'none';
    circleVisual.style.transition = 'all 0.2s ease';
    document.body.appendChild(circleVisual);

    // Minimap overlay
    function createMinimap() {
        if (state.minimapCanvas) return;
        const canvas = document.createElement('canvas');
        canvas.width = 160;
        canvas.height = 160;
        canvas.style.position = 'fixed';
        canvas.style.left = '10px';
        canvas.style.bottom = '10px';
        canvas.style.zIndex = '9999';
        canvas.style.background = 'rgba(0,0,0,0.5)';
        canvas.style.border = '2px solid #4CAF50';
        canvas.style.borderRadius = '10px';
        canvas.style.display = 'none';
        document.body.appendChild(canvas);
        state.minimapCanvas = canvas;
    }
    createMinimap();

    // =====================
    // Menu Content
    // =====================
    function updateMenu() {
        menu.innerHTML = `
            <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:10px">
                <h2 style="margin:0;color:#4CAF50;">GAY MANS MOD MENU</h2>
                <div style="color:#aaa;font-size:12px">v7.9</div>
            </div>
            <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom:15px">
                <div>
                    <h3 style="color:#4CAF50;border-bottom:1px solid #444;padding-bottom:5px;margin-top:0">MOVEMENT</h3>
                    <p><strong>K: Circle Restriction:</strong> <span style="color:${state.features.circleRestriction ? 'lime' : 'red'}">${state.features.circleRestriction ? 'ON' : 'OFF'}</span></p>
                    <p><strong>J/L: Circle Size:</strong> ${state.circleRadius}px</p>
                    <p><strong>A: Auto Circle:</strong> <span style="color:${state.features.autoCircle ? 'lime' : 'red'}">${state.features.autoCircle ? 'ON' : 'OFF'}</span></p>
                    <p><strong>B: Auto Boost:</strong> <span style="color:${state.features.autoBoost ? 'lime' : 'red'}">${state.features.autoBoost ? 'ON' : 'OFF'}</span></p>
                    <h3 style="color:#4CAF50;border-bottom:1px solid #444;padding-bottom:5px;margin-top:15px">ZOOM</h3>
                    <p><strong>Z: Zoom In</strong></p>
                    <p><strong>X: Zoom Out</strong></p>
                    <p><strong>C: Reset Zoom</strong></p>
                </div>
                <div>
                    <h3 style="color:#4CAF50;border-bottom:1px solid #444;padding-bottom:5px;margin-top:0">VISUALS</h3>
                    <p><strong>1-4: Performance Mode</strong> <span style="color:${state.features.performanceMode ? ['lime','cyan','orange','red'][state.features.performanceMode-1] : '#aaa'}">${state.features.performanceMode ? ['Low','Medium','High','Ultra'][state.features.performanceMode-1] : 'Off'}</span></p>
                    <p><strong>F: FPS Display:</strong> <span style="color:${state.features.fpsDisplay ? 'lime' : 'red'}">${state.features.fpsDisplay ? 'ON' : 'OFF'}</span></p>
                    <p><strong>R: Rainbow Skin:</strong> <span style="color:${state.features.rainbowSkin ? 'lime' : 'red'}">${state.features.rainbowSkin ? 'ON' : 'OFF'}</span></p>
                    <p><strong>N: Minimap:</strong> <span style="color:${state.features.minimap ? 'lime' : 'red'}">${state.features.minimap ? 'ON' : 'OFF'}</span></p>
                    <p><strong>V: Death Sound:</strong> <span style="color:${state.features.deathSound ? 'lime' : 'red'}">${state.features.deathSound ? 'ON' : 'OFF'}</span></p>
                    <h3 style="color:#4CAF50;border-bottom:1px solid #444;padding-bottom:5px;margin-top:15px">LINKS</h3>
                    <p><strong>G: GitHub</strong></p>
                    <p><strong>D: Discord</strong></p>
                </div>
            </div>
            <div style="background:rgba(76, 175, 80, 0.1);padding:10px;border-radius:5px;margin-bottom:15px">
                <h3 style="color:#4CAF50;margin-top:0;margin-bottom:10px">STATUS</h3>
                <p><strong>Game State:</strong> ${state.isInGame ? 'In Game' : 'Menu'}</p>
                <p><strong>Zoom Level:</strong> ${Math.round(100 / state.zoomFactor)}%</p>
            </div>
            <div style="text-align:center;font-size:12px;color:#aaa;border-top:1px solid #444;padding-top:10px">
                Press <strong>M</strong> to hide/show menu | DSC.GG/143X
            </div>
        `;
    }
    updateMenu();

    // =====================
    // Track Game State
    // =====================
    function checkGameState() {
        const gameCanvas = document.querySelector('canvas');
        state.isInGame = !!(gameCanvas && gameCanvas.style.display !== 'none');
        setTimeout(checkGameState, 1000);
    }
    checkGameState();

    // =====================
    // Draw Circle Restriction (centered in viewport)
    // =====================
    function drawCircleRestriction() {
        if (state.features.circleRestriction && state.isInGame) {
            const centerX = window.innerWidth / 2;
            const centerY = window.innerHeight / 2;
            circleVisual.style.left = `${centerX}px`;
            circleVisual.style.top = `${centerY}px`;
            circleVisual.style.width = `${state.circleRadius * 2}px`;
            circleVisual.style.height = `${state.circleRadius * 2}px`;
            circleVisual.style.display = 'block';
        } else {
            circleVisual.style.display = 'none';
        }
        requestAnimationFrame(drawCircleRestriction);
    }
    drawCircleRestriction();

    // =====================
    // Circle Restriction Logic
    // =====================
    function restrictToCircle(e) {
        if (!state.features.circleRestriction || !state.isInGame) return;
        const centerX = window.innerWidth / 2;
        const centerY = window.innerHeight / 2;
        const dx = e.clientX - centerX;
        const dy = e.clientY - centerY;
        const distance = Math.sqrt(dx * dx + dy * dy);
        if (distance > state.circleRadius) {
            const angle = Math.atan2(dy, dx);
            const newX = centerX + Math.cos(angle) * state.circleRadius;
            const newY = centerY + Math.sin(angle) * state.circleRadius;
            const canvas = document.querySelector('canvas');
            if (canvas) {
                const event = new MouseEvent('mousemove', {
                    clientX: newX,
                    clientY: newY,
                    bubbles: true
                });
                canvas.dispatchEvent(event);
            }
        }
    }

    // =====================
    // Auto Circle: Move mouse in a real circle
    // =====================
    function autoCircle() {
        if (!state.features.autoCircle || !state.isInGame) return;
        state.autoCircleAngle += 0.04;
        const centerX = window.innerWidth / 2;
        const centerY = window.innerHeight / 2;
        const radius = state.circleRadius * 0.8;
        const moveX = centerX + Math.cos(state.autoCircleAngle) * radius;
        const moveY = centerY + Math.sin(state.autoCircleAngle) * radius;
        const canvas = document.querySelector('canvas');
        if (canvas) {
            const event = new MouseEvent('mousemove', {
                clientX: moveX,
                clientY: moveY,
                bubbles: true
            });
            canvas.dispatchEvent(event);
        }
    }

    // =====================
    // Auto Boost: Works on both slither.io and slither.com/io
    // =====================
    function autoBoost() {
        if (!state.features.autoBoost || !state.isInGame) {
            if (state.boosting) {
                state.boosting = false;
                if (typeof window.setAcceleration === 'function') window.setAcceleration(0);
                // fallback for slither.io
                document.dispatchEvent(new KeyboardEvent('keyup', { key: ' ' }));
            }
            return;
        }
        if (!state.boosting) {
            state.boosting = true;
            if (typeof window.setAcceleration === 'function') window.setAcceleration(1);
            // fallback for slither.io
            document.dispatchEvent(new KeyboardEvent('keydown', { key: ' ' }));
        }
    }

    // =====================
    // Minimap
    // =====================
    function drawMinimap() {
        if (!state.minimapCanvas) return;
        if (state.features.minimap && state.isInGame) {
            state.minimapCanvas.style.display = 'block';
            const ctx = state.minimapCanvas.getContext('2d');
            ctx.clearRect(0, 0, state.minimapCanvas.width, state.minimapCanvas.height);
            // Draw map border
            ctx.strokeStyle = "#4CAF50";
            ctx.lineWidth = 2;
            ctx.strokeRect(0, 0, 160, 160);
            // Draw your snake
            if (window.snake && window.snake.xx != null && window.snake.yy != null) {
                // Slither.io map is 21600x21600, center is 10800,10800
                const mapSize = 21600;
                const scale = 160 / mapSize;
                const x = window.snake.xx * scale;
                const y = window.snake.yy * scale;
                ctx.fillStyle = "#fff";
                ctx.beginPath();
                ctx.arc(x, y, 5, 0, 2 * Math.PI);
                ctx.fill();
            }
        } else {
            state.minimapCanvas.style.display = 'none';
        }
        requestAnimationFrame(drawMinimap);
    }
    drawMinimap();

    // =====================
    // FPS Counter
    // =====================
    function fpsCounter() {
        state.fpsFrames++;
        const now = Date.now();
        if (now - state.fpsLastCheck >= 1000) {
            state.fps = state.fpsFrames;
            state.fpsFrames = 0;
            state.fpsLastCheck = now;
            if (state.features.fpsDisplay) {
                fpsDisplay.textContent = `FPS: ${state.fps}`;
            }
        }
        requestAnimationFrame(fpsCounter);
    }
    fpsCounter();

    // =====================
    // Death Sound
    // =====================
    function checkDeath() {
        if (!state.features.deathSound) {
            setTimeout(checkDeath, 500);
            return;
        }
        if (window.snake && state.lastSnakeAlive && !window.snake.alive) {
            state.deathSound.play();
        }
        state.lastSnakeAlive = window.snake ? window.snake.alive : false;
        setTimeout(checkDeath, 500);
    }
    setTimeout(checkDeath, 2000);

    // =====================
    // Performance Modes
    // =====================
    function applyPerformanceMode() {
        // Fallback: try to set some common global vars for performance
        switch (state.features.performanceMode) {
            case 1: // Low
                window.high_quality = false;
                window.render_mode = 1;
                window.want_quality = 0;
                break;
            case 2: // Medium
                window.high_quality = false;
                window.render_mode = 2;
                window.want_quality = 1;
                break;
            case 3: // High
                window.high_quality = true;
                window.render_mode = 2;
                window.want_quality = 1;
                break;
            case 4: // Ultra
                window.high_quality = true;
                window.render_mode = 2;
                window.want_quality = 1;
                break;
            default:
                break;
        }
    }

    // =====================
    // Zoom Lock: Force zoom every frame
    // =====================
    function zoomLockLoop() {
        if (typeof window.gsc !== 'undefined') {
            window.gsc = state.zoomFactor;
        }
        requestAnimationFrame(zoomLockLoop);
    }
    zoomLockLoop();

    // =====================
    // Event Listeners
    // =====================
    document.addEventListener('mousemove', restrictToCircle);

    document.addEventListener('keydown', (e) => {
        const key = e.key.toLowerCase();

        // Toggle menu
        if (key === 'm') {
            state.menuVisible = !state.menuVisible;
            menu.style.display = state.menuVisible ? 'block' : 'none';
            return;
        }

        // Circle restriction toggle
        if (key === 'k') {
            state.features.circleRestriction = !state.features.circleRestriction;
            updateMenu();
            return;
        }

        // Auto circle toggle
        if (key === 'a') {
            state.features.autoCircle = !state.features.autoCircle;
            updateMenu();
            return;
        }

        // Auto boost toggle
        if (key === 'b') {
            state.features.autoBoost = !state.features.autoBoost;
            updateMenu();
            return;
        }

        // Rainbow skin toggle
        if (key === 'r') {
            state.features.rainbowSkin = !state.features.rainbowSkin;
            updateMenu();
            return;
        }

        // Minimap toggle
        if (key === 'n') {
            state.features.minimap = !state.features.minimap;
            updateMenu();
            return;
        }

        // Death sound toggle
        if (key === 'v') {
            state.features.deathSound = !state.features.deathSound;
            updateMenu();
            return;
        }

        // Circle size adjustment
        if (key === 'j') {
            state.circleRadius = Math.max(config.minCircleRadius, state.circleRadius - config.circleRadiusStep);
            updateMenu();
            return;
        }
        if (key === 'l') {
            state.circleRadius = Math.min(config.maxCircleRadius, state.circleRadius + config.circleRadiusStep);
            updateMenu();
            return;
        }

        // Performance modes
        if (key >= '1' && key <= '4') {
            state.features.performanceMode = parseInt(key);
            applyPerformanceMode();
            updateMenu();
            return;
        }

        // FPS display
        if (key === 'f') {
            state.features.fpsDisplay = !state.features.fpsDisplay;
            fpsDisplay.style.display = state.features.fpsDisplay ? 'block' : 'none';
            updateMenu();
            return;
        }

        // Quick links
        if (key === 'g') {
            window.open('https://github.com/DXXTHLY', '_blank');
            return;
        }
        if (key === 'd') {
            window.open('https://dsc.gg/143x', '_blank');
            return;
        }

        // Zoom controls
        if (key === 'z') {
            state.zoomFactor = Math.max(0.3, state.zoomFactor - 0.05);
            updateMenu();
            return;
        }
        if (key === 'x') {
            state.zoomFactor = Math.min(3.0, state.zoomFactor + 0.05);
            updateMenu();
            return;
        }
        if (key === 'c') {
            state.zoomFactor = 1.0;
            updateMenu();
            return;
        }
    });

    // =====================
    // Main Loop
    // =====================
    function mainLoop() {
        if (state.features.autoCircle && state.isInGame) autoCircle();
        if (state.features.autoBoost && state.isInGame) autoBoost();
        else if (!state.features.autoBoost && state.boosting) {
            state.boosting = false;
            if (typeof window.setAcceleration === 'function') window.setAcceleration(0);
            document.dispatchEvent(new KeyboardEvent('keyup', { key: ' ' }));
        }
        requestAnimationFrame(mainLoop);
    }
    mainLoop();

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
