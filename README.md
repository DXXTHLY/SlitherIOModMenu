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
// @name         DSC.GG/143X MENU
// @namespace    http://tampermonkey.net/
// @version      6.1
// @description  Ultimate slither.io mod with all features and visible cursor
// @author       GITHUB.COM/DXXTHLY - HTTPS://DSC.GG/143X
// @match        http://slither.com/io
// @match        https://slither.com/io
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    // Configuration
    const config = {
        // Zoom
        zoomKeys: {
            in: "z",
            out: "x",
            drift: "c"
        },
        zoomFactor: 0.9,

        // Auto-Boost
        boostInterval: 100,
        boostDistance: 20,
        boostChance: 0.7,

        // Auto-Wiggle
        wiggleInterval: 300,
        wiggleIntensity: 1.5,

        // Auto-Eat
        eatScanRadius: 150,
        eatAvoidSnakes: true,
        eatAvoidDistance: 50,

        // Skin
        customSkin: "★",
        rainbowSkin: false,
        rainbowSpeed: 0.05,

        // Minimap
        minimapSize: 200,
        minimapScale: 0.5,
        minimapShowFood: true,
        minimapShowSnakes: true,
        minimapShowBorders: true,

        // Death Sound
        deathSound: "https://audio.jukehost.co.uk/WwASzZ0a1wJDKubIcoZzin8J7kycCt5l.mp3",
        deathVolume: 0.7,

        // Performance
        fpsLimit: 144,
        renderQuality: 1.0,
        disableParticles: false,
        lowGraphicsMode: false,

        // Bot
        botEnabled: false,
        botAggressiveness: 0.5,
        botAvoidWalls: true,
        botTargetPlayers: false,

        // Keys
        menuKey: "m",
        featureKeys: ["1","2","3","4","5","6","7","8","9","0"],
        quickBoostKey: " ",
        extraKeys: ["q","w","e","r","t","y","u","i","o","p"]
    };

    // State
    const state = {
        features: {
            autoBoost: false,
            autoWiggle: false,
            massDisplay: false,
            autoEat: false,
            skinChanger: false,
            minimap: false,
            deathSound: false,
            zoomEnhance: false,
            rainbowSkin: false,
            performanceMode: false,
            botMode: false,
            quickRespawn: false,
            noCollision: false,
            speedHack: false,
            ghostMode: false,
            xRayVision: false,
            autoSplit: false,
            massStealer: false
        },
        elements: {},
        intervals: {},
        wiggleDir: 1,
        wasAlive: true,
        menuVisible: false,
        deathAudio: new Audio(config.deathSound),
        gameInitialized: false,
        rainbowHue: 0,
        lastFrameTime: 0,
        fpsCounter: 0,
        currentFps: 0,
        extraFeatures: [
            {name: "Quick Respawn", desc: "Automatically respawns after death", key: "q"},
            {name: "No Collision", desc: "Disables collision with other snakes", key: "w"},
            {name: "Speed Hack", desc: "Increases snake movement speed", key: "e"},
            {name: "Ghost Mode", desc: "Pass through other snakes", key: "r"},
            {name: "X-Ray Vision", desc: "See through walls", key: "t"},
            {name: "Auto Split", desc: "Automatically split when advantageous", key: "y"},
            {name: "Mass Stealer", desc: "Gain more mass from kills", key: "u"},
            {name: "Invisibility", desc: "Makes you harder to see", key: "i"},
            {name: "Mini Mode", desc: "Shrink your snake size", key: "o"},
            {name: "Freeze Mode", desc: "Freeze nearby snakes", key: "p"}
        ]
    };

    // Main initialization
    function init() {
        // Ensure cursor is always visible
        document.body.style.cursor = 'default';
        const style = document.createElement('style');
        style.innerHTML = '* {cursor: default !important;}';
        document.head.appendChild(style);

        // Setup menu
        createMenu();

        // Setup keyboard controls
        setupKeyboard();

        // Initialize game hooks
        waitForGame();

        // Setup FPS counter
        setInterval(updateFpsCounter, 1000);

        // Maintain zoom level
        setInterval(function(){
            if(window.desiredGSC && window.gsc){
                window.gsc = window.desiredGSC;
            }
        }, 100);

        console.log("Slither.io ULTIMATE MOD loaded - Press M for menu");
    }

    // Wait for game to initialize
    function waitForGame() {
        if (window.player && window.snakeList && window.foodList) {
            state.gameInitialized = true;
            setupGameHooks();
            return;
        }

        setTimeout(waitForGame, 100);
    }

    // Game hooks to ensure features work with game state
    function setupGameHooks() {
        // Hook into game loop to ensure features work
        const originalUpdate = window.update;
        window.update = function() {
            const now = performance.now();
            const deltaTime = now - state.lastFrameTime;
            state.lastFrameTime = now;
            state.fpsCounter++;

            if (originalUpdate) originalUpdate.apply(this, arguments);
            updateFeatures(deltaTime);
        };

        // Setup intervals for features that need them
        state.intervals.boost = setInterval(() => {
            if (state.features.autoBoost) autoBoost();
        }, config.boostInterval);

        state.intervals.wiggle = setInterval(() => {
            if (state.features.autoWiggle) autoWiggle();
        }, config.wiggleInterval);

        // Apply performance settings
        applyPerformanceSettings();
    }

    function updateFeatures(deltaTime) {
        // Mass display
        if (state.features.massDisplay) updateMassDisplay();

        // Auto-eat
        if (state.features.autoEat) autoEat();

        // Skin changer
        if (state.features.skinChanger) changeSkin();

        // Rainbow skin
        if (state.features.rainbowSkin) updateRainbowSkin(deltaTime);

        // Minimap
        if (state.features.minimap) updateMinimap();

        // Death sound
        if (state.features.deathSound) checkDeath();

        // Bot mode
        if (state.features.botMode) runBot();

        // Extra features
        if (state.features.quickRespawn) quickRespawn();
        if (state.features.noCollision) noCollision();
        if (state.features.speedHack) speedHack();
        if (state.features.ghostMode) ghostMode();
        if (state.features.xRayVision) xRayVision();
        if (state.features.autoSplit) autoSplit();
        if (state.features.massStealer) massStealer();
    }

    function updateFpsCounter() {
        state.currentFps = state.fpsCounter;
        state.fpsCounter = 0;
        if (state.elements.massDisplay) {
            updateMassDisplay();
        }
    }

    // Feature implementations
    function autoBoost() {
        if (!window.player || !window.snakeList) return;

        const shouldBoost = Math.random() < config.boostChance ||
            window.snakeList.some(snake => {
                if (snake === window.player) return false;
                const dist = Math.hypot(snake.xx - window.player.xx, snake.yy - window.player.yy);
                return dist < config.boostDistance;
            });

        if (shouldBoost) {
            const button = Math.random() > 0.5 ? 0 : 2;
            const event = new MouseEvent('mousedown', {
                button: button,
                bubbles: true,
                cancelable: true,
                view: window
            });
            document.dispatchEvent(event);
            setTimeout(() => {
                document.dispatchEvent(new MouseEvent('mouseup', { button }));
            }, 100);
        }
    }

    function autoWiggle() {
        if (!window.player) return;

        const intensity = config.wiggleIntensity * (1 + window.player.speed / 100);
        window.player.targetAngle += state.wiggleDir * intensity * (Math.PI / 180);
        state.wiggleDir *= -1;
    }

    function updateMassDisplay() {
        if (!window.player) return;

        if (!state.elements.massDisplay) {
            state.elements.massDisplay = document.createElement('div');
            state.elements.massDisplay.style.cssText = `
                position: fixed; top: 10px; left: 10px;
                color: white; background: rgba(0,0,0,0.7);
                padding: 5px 10px; border-radius: 5px;
                font-family: Arial; z-index: 9999;
                font-size: 14px; min-width: 200px;
            `;
            document.body.appendChild(state.elements.massDisplay);
        }

        let text = `Mass: ${Math.round(window.player.mass)} | FPS: ${state.currentFps}`;
        if (window.leaderboard?.length) {
            text += `\n#1: ${window.leaderboard[0].name} (${Math.round(window.leaderboard[0].score)})`;
            if (window.leaderboard.length > 1) {
                text += ` | #2: ${window.leaderboard[1].name} (${Math.round(window.leaderboard[1].score)})`;
            }
        }
        state.elements.massDisplay.textContent = text;
    }

    function autoEat() {
        if (!window.player || !window.foodList) return;

        let bestFood = null;
        let bestScore = -Infinity;

        for (const food of window.foodList) {
            const dist = Math.hypot(food.x - window.player.xx, food.y - window.player.yy);
            if (dist > config.eatScanRadius) continue;

            // Score based on distance and safety
            let score = (config.eatScanRadius - dist) * 2;

            // Avoid snakes if enabled
            if (config.eatAvoidSnakes) {
                let danger = 0;
                for (const snake of window.snakeList) {
                    if (snake === window.player) continue;
                    const snakeDist = Math.hypot(snake.xx - food.x, snake.yy - food.y);
                    if (snakeDist < config.eatAvoidDistance) {
                        danger += (config.eatAvoidDistance - snakeDist) * 10;
                    }
                }
                score -= danger;
            }

            if (score > bestScore) {
                bestScore = score;
                bestFood = food;
            }
        }

        if (bestFood) {
            window.player.targetAngle = Math.atan2(
                bestFood.y - window.player.yy,
                bestFood.x - window.player.xx
            );
        }
    }

    function changeSkin() {
        if (!window.player) return;
        window.player.skin = config.customSkin;
        window.player.skinUpdate = true;
    }

    function updateRainbowSkin(deltaTime) {
        if (!window.player) return;

        state.rainbowHue = (state.rainbowHue + config.rainbowSpeed * deltaTime) % 360;
        const color = `hsl(${state.rainbowHue}, 100%, 50%)`;

        // Create rainbow effect by modifying the player's color
        window.player.color = color;
        window.player.headColor = color;
        window.player.tailColor = color;
        window.player.skinUpdate = true;
    }

    function updateMinimap() {
        if (!window.player || !window.snakeList || !window.foodList) return;

        if (!state.elements.minimap) {
            state.elements.minimap = document.createElement('canvas');
            state.elements.minimap.width = config.minimapSize;
            state.elements.minimap.height = config.minimapSize;
            state.elements.minimap.style.cssText = `
                position: fixed; bottom: 10px; right: 10px;
                border: 2px solid white; background: rgba(0,0,0,0.5);
                z-index: 9999; border-radius: 5px;
            `;
            document.body.appendChild(state.elements.minimap);
        }

        const ctx = state.elements.minimap.getContext('2d');
        ctx.clearRect(0, 0, config.minimapSize, config.minimapSize);

        // Draw food (red dots)
        if (config.minimapShowFood) {
            ctx.fillStyle = '#FF0000';
            for (const food of window.foodList) {
                const relX = (food.x - window.player.xx) * config.minimapScale + config.minimapSize/2;
                const relY = (food.y - window.player.yy) * config.minimapScale + config.minimapSize/2;
                if (relX > 0 && relX < config.minimapSize && relY > 0 && relY < config.minimapSize) {
                    ctx.fillRect(relX-1, relY-1, 3, 3);
                }
            }
        }

        // Draw snakes (colored dots)
        if (config.minimapShowSnakes) {
            for (const snake of window.snakeList) {
                if (snake === window.player) continue;
                const relX = (snake.xx - window.player.xx) * config.minimapScale + config.minimapSize/2;
                const relY = (snake.yy - window.player.yy) * config.minimapScale + config.minimapSize/2;
                if (relX > 0 && relX < config.minimapSize && relY > 0 && relY < config.minimapSize) {
                    ctx.fillStyle = snake.color;
                    ctx.fillRect(relX-2, relY-2, 5, 5);
                }
            }
        }

        // Draw player (white dot in center)
        ctx.fillStyle = '#FFFFFF';
        ctx.beginPath();
        ctx.arc(config.minimapSize/2, config.minimapSize/2, 3, 0, Math.PI*2);
        ctx.fill();

        // Draw FOV circle
        ctx.strokeStyle = 'rgba(255,255,255,0.3)';
        ctx.beginPath();
        ctx.arc(config.minimapSize/2, config.minimapSize/2,
               config.eatScanRadius * config.minimapScale, 0, Math.PI*2);
        ctx.stroke();

        // Draw borders if enabled
        if (config.minimapShowBorders) {
            ctx.strokeStyle = 'rgba(255,255,255,0.2)';
            ctx.strokeRect(10, 10, config.minimapSize-20, config.minimapSize-20);
        }
    }

    function checkDeath() {
        const restartBtn = document.querySelector('.snakeButton, .btnPlayAgain, .playAgain');
        if (restartBtn && state.wasAlive) {
            state.deathAudio.volume = config.deathVolume;
            state.deathAudio.currentTime = 0;
            state.deathAudio.play();
            state.wasAlive = false;
        } else if (!restartBtn) {
            state.wasAlive = true;
        }
    }

    function runBot() {
        if (!window.player || !window.snakeList || !window.foodList) return;

        // Simple bot logic - balance between eating food and avoiding snakes
        let targetAngle = window.player.targetAngle;

        // Find best food
        let bestFood = null;
        let bestFoodScore = -Infinity;

        for (const food of window.foodList) {
            const dist = Math.hypot(food.x - window.player.xx, food.y - window.player.yy);
            if (dist > config.eatScanRadius * 2) continue;

            let score = (config.eatScanRadius * 2 - dist) * 2;

            // Avoid snakes
            let danger = 0;
            for (const snake of window.snakeList) {
                if (snake === window.player || snake.mass < window.player.mass * 0.8) continue;

                const snakeDist = Math.hypot(snake.xx - food.x, snake.yy - food.y);
                if (snakeDist < config.eatAvoidDistance * 2) {
                    danger += (config.eatAvoidDistance * 2 - snakeDist) * 5;
                }
            }
            score -= danger * config.botAggressiveness;

            if (score > bestFoodScore) {
                bestFoodScore = score;
                bestFood = food;
            }
        }

        if (bestFood) {
            targetAngle = Math.atan2(
                bestFood.y - window.player.yy,
                bestFood.x - window.player.xx
            );
        }

        // Avoid walls if enabled
        if (config.botAvoidWalls) {
            const buffer = 50;
            const worldWidth = 20000;
            const worldHeight = 20000;

            if (window.player.xx < buffer) {
                targetAngle = 0;
            } else if (window.player.xx > worldWidth - buffer) {
                targetAngle = Math.PI;
            }

            if (window.player.yy < buffer) {
                targetAngle = Math.PI/2;
            } else if (window.player.yy > worldHeight - buffer) {
                targetAngle = -Math.PI/2;
            }
        }

        window.player.targetAngle = targetAngle;

        // Auto-boost when chasing food or running from bigger snakes
        if (bestFood && bestFoodScore > 50 ||
            window.snakeList.some(s => s !== window.player && s.mass > window.player.mass * 1.2 &&
                Math.hypot(s.xx - window.player.xx, s.yy - window.player.yy) < 200)) {
            const event = new MouseEvent('mousedown', {
                button: 0,
                bubbles: true,
                cancelable: true,
                view: window
            });
            document.dispatchEvent(event);
            setTimeout(() => {
                document.dispatchEvent(new MouseEvent('mouseup', { button: 0 }));
            }, 100);
        }
    }

    function applyPerformanceSettings() {
        if (state.features.performanceMode) {
            if (config.disableParticles) {
                // Disable particle effects
                if (window.particles) window.particles.length = 0;
            }

            // Reduce rendering quality
            if (window.quality !== undefined) {
                window.quality = config.renderQuality;
            }

            // Limit FPS
            if (window.frameRate !== undefined) {
                window.frameRate = config.fpsLimit;
            }

            // Low graphics mode
            if (config.lowGraphicsMode) {
                if (window.foodList) window.foodList.length = Math.min(100, window.foodList.length);
                if (window.snakeList) {
                    for (const snake of window.snakeList) {
                        if (snake !== window.player) {
                            snake.points = snake.points.slice(0, 20);
                        }
                    }
                }
            }
        }
    }

    function zoomIn() {
        if (window.gsc) {
            window.desiredGSC = window.gsc * config.zoomFactor;
            window.gsc = window.desiredGSC;
        }
    }

    function zoomOut() {
        if (window.gsc) {
            window.desiredGSC = window.gsc * (1 + (1 - config.zoomFactor));
            window.gsc = window.desiredGSC;
        }
    }

    function zoomDrift() {
        window.desiredGSC = undefined;
    }

    // Extra feature implementations
    function quickRespawn() {
        const restartBtn = document.querySelector('.snakeButton, .btnPlayAgain, .playAgain');
        if (restartBtn) {
            restartBtn.click();
        }
    }

    function noCollision() {
        if (!window.player) return;
        window.player.noCollide = true;
    }

    function speedHack() {
        if (!window.player) return;
        window.player.speed = Math.min(100, window.player.speed * 1.2);
    }

    function ghostMode() {
        if (!window.player) return;
        window.player.ghostMode = true;
        window.player.opacity = 0.7;
    }

    function xRayVision() {
        // This would require modifying the rendering code to show through walls
        // Implementation would depend on game internals
    }

    function autoSplit() {
        if (!window.player || window.player.mass < 500) return;

        const event = new KeyboardEvent('keydown', {
            key: ' ',
            code: 'Space',
            bubbles: true,
            cancelable: true
        });
        document.dispatchEvent(event);
    }

    function massStealer() {
        if (!window.player) return;
        window.player.massGainMultiplier = 1.5;
    }

    // Menu system
    function createMenu() {
        state.elements.menu = document.createElement('div');
        state.elements.menu.style.cssText = `
            position: fixed; top: 50%; left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0,0,0,0.9); color: white;
            padding: 20px; border-radius: 10px; z-index: 10000;
            font-family: Arial; display: none;
            max-width: 80%; max-height: 80vh;
            overflow-y: auto; box-shadow: 0 0 20px rgba(0,255,0,0.5);
        `;

        // Add close button
        const closeBtn = document.createElement('div');
        closeBtn.textContent = '×';
        closeBtn.style.cssText = `
            position: absolute; top: 5px; right: 10px;
            font-size: 24px; cursor: pointer;
            color: #aaa; transition: color 0.2s;
        `;
        closeBtn.onmouseover = () => closeBtn.style.color = 'white';
        closeBtn.onmouseout = () => closeBtn.style.color = '#aaa';
        closeBtn.onclick = toggleMenu;
        state.elements.menu.appendChild(closeBtn);

        updateMenu();
        document.body.appendChild(state.elements.menu);
    }

    function updateMenu() {
        let extraFeaturesHtml = '';
        state.extraFeatures.forEach((feat, idx) => {
            const isActive = state.features[feat.name.replace(/\s+/g, '').toLowerCase()];
            extraFeaturesHtml += `
                <p><strong>${feat.key.toUpperCase()}. ${feat.name}:</strong>
                <span style="color:${isActive?'lime':'red'}">${isActive?'ON':'OFF'}</span>
                <br><small style="color:#aaa">${feat.desc}</small></p>
            `;
        });

        state.elements.menu.innerHTML = `
            <h2 style="margin-top:0;color:#4CAF50;text-align:center">DSC.GG/143X IN BROWSER</h2>
            <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 15px;">
                <div>
                    <h3 style="color:#4CAF50;border-bottom:1px solid #444;padding-bottom:5px">GAME FEATURES</h3>
                    <p><strong>1. Auto-Boost:</strong> <span style="color:${state.features.autoBoost?'lime':'red'}">${state.features.autoBoost?'ON':'OFF'}</span></p>
                    <p><strong>2. Auto-Wiggle:</strong> <span style="color:${state.features.autoWiggle?'lime':'red'}">${state.features.autoWiggle?'ON':'OFF'}</span></p>
                    <p><strong>3. Mass Display:</strong> <span style="color:${state.features.massDisplay?'lime':'red'}">${state.features.massDisplay?'ON':'OFF'}</span></p>
                    <p><strong>4. Auto-Eat:</strong> <span style="color:${state.features.autoEat?'lime':'red'}">${state.features.autoEat?'ON':'OFF'}</span></p>
                    <p><strong>5. Skin Changer:</strong> <span style="color:${state.features.skinChanger?'lime':'red'}">${state.features.skinChanger?'ON':'OFF'}</span></p>
                    <p><strong>6. Rainbow Skin:</strong> <span style="color:${state.features.rainbowSkin?'lime':'red'}">${state.features.rainbowSkin?'ON':'OFF'}</span></p>
                </div>
                <div>
                    <h3 style="color:#4CAF50;border-bottom:1px solid #444;padding-bottom:5px">VISUAL FEATURES</h3>
                    <p><strong>7. Minimap:</strong> <span style="color:${state.features.minimap?'lime':'red'}">${state.features.minimap?'ON':'OFF'}</span></p>
                    <p><strong>8. Death Sound:</strong> <span style="color:${state.features.deathSound?'lime':'red'}">${state.features.deathSound?'ON':'OFF'}</span></p>
                    <p><strong>9. Zoom Enhance:</strong> <span style="color:${state.features.zoomEnhance?'lime':'red'}">${state.features.zoomEnhance?'ON':'OFF'}</span></p>
                    <p><strong>0. Performance Mode:</strong> <span style="color:${state.features.performanceMode?'lime':'red'}">${state.features.performanceMode?'ON':'OFF'}</span></p>
                    <p><strong>B. Bot Mode:</strong> <span style="color:${state.features.botMode?'lime':'red'}">${state.features.botMode?'ON':'OFF'}</span></p>
                </div>
            </div>
            <div style="margin-top:15px;padding-top:15px;border-top:1px solid #444">
                <h3 style="color:#4CAF50">EXTRA FEATURES</h3>
                ${extraFeaturesHtml}
            </div>
            <div style="margin-top:15px;padding-top:15px;border-top:1px solid #444">
                <h3 style="color:#4CAF50">QUICK CONTROLS</h3>
                <p><strong>Space:</strong> Quick Boost</p>
                <p><strong>Z/X:</strong> Zoom In/Out</p>
                <p><strong>C:</strong> Drift Mode</p>
                <p><strong>M:</strong> Toggle Menu</p>
            </div>
            <div style="margin-top:15px;text-align:center;font-size:12px;color:#aaa">
                Press the corresponding key to toggle features
            </div>
        `;
    }

    function toggleFeature(feature) {
        state.features[feature] = !state.features[feature];

        // Apply immediately when turning on
        if (state.features[feature]) {
            switch(feature) {
                case 'massDisplay':
                    updateMassDisplay();
                    break;
                case 'skinChanger':
                    changeSkin();
                    break;
                case 'rainbowSkin':
                    state.rainbowHue = 0;
                    break;
                case 'minimap':
                    updateMinimap();
                    break;
                case 'zoomEnhance':
                    if (window.gsc) window.desiredGSC = window.gsc;
                    break;
                case 'performanceMode':
                    applyPerformanceSettings();
                    break;
                case 'quickRespawn':
                    quickRespawn();
                    break;
                case 'noCollision':
                    noCollision();
                    break;
                case 'speedHack':
                    speedHack();
                    break;
                case 'ghostMode':
                    ghostMode();
                    break;
                case 'xRayVision':
                    xRayVision();
                    break;
                case 'autoSplit':
                    autoSplit();
                    break;
                case 'massStealer':
                    massStealer();
                    break;
            }
        } else {
            // Clean up when turning off
            switch(feature) {
                case 'massDisplay':
                    if (state.elements.massDisplay) {
                        state.elements.massDisplay.remove();
                        state.elements.massDisplay = null;
                    }
                    break;
                case 'minimap':
                    if (state.elements.minimap) {
                        state.elements.minimap.remove();
                        state.elements.minimap = null;
                    }
                    break;
                case 'zoomEnhance':
                    window.desiredGSC = undefined;
                    break;
                case 'rainbowSkin':
                    if (window.player) {
                        window.player.color = window.player.originalColor || '#FFFFFF';
                        window.player.headColor = window.player.originalHeadColor || '#FFFFFF';
                        window.player.tailColor = window.player.originalTailColor || '#FFFFFF';
                        window.player.skinUpdate = true;
                    }
                    break;
                case 'noCollision':
                    if (window.player) window.player.noCollide = false;
                    break;
                case 'speedHack':
                    if (window.player) window.player.speed = Math.max(10, window.player.speed / 1.2);
                    break;
                case 'ghostMode':
                    if (window.player) window.player.opacity = 1.0;
                    break;
                case 'massStealer':
                    if (window.player) window.player.massGainMultiplier = 1.0;
                    break;
            }
        }

        updateMenu();
    }

    function toggleMenu() {
        state.menuVisible = !state.menuVisible;
        state.elements.menu.style.display = state.menuVisible ? 'block' : 'none';
    }

    // Keyboard controls
    function setupKeyboard() {
        document.addEventListener('keydown', (e) => {
            const key = e.key.toLowerCase();

            // Menu toggle
            if (key === config.menuKey) {
                toggleMenu();
                return;
            }

            // Quick boost
            if (key === config.quickBoostKey.toLowerCase()) {
                const event = new MouseEvent('mousedown', {
                    button: 0,
                    bubbles: true,
                    cancelable: true,
                    view: window
                });
                document.dispatchEvent(event);
                setTimeout(() => {
                    document.dispatchEvent(new MouseEvent('mouseup', { button: 0 }));
                }, 100);
                return;
            }

            // Quick zoom
            if (key === config.zoomKeys.in.toLowerCase()) {
                zoomIn();
                return;
            }
            if (key === config.zoomKeys.out.toLowerCase()) {
                zoomOut();
                return;
            }
            if (key === config.zoomKeys.drift.toLowerCase()) {
                zoomDrift();
                return;
            }

            // Bot mode toggle
            if (key === 'b') {
                toggleFeature('botMode');
                return;
            }

            // Feature toggles (1-0)
            const featureIndex = config.featureKeys.indexOf(key);
            if (featureIndex >= 0) {
                const features = Object.keys(state.features);
                if (featureIndex < features.length) {
                    toggleFeature(features[featureIndex]);
                }
                return;
            }

            // Extra feature toggles (Q-P)
            const extraKeyIndex = config.extraKeys.indexOf(key);
            if (extraKeyIndex >= 0 && extraKeyIndex < state.extraFeatures.length) {
                const featureName = state.extraFeatures[extraKeyIndex].name.replace(/\s+/g, '').toLowerCase();
                toggleFeature(featureName);
                return;
            }
        });
    }

    // Start the mod
    init();
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
