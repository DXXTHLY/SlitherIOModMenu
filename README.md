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

```javascript// ==UserScript==
// @name         DSC.GG/143X ULTIMATE MOD MENU (Stable v9.0)
// @namespace    http://tampermonkey.net/
// @version      9.0
// @description  Ultimate Slither.io Mod Menu, all features actually work!
// @author       GITHUB.COM/DXXTHLY - HTTPS://DSC.GG/143X | waynesg
// @match        http://slither.io/
// @match        https://slither.io/
// @match        http://slither.com/io
// @match        https://slither.com/io
// @grant        none
// ==/UserScript==

(function () {
    'use strict';

    // === CONFIG ===
    const config = {
        menuPosition: 'right',
        defaultCircleRadius: 150,
        circleRadiusStep: 20,
        minCircleRadius: 50,
        maxCircleRadius: 300,
        deathSoundURL: 'https://www.myinstants.com/media/sounds/minecraft-death-sound.mp3'
    };

    // === STATE ===
    const state = {
        features: {
            circleRestriction: false,
            autoCircle: false,
            performanceMode: 1,
            deathSound: true,
            fpsDisplay: false,
            autoBoost: false,
            minimap: false,
            noGlow: false,
            quickRespawn: false,
            showServer: false,
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
        autoCircleAngle: 0,
        minimapCanvas: null,
        ping: 0,
        server: '',
        leaderboard: [],
        lastSnakeAlive: true,
        boostingInterval: null,
    };

    let autoCircleRAF = null;

    // === MENU CREATION ===
    const menu = document.createElement('div');
    menu.id = 'mod-menu';
    menu.style.position = 'fixed';
    menu.style.top = '50px';
    menu.style.background = 'rgba(17, 17, 17, 0.92)';
    menu.style.border = '2px solid #4CAF50';
    menu.style.borderRadius = '10px';
    menu.style.padding = '20px';
    menu.style.zIndex = '9999';
    menu.style.color = '#fff';
    menu.style.fontFamily = 'Arial, sans-serif';
    menu.style.fontSize = '14px';
    menu.style.width = '460px';
    menu.style.boxShadow = '0 0 15px rgba(0,0,0,0.7)';
    menu.style.backdropFilter = 'blur(5px)';
    menu.style.transition = 'all 0.3s ease';
    menu.style.userSelect = "text";
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

    // Ping display
    const pingDisplay = document.createElement('div');
    pingDisplay.id = 'ping-display';
    pingDisplay.style.position = 'fixed';
    pingDisplay.style.bottom = '35px';
    pingDisplay.style.right = '10px';
    pingDisplay.style.color = '#fff';
    pingDisplay.style.fontFamily = 'Arial, sans-serif';
    pingDisplay.style.fontSize = '14px';
    pingDisplay.style.zIndex = '9999';
    pingDisplay.style.display = 'block';
    pingDisplay.style.background = 'rgba(0,0,0,0.5)';
    pingDisplay.style.padding = '5px 10px';
    pingDisplay.style.borderRadius = '5px';
    document.body.appendChild(pingDisplay);

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

    // === MENU CONTENT ===
    function updateMenu() {
        menu.innerHTML = `
            <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:10px">
                <h2 style="margin:0;color:#4CAF50;">GAY MANS MOD MENU</h2>
                <div style="color:#aaa;font-size:12px">v9.0</div>
            </div>
            <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom:15px">
                <div>
                    <h3 style="color:#4CAF50;border-bottom:1px solid #444;padding-bottom:5px;margin-top:0">MOVEMENT</h3>
                    <p><strong>K: Circle Restriction:</strong> <span style="color:${state.features.circleRestriction ? 'lime' : 'red'}">${state.features.circleRestriction ? 'ON' : 'OFF'}</span></p>
                    <p><strong>J/L: Circle Size:</strong> ${state.circleRadius}px</p>
                    <p><strong>A: Auto Circle (left):</strong> <span style="color:${state.features.autoCircle ? 'lime' : 'red'}">${state.features.autoCircle ? 'ON' : 'OFF'}</span></p>
                    <p><strong>B: Auto Boost:</strong> <span style="color:${state.features.autoBoost ? 'lime' : 'red'}">${state.features.autoBoost ? 'ON' : 'OFF'}</span></p>
                    <h3 style="color:#4CAF50;border-bottom:1px solid #444;padding-bottom:5px;margin-top:15px">ZOOM</h3>
                    <p><strong>Z: Zoom In</strong></p>
                    <p><strong>X: Zoom Out</strong></p>
                    <p><strong>C: Reset Zoom</strong></p>
                </div>
                <div>
                    <h3 style="color:#4CAF50;border-bottom:1px solid #444;padding-bottom:5px;margin-top:0">VISUALS</h3>
                    <p><strong>1-3: Performance Mode</strong> <span style="color:${['lime','cyan','orange'][state.features.performanceMode-1] || '#aaa'}">${['Low: Minimal','Medium: Balanced','High: Quality'][state.features.performanceMode-1] || 'Off'}</span></p>
                    <p><strong>F: FPS Display:</strong> <span style="color:${state.features.fpsDisplay ? 'lime' : 'red'}">${state.features.fpsDisplay ? 'ON' : 'OFF'}</span></p>
                    <p><strong>N: Minimap:</strong> <span style="color:${state.features.minimap ? 'lime' : 'red'}">${state.features.minimap ? 'ON' : 'OFF'}</span></p>
                    <p><strong>V: Death Sound:</strong> <span style="color:${state.features.deathSound ? 'lime' : 'red'}">${state.features.deathSound ? 'ON' : 'OFF'}</span></p>
                    <p><strong>U: Quick Respawn</strong></p>
                    <p><strong>H: No Glow:</strong> <span style="color:${state.features.noGlow ? 'lime' : 'red'}">${state.features.noGlow ? 'ON' : 'OFF'}</span></p>
                    <p><strong>T: Show Server IP:</strong> <span style="color:${state.features.showServer ? 'lime' : 'red'}">${state.features.showServer ? 'ON' : 'OFF'}</span></p>
                    <h3 style="color:#4CAF50;border-bottom:1px solid #444;padding-bottom:5px;margin-top:15px">LINKS</h3>
                    <p><strong>G: GitHub</strong></p>
                    <p><strong>D: Discord</strong></p>
                </div>
            </div>
            <div style="display:grid;grid-template-columns:1fr 1fr;gap:10px;background:rgba(76, 175, 80, 0.1);padding:10px;border-radius:5px;margin-bottom:15px">
                <div>
                    <h3 style="color:#4CAF50;margin-top:0;margin-bottom:10px">STATUS</h3>
                    <p><strong>Game State:</strong> ${state.isInGame ? 'In Game' : 'Menu'}</p>
                    <p><strong>Zoom:</strong> ${Math.round(100 / state.zoomFactor)}%</p>
                    <p><strong>Ping:</strong> <span id="ping-value">${state.ping} ms</span></p>
                    <p><strong>FPS:</strong> ${state.fps}</p>
                </div>
                <div>
                    <h3 style="color:#4CAF50;margin-top:0;margin-bottom:10px">EXTRA</h3>
                    <p><strong>Server:</strong> ${state.features.showServer ? (state.server || 'N/A') : 'Hidden'}</p>
                    <p><strong>Leaderboard:</strong><br>
                        ${state.leaderboard.length ? state.leaderboard.map((x,i)=>`${i+1}. ${x}`).join('<br>') : 'N/A'}
                    </p>
                </div>
            </div>
            <div style="text-align:center;font-size:12px;color:#aaa;border-top:1px solid #444;padding-top:10px">
                Press <strong>M</strong> to hide/show menu | DSC.GG/143X | <strong>P</strong> Screenshot<br>
                <span style="color:#aaa;">Made by: <b>dxxthly.</b> & <b>waynesg</b> on Discord</span>
            </div>
        `;
    }
    updateMenu();

    // === GAME STATE DETECTION ===
    function checkGameState() {
        const gameCanvas = document.querySelector('canvas');
        const loginForm = document.getElementById('login');
        state.isInGame = !!(gameCanvas && gameCanvas.style.display !== 'none' && (!loginForm || loginForm.style.display === 'none'));
        setTimeout(checkGameState, 1000);
    }
    checkGameState();

    // === CIRCLE RESTRICTION ===
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

    document.addEventListener('mousemove', function restrictToCircle(e) {
        if (!state.features.circleRestriction || !state.isInGame) return;
        const centerX = window.innerWidth / 2;
        const centerY = window.innerHeight / 2;
        const dx = e.clientX - centerX;
        const dy = e.clientY - centerY;
        const distance = Math.sqrt(dx * dx + dy * dy);
        if (distance > state.circleRadius) {
            e.stopImmediatePropagation();
            e.preventDefault();
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
    }, true);

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



    // === AUTO BOOST: Works on both slither.io and slither.com/io ===
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

    // === MINIMAP (shows your snake as a white dot) ===
    function drawMinimap() {
        if (!state.minimapCanvas) return;
        if (state.features.minimap && state.isInGame) {
            state.minimapCanvas.style.display = 'block';
            const ctx = state.minimapCanvas.getContext('2d');
            ctx.clearRect(0, 0, state.minimapCanvas.width, state.minimapCanvas.height);
            ctx.strokeStyle = "#4CAF50";
            ctx.lineWidth = 2;
            ctx.strokeRect(0, 0, 160, 160);
            if (window.snake && typeof window.snake.xx === "number" && typeof window.snake.yy === "number") {
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

    // === FPS COUNTER ===
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

    // === DEATH SOUND (robust: plays on death overlay and snake death) ===
    function deathSoundObserver() {
        let lastState = true;
        setInterval(() => {
            if (!state.features.deathSound) return;
            // Play sound if snake just died
            if (window.snake && lastState && !window.snake.alive) {
                state.deathSound.pause();
                state.deathSound.currentTime = 0;
                state.deathSound.play();
            }
            // Play sound if "died" overlay appears
            const died = document.getElementById('died');
            if (died && died.style.display !== 'none' && lastState) {
                state.deathSound.pause();
                state.deathSound.currentTime = 0;
                state.deathSound.play();
            }
            lastState = window.snake ? window.snake.alive : false;
        }, 100);
    }
    state.deathSound.preload = 'auto';
    state.deathSound.load();
    state.deathSound.addEventListener('ended', () => {
        state.deathSound.currentTime = 0;
    });
    deathSoundObserver();

    // === NO GLOW (removes snake glow for FPS, works on .io and .com) ===
    function noGlowLoop() {
        if (state.features.noGlow && window.snakes) {
            for (let s of window.snakes) {
                if (s && s.rbcs !== undefined) s.rbcs = 0;
            }
        }
        requestAnimationFrame(noGlowLoop);
    }
    noGlowLoop();

    // === PERFORMANCE MODES ===
    function applyPerformanceMode() {
        if (typeof window !== "undefined") {
            switch (state.features.performanceMode) {
                case 1: // Low
                    window.want_quality = 0;
                    window.high_quality = false;
                    window.render_mode = 1;
                    break;
                case 2: // Medium
                    window.want_quality = 1;
                    window.high_quality = false;
                    window.render_mode = 2;
                    break;
                case 3: // High
                    window.want_quality = 2;
                    window.high_quality = true;
                    window.render_mode = 2;
                    break;
                default:
                    break;
            }
        }
        updateMenu();
    }

    // === ZOOM LOCK ===
    function zoomLockLoop() {
        if (typeof window.gsc !== 'undefined') {
            window.gsc = state.zoomFactor;
        }
        requestAnimationFrame(zoomLockLoop);
    }
    zoomLockLoop();

    // === PING DISPLAY ===
    function pingLoop() {
        let ping = 0;
        if (window.lagging && typeof window.lagging === "number") {
            ping = Math.round(window.lagging);
        } else if (window.lag && typeof window.lag === "number") {
            ping = Math.round(window.lag);
        }
        state.ping = ping;
        pingDisplay.textContent = `Ping: ${ping} ms`;
        const pingValue = document.getElementById("ping-value");
        if (pingValue) pingValue.textContent = `${ping} ms`;
        setTimeout(pingLoop, 500);
    }
    pingLoop();

    // === SCREENSHOT BUTTON (P) ===
    document.addEventListener('keydown', function (e) {
        if (e.key.toLowerCase() === 'p' && state.isInGame) {
            try {
                const canvas = document.querySelector('canvas');
                if (canvas) {
                    const dataURL = canvas.toDataURL('image/png');
                    const a = document.createElement('a');
                    a.href = dataURL;
                    a.download = `slitherio_screenshot_${Date.now()}.png`;
                    a.click();
                }
            } catch (err) {
                alert('Screenshot failed.');
            }
        }
    });

    // === QUICK RESPAWN (U) ===
    document.addEventListener('keydown', function (e) {
        if (e.key.toLowerCase() === 'u' && !state.isInGame) {
            const playBtn = document.getElementById('playh');
            if (playBtn) playBtn.click();
            else if (typeof window.play_btn_click === 'function') window.play_btn_click();
        }
    });

    // === GET SERVER IP ===
    function getServerIP() {
        if (window.bso && window.bso.ip) {
            state.server = window.bso.ip;
        } else if (window.bso && window.bso.url) {
            state.server = window.bso.url;
        } else {
            state.server = '';
        }
    }
    setInterval(getServerIP, 1000);

    // === GET LEADERBOARD ===
    function getLeaderboard() {
        // Try to get leaderboard names from the DOM
        const lb = [];
        for (let i = 1; i <= 3; ++i) {
            const el = document.getElementById('lb' + i);
            if (el && el.textContent) lb.push(el.textContent.trim());
        }
        state.leaderboard = lb;
    }
    setInterval(getLeaderboard, 1000);

    // === MAIN LOOPS ===
    function mainLoop() {
        if (state.features.autoCircle && state.isInGame) autoCircle();
        autoBoost();
        requestAnimationFrame(mainLoop);
    }
    mainLoop();

    // === KEYBOARD SHORTCUTS ===
    document.addEventListener('keydown', function (e) {
        if (e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') return;
        switch (e.key.toLowerCase()) {
            case 'm':
                state.menuVisible = !state.menuVisible;
                menu.style.display = state.menuVisible ? 'block' : 'none';
                break;
            case 'k':
                state.features.circleRestriction = !state.features.circleRestriction;
                updateMenu();
                break;
            case 'j':
                state.circleRadius = Math.max(config.minCircleRadius, state.circleRadius - config.circleRadiusStep);
                updateMenu();
                break;
            case 'l':
                state.circleRadius = Math.min(config.maxCircleRadius, state.circleRadius + config.circleRadiusStep);
                updateMenu();
                break;
            case 'a':
                state.features.autoCircle = !state.features.autoCircle;
                if (state.features.autoCircle) {
                    if (!autoCircleRAF) autoCircle();
                } else {
                    if (autoCircleRAF) {
                        cancelAnimationFrame(autoCircleRAF);
                        autoCircleRAF = null;
                    }
                }
                updateMenu();
                break;


            case 'b':
                state.features.autoBoost = !state.features.autoBoost;
                updateMenu();
                break;
            case 'z':
                state.zoomFactor = Math.max(0.3, state.zoomFactor - 0.1);
                updateMenu();
                break;
            case 'x':
                state.zoomFactor = Math.min(2.0, state.zoomFactor + 0.1);
                updateMenu();
                break;
            case 'c':
                state.zoomFactor = 1.0;
                updateMenu();
                break;
            case '1':
            case '2':
            case '3':
                state.features.performanceMode = parseInt(e.key);
                applyPerformanceMode();
                break;
            case 'f':
                state.features.fpsDisplay = !state.features.fpsDisplay;
                fpsDisplay.style.display = state.features.fpsDisplay ? 'block' : 'none';
                updateMenu();
                break;
            case 'n':
                state.features.minimap = !state.features.minimap;
                updateMenu();
                break;
            case 'v':
                state.features.deathSound = !state.features.deathSound;
                updateMenu();
                break;
            case 'h':
                state.features.noGlow = !state.features.noGlow;
                updateMenu();
                break;
            case 't':
                state.features.showServer = !state.features.showServer;
                updateMenu();
                break;
            case 'g':
                window.open('https://github.com/Dxxthly', '_blank');
                break;
            case 'd':
                window.open('https://dsc.gg/143x', '_blank');
                break;
        }
    });

    // === INITIAL SETUP ===
    applyPerformanceMode();

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
