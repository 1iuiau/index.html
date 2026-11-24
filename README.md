<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>霓虹打磚塊 | NEON BREAKOUT</title>
    <!-- 引入 Google Fonts 復古與現代繁體中文字型 -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@500;700;900&family=Press+Start+2P&display=swap" rel="stylesheet">
    <!-- 引入 Tone.js 產生復古 8-bit 音效 (無需外部音檔) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
    <style>
        :root {
            --bg-color: #070710;
            --cabinet-bg: #111122;
            --neon-cyan: #00f3ff;
            --neon-pink: #ff007f;
            --neon-yellow: #ffe600;
            --neon-green: #00ff66;
            --neon-purple: #b537f2;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
            -webkit-user-select: none;
            touch-action: none; /* 防止手機端下拉刷新或頁面捲動 */
        }

        body {
            background-color: var(--bg-color);
            background-image: 
                radial-gradient(circle at 50% 0%, rgba(181, 55, 242, 0.15) 0%, transparent 60%),
                radial-gradient(circle at 50% 100%, rgba(0, 243, 255, 0.15) 0%, transparent 60%);
            color: #ffffff;
            font-family: 'Noto Sans TC', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            min-height: 100dvh;
            overflow: hidden;
            padding: 12px;
        }

        /* 遊戲主容器 - 響應式比例 */
        .game-wrapper {
            position: relative;
            width: 100%;
            max-width: 480px;
            height: 85vh;
            height: 85dvh;
            max-height: 760px;
            background: var(--cabinet-bg);
            border: 3px solid var(--neon-cyan);
            border-radius: 16px;
            box-shadow: 
                0 0 20px rgba(0, 243, 255, 0.3),
                inset 0 0 30px rgba(0, 0, 0, 0.8);
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        /* 復古 CRT 掃描線效果 */
        .crt-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(
                rgba(18, 16, 16, 0) 50%, 
                rgba(0, 0, 0, 0.25) 50%
            );
            background-size: 100% 4px;
            pointer-events: none;
            z-index: 10;
        }

        /* 頂部資訊欄 (分數、關卡、生命) */
        .hud-bar {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 12px 16px;
            background: rgba(7, 7, 16, 0.9);
            border-bottom: 2px solid rgba(0, 243, 255, 0.3);
            z-index: 5;
        }

        .hud-item {
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .hud-label {
            font-size: 0.7rem;
            color: #8a8ab5;
            letter-spacing: 1px;
            margin-bottom: 2px;
            font-weight: 700;
        }

        .hud-value {
            font-family: 'Press Start 2P', 'Noto Sans TC', monospace;
            font-size: 0.95rem;
            color: var(--neon-cyan);
            text-shadow: 0 0 8px rgba(0, 243, 255, 0.6);
        }

        .hud-value.level {
            color: var(--neon-yellow);
            text-shadow: 0 0 8px rgba(255, 230, 0, 0.6);
        }

        .lives-container {
            display: flex;
            gap: 4px;
            font-size: 1rem;
            color: var(--neon-pink);
            text-shadow: 0 0 8px rgba(255, 0, 127, 0.8);
        }

        /* 音效開關按鈕 */
        .sound-btn {
            background: transparent;
            border: 1px solid rgba(255, 255, 255, 0.2);
            color: #fff;
            border-radius: 8px;
            width: 34px;
            height: 34px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            font-size: 1.1rem;
            transition: all 0.2s;
        }

        .sound-btn:hover {
            border-color: var(--neon-cyan);
            box-shadow: 0 0 8px rgba(0, 243, 255, 0.4);
        }

        /* 畫布容器 */
        .canvas-container {
            position: relative;
            flex: 1;
            width: 100%;
            height: 100%;
            overflow: hidden;
        }

        canvas {
            display: block;
            width: 100%;
            height: 100%;
        }

        /* 覆蓋層 (開始畫面 / 關卡過渡 / 遊戲結束) */
        .overlay-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(7, 7, 16, 0.85);
            backdrop-filter: blur(4px);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 24px;
            text-align: center;
            z-index: 20;
            transition: opacity 0.3s ease;
        }

        .overlay-screen.hidden {
            opacity: 0;
            pointer-events: none;
        }

        .game-title {
            font-size: 2.2rem;
            font-weight: 900;
            letter-spacing: 2px;
            margin-bottom: 6px;
            background: linear-gradient(180deg, #ffffff 0%, var(--neon-cyan) 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            filter: drop-shadow(0 0 12px rgba(0, 243, 255, 0.6));
        }

        .game-subtitle {
            font-family: 'Press Start 2P', monospace;
            font-size: 0.75rem;
            color: var(--neon-pink);
            letter-spacing: 2px;
            margin-bottom: 24px;
            text-shadow: 0 0 10px rgba(255, 0, 127, 0.8);
        }

        .instructions-box {
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid rgba(0, 243, 255, 0.25);
            border-radius: 12px;
            padding: 16px;
            margin-bottom: 24px;
            width: 100%;
            max-width: 340px;
            font-size: 0.85rem;
            line-height: 1.6;
            color: #d0d0f0;
        }

        .instructions-box p {
            margin-bottom: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
        }

        .instructions-box p:last-child {
            margin-bottom: 0;
        }

        .badge {
            background: rgba(0, 243, 255, 0.15);
            border: 1px solid var(--neon-cyan);
            color: var(--neon-cyan);
            padding: 2px 8px;
            border-radius: 6px;
            font-size: 0.75rem;
            font-weight: 700;
        }

        /* 道具圖例 */
        .powerup-legend {
            display: flex;
            justify-content: center;
            gap: 12px;
            margin-top: 10px;
            padding-top: 10px;
            border-top: 1px dashed rgba(255, 255, 255, 0.15);
            font-size: 0.75rem;
        }

        .legend-item {
            display: flex;
            align-items: center;
            gap: 4px;
        }

        .dot {
            width: 10px;
            height: 10px;
            border-radius: 50%;
            display: inline-block;
        }

        /* 復古街機按鈕 */
        .arcade-btn {
            background: linear-gradient(135deg, var(--neon-pink) 0%, #b537f2 100%);
            color: #fff;
            border: none;
            border-radius: 50px;
            padding: 14px 36px;
            font-size: 1.1rem;
            font-weight: 900;
            letter-spacing: 2px;
            cursor: pointer;
            box-shadow: 
                0 0 20px rgba(255, 0, 127, 0.6),
                inset 0 2px 4px rgba(255, 255, 255, 0.4);
            transition: transform 0.15s, box-shadow 0.15s;
            animation: pulseGlow 2s infinite;
        }

        .arcade-btn:hover {
            transform: scale(1.05);
            box-shadow: 0 0 28px rgba(255, 0, 127, 0.9);
        }

        .arcade-btn:active {
            transform: scale(0.96);
        }

        @keyframes pulseGlow {
            0%, 100% { box-shadow: 0 0 15px rgba(255, 0, 127, 0.6); }
            50% { box-shadow: 0 0 28px rgba(255, 0, 127, 0.95); }
        }

        /* 結算分數看板 */
        .score-summary {
            margin: 16px 0 24px;
            padding: 16px 24px;
            background: rgba(0, 0, 0, 0.5);
            border: 1px solid var(--neon-yellow);
            border-radius: 12px;
            width: 100%;
            max-width: 300px;
        }

        .final-score-label {
            font-size: 0.85rem;
            color: #aaa;
            margin-bottom: 4px;
        }

        .final-score-value {
            font-family: 'Press Start 2P', monospace;
            font-size: 1.6rem;
            color: var(--neon-yellow);
            text-shadow: 0 0 12px rgba(255, 230, 0, 0.7);
            margin-bottom: 10px;
        }

        .high-score-text {
            font-size: 0.8rem;
            color: var(--neon-green);
        }

        /* 底部觸控提示條 */
        .footer-hint {
            margin-top: 8px;
            font-size: 0.75rem;
            color: rgba(255, 255, 255, 0.4);
            text-align: center;
        }
    </style>
</head>
<body>

    <div class="game-wrapper" id="gameWrapper">
        <!-- CRT 掃描線濾鏡 -->
        <div class="crt-overlay"></div>

        <!-- 頂部狀態列 -->
        <div class="hud-bar">
            <div class="hud-item">
                <span class="hud-label">分數 SCORE</span>
                <span class="hud-value" id="scoreDisplay">0</span>
            </div>
            <div class="hud-item">
                <span class="hud-label">關卡 LEVEL</span>
                <span class="hud-value level" id="levelDisplay">1</span>
            </div>
            <div class="hud-item">
                <span class="hud-label">生命 LIVES</span>
                <div class="lives-container" id="livesDisplay">♥♥♥</div>
            </div>
            <button class="sound-btn" id="soundToggleBtn" title="切換音效" aria-label="切換音效">🔊</button>
        </div>

        <!-- 遊戲畫布區 -->
        <div class="canvas-container" id="canvasContainer">
            <canvas id="gameCanvas"></canvas>

            <!-- 開始畫面 -->
            <div class="overlay-screen" id="startScreen">
                <h1 class="game-title">霓虹打磚塊</h1>
                <div class="game-subtitle">NEON BREAKOUT</div>
                
                <div class="instructions-box">
                    <p><span class="badge">電腦</span> 左右方向鍵 或 A / D 移動</p>
                    <p><span class="badge">手機</span> 螢幕任意處左右滑動拖曳</p>
                    <div class="powerup-legend">
                        <div class="legend-item"><span class="dot" style="background:#00ff66; box-shadow:0 0 6px #00ff66;"></span>加寬檔板</div>
                        <div class="legend-item"><span class="dot" style="background:#00f3ff; box-shadow:0 0 6px #00f3ff;"></span>多重球</div>
                        <div class="legend-item"><span class="dot" style="background:#ff007f; box-shadow:0 0 6px #ff007f;"></span>+1 生命</div>
                    </div>
                </div>

                <button class="arcade-btn" id="startBtn">開始遊戲</button>
            </div>

            <!-- 遊戲結束畫面 -->
            <div class="overlay-screen hidden" id="gameOverScreen">
                <h2 class="game-title" id="gameOverTitle">遊戲結束</h2>
                <div class="game-subtitle">GAME OVER</div>

                <div class="score-summary">
                    <div class="final-score-label">最終得分</div>
                    <div class="final-score-value" id="finalScoreDisplay">0</div>
                    <div class="high-score-text" id="highScoreDisplay">最高紀錄: 0</div>
                </div>

                <button class="arcade-btn" id="restartBtn">再玩一次</button>
            </div>
        </div>
    </div>

    <div class="footer-hint">支援桌機鍵盤與手機觸控 • 擊碎所有霓虹磚塊</div>

    <script>
        /**
         * 霓虹打磚塊 (NEON BREAKOUT) - 遊戲核心邏輯
         * 包含響應式 Canvas、粒子物理系統、道具系統、Tone.js 復古合成器音效
         */

        // --- 1. 音效系統 (Tone.js 復古 8-bit 合成音效) ---
        class SoundEffects {
            constructor() {
                this.enabled = true;
                this.initialized = false;
                this.synth = null;
            }

            async init() {
                if (this.initialized) return;
                try {
                    await Tone.start();
                    this.synth = new Tone.PolySynth(Tone.Synth, {
                        oscillator: { type: "square" },
                        envelope: { attack: 0.005, decay: 0.08, sustain: 0.01, release: 0.08 }
                    }).toDestination();
                    this.synth.volume.value = -10;
                    this.initialized = true;
                } catch (e) {
                    console.warn("音效初始化略過:", e);
                }
            }

            playPaddleHit() {
                if (!this.enabled || !this.initialized) return;
                this.synth.triggerAttackRelease("C5", "32n");
            }

            playWallHit() {
                if (!this.enabled || !this.initialized) return;
                this.synth.triggerAttackRelease("G4", "32n");
            }

            playBrickHit(combo = 1) {
                if (!this.enabled || !this.initialized) return;
                const notes = ["C5", "D5", "E5", "G5", "A5", "C6"];
                const note = notes[Math.min(combo - 1, notes.length - 1)];
                this.synth.triggerAttackRelease(note, "16n");
            }

            playPowerup() {
                if (!this.enabled || !this.initialized) return;
                const now = Tone.now();
                this.synth.triggerAttackRelease("E5", "32n", now);
                this.synth.triggerAttackRelease("G5", "32n", now + 0.06);
                this.synth.triggerAttackRelease("C6", "16n", now + 0.12);
            }

            playLoseLife() {
                if (!this.enabled || !this.initialized) return;
                const now = Tone.now();
                this.synth.triggerAttackRelease("E4", "16n", now);
                this.synth.triggerAttackRelease("C4", "8n", now + 0.1);
            }

            playLevelUp() {
                if (!this.enabled || !this.initialized) return;
                const now = Tone.now();
                this.synth.triggerAttackRelease("C5", "16n", now);
                this.synth.triggerAttackRelease("E5", "16n", now + 0.08);
                this.synth.triggerAttackRelease("G5", "16n", now + 0.16);
                this.synth.triggerAttackRelease("C6", "8n", now + 0.24);
            }

            playGameOver() {
                if (!this.enabled || !this.initialized) return;
                const now = Tone.now();
                this.synth.triggerAttackRelease("G4", "8n", now);
                this.synth.triggerAttackRelease("F#4", "8n", now + 0.15);
                this.synth.triggerAttackRelease("F4", "8n", now + 0.30);
                this.synth.triggerAttackRelease("E4", "4n", now + 0.45);
            }
        }

        const sfx = new SoundEffects();

        // --- 2. DOM 元素與 Canvas 初始化 ---
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const canvasContainer = document.getElementById('canvasContainer');

        const scoreDisplay = document.getElementById('scoreDisplay');
        const levelDisplay = document.getElementById('levelDisplay');
        const livesDisplay = document.getElementById('livesDisplay');
        const startScreen = document.getElementById('startScreen');
        const gameOverScreen = document.getElementById('gameOverScreen');
        const finalScoreDisplay = document.getElementById('finalScoreDisplay');
        const highScoreDisplay = document.getElementById('highScoreDisplay');
        const startBtn = document.getElementById('startBtn');
        const restartBtn = document.getElementById('restartBtn');
        const soundToggleBtn = document.getElementById('soundToggleBtn');

        // 遊戲狀態變數
        let gameState = 'START'; // 'START', 'PLAYING', 'LEVEL_TRANSITION', 'GAMEOVER'
        let score = 0;
        let highScore = parseInt(localStorage.getItem('neonBreakoutHighScore')) || 0;
        let level = 1;
        let lives = 3;
        let comboCount = 0;
        let screenShake = 0;
        let transitionTimer = 0;

        // 畫布邏輯尺寸
        let width = 0;
        let height = 0;

        // 遊戲物件陣列
        let paddle = null;
        let balls = [];
        let bricks = [];
        let particles = [];
        let powerups = [];
        let floatingTexts = [];

        // 控制狀態
        const keys = { left: false, right: false };
        let isTouching = false;
        let lastTouchX = null;

        // 霓虹磚塊配色表
        const ROW_COLORS = [
            { fill: '#ff007f', glow: 'rgba(255, 0, 127, 0.7)' },   // 霓虹粉
            { fill: '#b537f2', glow: 'rgba(181, 55, 242, 0.7)' },  // 霓虹紫
            { fill: '#00f3ff', glow: 'rgba(0, 243, 255, 0.7)' },   // 霓虹青
            { fill: '#00ff66', glow: 'rgba(0, 255, 102, 0.7)' },   // 霓虹綠
            { fill: '#ffe600', glow: 'rgba(255, 230, 0, 0.7)' }    // 霓虹黃
        ];

        // --- 3. 響應式畫布尺寸調整 ---
        function resizeCanvas() {
            const rect = canvasContainer.getBoundingClientRect();
            const dpr = window.devicePixelRatio || 1;
            
            const oldWidth = width || rect.width;
            const oldHeight = height || rect.height;

            width = rect.width;
            height = rect.height;

            canvas.width = width * dpr;
            canvas.height = height * dpr;
            ctx.scale(dpr, dpr);

            // 若遊戲正在進行中調整視窗，等比例調整檔板與球位置
            if (paddle && oldWidth > 0) {
                const ratioX = width / oldWidth;
                const ratioY = height / oldHeight;
                paddle.width = Math.min(width * 0.24, 110) * (paddle.wideTimer > 0 ? 1.4 : 1);
                paddle.x = Math.max(0, Math.min(width - paddle.width, paddle.x * ratioX));
                paddle.y = height - 36;

                balls.forEach(b => {
                    b.x *= ratioX;
                    b.y *= ratioY;
                });

                repositionBricks();
            }
        }

        window.addEventListener('resize', resizeCanvas);

        // --- 4. 遊戲實體定義與初始化 ---
        function initPaddle() {
            const baseW = Math.max(75, Math.min(width * 0.24, 105));
            paddle = {
                baseWidth: baseW,
                width: baseW,
                height: 14,
                x: (width - baseW) / 2,
                y: height - 36,
                speed: Math.max(6, width * 0.016),
                wideTimer: 0
            };
        }

        function createBall(x, y, vx, vy) {
            // 隨關卡微幅提升基礎球速
            const baseSpeed = Math.min(5.2 + (level - 1) * 0.45, 9.5);
            const angle = -Math.PI / 2 + (Math.random() * 0.6 - 0.3);
            return {
                x: x || width / 2,
                y: y || paddle.y - 12,
                radius: 7,
                speed: baseSpeed,
                vx: vx !== undefined ? vx : Math.cos(angle) * baseSpeed,
                vy: vy !== undefined ? vy : Math.sin(angle) * baseSpeed,
                trail: []
            };
        }

        function initBricks() {
            bricks = [];
            const cols = 7;
            // 隨關卡增加行數 (最多 7 行)
            const rows = Math.min(4 + Math.floor((level - 1) / 1), 7);
            const padding = 6;
            const offsetTop = 30;
            const offsetLeft = 14;
            const availableWidth = width - offsetLeft * 2 - padding * (cols - 1);
            const brickWidth = availableWidth / cols;
            const brickHeight = 20;

            for (let r = 0; r < rows; r++) {
                const colorObj = ROW_COLORS[r % ROW_COLORS.length];
                for (let c = 0; c < cols; c++) {
                    // 高關卡出現需要撞擊 2 次的裝甲磚塊
                    const isArmored = (level >= 2 && r === 0 && (c % 2 === 0)) || (level >= 4 && r === 1 && (c % 2 === 1));
                    bricks.push({
                        row: r,
                        col: c,
                        x: offsetLeft + c * (brickWidth + padding),
                        y: offsetTop + r * (brickHeight + padding),
                        width: brickWidth,
                        height: brickHeight,
                        hp: isArmored ? 2 : 1,
                        maxHp: isArmored ? 2 : 1,
                        color: colorObj.fill,
                        glow: colorObj.glow
                    });
                }
            }
        }

        function repositionBricks() {
            if (!bricks.length) return;
            const cols = 7;
            const padding = 6;
            const offsetTop = 30;
            const offsetLeft = 14;
            const availableWidth = width - offsetLeft * 2 - padding * (cols - 1);
            const brickWidth = availableWidth / cols;
            const brickHeight = 20;

            bricks.forEach(b => {
                b.x = offsetLeft + b.col * (brickWidth + padding);
                b.y = offsetTop + b.row * (brickHeight + padding);
                b.width = brickWidth;
                b.height = brickHeight;
            });
        }

        // --- 5. 特效與道具系統 ---
        function spawnParticles(x, y, color, count = 14) {
            for (let i = 0; i < count; i++) {
                const angle = Math.random() * Math.PI * 2;
                const speed = 1.5 + Math.random() * 4;
                particles.push({
                    x,
                    y,
                    vx: Math.cos(angle) * speed,
                    vy: Math.sin(angle) * speed,
                    size: 2 + Math.random() * 3.5,
                    color,
                    alpha: 1,
                    decay: 0.02 + Math.random() * 0.025
                });
            }
        }

        function spawnFloatingText(x, y, text, color) {
            floatingTexts.push({
                x,
                y,
                text,
                color,
                alpha: 1,
                vy: -1.2
            });
        }

        function maybeSpawnPowerup(x, y) {
            // 22% 機率掉落道具
            if (Math.random() > 0.22) return;
            const types = [
                { type: 'WIDE', label: 'W', color: '#00ff66', name: '加寬檔板!' },
                { type: 'MULTI', label: 'M', color: '#00f3ff', name: '多重霓虹球!' },
                { type: 'LIFE', label: '♥', color: '#ff007f', name: '生命 +1!' }
            ];
            // 生命已滿時降低愛心掉落權重
            const chosen = (lives >= 5 && Math.random() < 0.7) 
                ? types[Math.floor(Math.random() * 2)] 
                : types[Math.floor(Math.random() * types.length)];

            powerups.push({
                x,
                y,
                radius: 12,
                vy: 2.6,
                ...chosen
            });
        }

        function applyPowerup(p) {
            sfx.playPowerup();
            score += 50;
            updateHUD();
            spawnFloatingText(p.x, p.y - 10, p.name, p.color);

            if (p.type === 'WIDE') {
                paddle.wideTimer = 600; // 約 10 秒 (60fps)
                paddle.width = paddle.baseWidth * 1.45;
                paddle.x = Math.max(0, Math.min(width - paddle.width, paddle.x - (paddle.width - paddle.baseWidth) / 2));
            } else if (p.type === 'MULTI') {
                if (balls.length > 0 && balls.length < 5) {
                    const refBall = balls[0];
                    const speed = refBall.speed;
                    balls.push(createBall(refBall.x, refBall.y, Math.cos(-Math.PI * 0.35) * speed, Math.sin(-Math.PI * 0.35) * speed));
                    balls.push(createBall(refBall.x, refBall.y, Math.cos(-Math.PI * 0.65) * speed, Math.sin(-Math.PI * 0.65) * speed));
                }
            } else if (p.type === 'LIFE') {
                if (lives < 5) {
                    lives++;
                    updateHUD();
                } else {
                    score += 150;
                    updateHUD();
                }
            }
        }

        // --- 6. HUD 與遊戲流程控制 ---
        function updateHUD() {
            scoreDisplay.textContent = score;
            levelDisplay.textContent = level;
            livesDisplay.textContent = '♥'.repeat(Math.max(0, lives));
        }

        async function startNewGame() {
            await sfx.init();
            score = 0;
            level = 1;
            lives = 3;
            comboCount = 0;
            particles = [];
            powerups = [];
            floatingTexts = [];

            resizeCanvas();
            initPaddle();
            initBricks();
            balls = [createBall()];
            
            updateHUD();
            startScreen.classList.add('hidden');
            gameOverScreen.classList.add('hidden');
            gameState = 'PLAYING';
        }

        function nextLevel() {
            level++;
            sfx.playLevelUp();
            updateHUD();
            powerups = [];
            particles = [];
            initPaddle();
            initBricks();
            balls = [createBall()];
            gameState = 'LEVEL_TRANSITION';
            transitionTimer = 90; // 1.5 秒關卡提示
        }

        function triggerGameOver() {
            gameState = 'GAMEOVER';
            sfx.playGameOver();
            if (score > highScore) {
                highScore = score;
                localStorage.setItem('neonBreakoutHighScore', highScore);
            }
            finalScoreDisplay.textContent = score;
            highScoreDisplay.textContent = `最高紀錄: ${highScore}`;
            gameOverScreen.classList.remove('hidden');
        }

        // --- 7. 輸入監聽 (鍵盤 + 手機觸控) ---
        window.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.left = true;
            if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.right = true;
        });

        window.addEventListener('keyup', (e) => {
            if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.left = false;
            if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.right = false;
        });

        // 手機觸控拖曳支援 (相對位移拖曳，手指不會遮擋檔板視線)
        canvas.addEventListener('touchstart', (e) => {
            if (gameState !== 'PLAYING') return;
            isTouching = true;
            lastTouchX = e.touches[0].clientX;
        }, { passive: true });

        canvas.addEventListener('touchmove', (e) => {
            if (!isTouching || gameState !== 'PLAYING' || !paddle) return;
            const currentX = e.touches[0].clientX;
            const deltaX = currentX - lastTouchX;
            lastTouchX = currentX;

            paddle.x += deltaX * 1.15; // 靈敏度微調讓單手拇指操作更順暢
            paddle.x = Math.max(0, Math.min(width - paddle.width, paddle.x));
        }, { passive: true });

        canvas.addEventListener('touchend', () => {
            isTouching = false;
            lastTouchX = null;
        });

        // 滑鼠拖曳/移動支援 (方便桌機用戶直接用滑鼠測試)
        let isMouseDown = false;
        canvas.addEventListener('mousedown', (e) => {
            isMouseDown = true;
            lastTouchX = e.clientX;
        });
        window.addEventListener('mousemove', (e) => {
            if (!isMouseDown || gameState !== 'PLAYING' || !paddle) return;
            const deltaX = e.clientX - lastTouchX;
            lastTouchX = e.clientX;
            paddle.x += deltaX;
            paddle.x = Math.max(0, Math.min(width - paddle.width, paddle.x));
        });
        window.addEventListener('mouseup', () => {
            isMouseDown = false;
        });

        // 按鈕事件
        startBtn.addEventListener('click', startNewGame);
        restartBtn.addEventListener('click', startNewGame);

        soundToggleBtn.addEventListener('click', () => {
            sfx.enabled = !sfx.enabled;
            soundToggleBtn.textContent = sfx.enabled ? '🔊' : '🔇';
        });

        // --- 8. 遊戲更新與碰撞偵測迴圈 ---
        function update() {
            if (screenShake > 0) screenShake *= 0.85;
            if (screenShake < 0.2) screenShake = 0;

            // 更新背景粒子與浮動文字
            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.x += p.vx;
                p.y += p.vy;
                p.alpha -= p.decay;
                if (p.alpha <= 0) particles.splice(i, 1);
            }

            for (let i = floatingTexts.length - 1; i >= 0; i--) {
                const ft = floatingTexts[i];
                ft.y += ft.vy;
                ft.alpha -= 0.018;
                if (ft.alpha <= 0) floatingTexts.splice(i, 1);
            }

            if (gameState === 'LEVEL_TRANSITION') {
                transitionTimer--;
                if (transitionTimer <= 0) {
                    gameState = 'PLAYING';
                }
                return;
            }

            if (gameState !== 'PLAYING') return;

            // 檔板鍵盤移動
            if (keys.left) paddle.x -= paddle.speed;
            if (keys.right) paddle.x += paddle.speed;
            paddle.x = Math.max(0, Math.min(width - paddle.width, paddle.x));

            // 檔板加寬計時器
            if (paddle.wideTimer > 0) {
                paddle.wideTimer--;
                if (paddle.wideTimer === 0) {
                    const center = paddle.x + paddle.width / 2;
                    paddle.width = paddle.baseWidth;
                    paddle.x = Math.max(0, Math.min(width - paddle.width, center - paddle.width / 2));
                }
            }

            // 更新道具掉落
            for (let i = powerups.length - 1; i >= 0; i--) {
                const p = powerups[i];
                p.y += p.vy;

                // 檢查是否被檔板接住
                if (
                    p.y + p.radius >= paddle.y &&
                    p.y - p.radius <= paddle.y + paddle.height &&
                    p.x + p.radius >= paddle.x &&
                    p.x - p.radius <= paddle.x + paddle.width
                ) {
                    applyPowerup(p);
                    spawnParticles(p.x, p.y, p.color, 12);
                    powerups.splice(i, 1);
                    continue;
                }

                if (p.y - p.radius > height) {
                    powerups.splice(i, 1);
                }
            }

            // 更新所有球體
            for (let bIdx = balls.length - 1; bIdx >= 0; bIdx--) {
                const b = balls[bIdx];

                // 記錄軌跡殘影
                b.trail.push({ x: b.x, y: b.y });
                if (b.trail.length > 8) b.trail.shift();

                b.x += b.vx;
                b.y += b.vy;

                // 左右牆壁反彈
                if (b.x - b.radius < 0) {
                    b.x = b.radius;
                    b.vx = Math.abs(b.vx);
                    sfx.playWallHit();
                } else if (b.x + b.radius > width) {
                    b.x = width - b.radius;
                    b.vx = -Math.abs(b.vx);
                    sfx.playWallHit();
                }

                // 頂部牆壁反彈
                if (b.y - b.radius < 0) {
                    b.y = b.radius;
                    b.vy = Math.abs(b.vy);
                    sfx.playWallHit();
                }

                // 檔板碰撞檢測
                if (
                    b.vy > 0 &&
                    b.y + b.radius >= paddle.y &&
                    b.y - b.radius <= paddle.y + paddle.height &&
                    b.x + b.radius >= paddle.x &&
                    b.x - b.radius <= paddle.x + paddle.width
                ) {
                    // 根據擊中檔板的位置計算反彈角度 (-60度 ~ +60度)
                    const hitPoint = (b.x - (paddle.x + paddle.width / 2)) / (paddle.width / 2);
                    const bounceAngle = hitPoint * (Math.PI / 3);
                    
                    // 每次擊球微幅加速增加刺激感
                    b.speed = Math.min(b.speed + 0.06, 11);
                    b.vx = Math.sin(bounceAngle) * b.speed;
                    b.vy = -Math.cos(bounceAngle) * b.speed;
                    b.y = paddle.y - b.radius;

                    comboCount = 0; // 重置連擊音階
                    sfx.playPaddleHit();
                    spawnParticles(b.x, paddle.y, '#00f3ff', 6);
                }

                // 磚塊碰撞檢測
                for (let i = bricks.length - 1; i >= 0; i--) {
                    const br = bricks[i];
                    if (
                        b.x + b.radius > br.x &&
                        b.x - b.radius < br.x + br.width &&
                        b.y + b.radius > br.y &&
                        b.y - b.radius < br.y + br.height
                    ) {
                        // 判斷碰撞面 (水平或垂直反彈)
                        const overlapLeft = (b.x + b.radius) - br.x;
                        const overlapRight = (br.x + br.width) - (b.x - b.radius);
                        const overlapTop = (b.y + b.radius) - br.y;
                        const overlapBottom = (br.y + br.height) - (b.y - b.radius);

                        const minOverlapX = Math.min(overlapLeft, overlapRight);
                        const minOverlapY = Math.min(overlapTop, overlapBottom);

                        if (minOverlapX < minOverlapY) {
                            b.vx = -b.vx;
                        } else {
                            b.vy = -b.vy;
                        }

                        br.hp--;
                        comboCount++;
                        sfx.playBrickHit(comboCount);

                        if (br.hp <= 0) {
                            const gained = (10 + (level - 1) * 5) * Math.min(comboCount, 4);
                            score += gained;
                            updateHUD();
                            spawnParticles(br.x + br.width / 2, br.y + br.height / 2, br.color, 16);
                            if (comboCount >= 2) {
                                spawnFloatingText(br.x + br.width / 2, br.y, `+${gained}`, '#ffe600');
                            }
                            maybeSpawnPowerup(br.x + br.width / 2, br.y + br.height / 2);
                            bricks.splice(i, 1);
                            screenShake = 3.5;
                        } else {
                            // 裝甲磚塊受損特效
                            spawnParticles(b.x, b.y, '#ffffff', 8);
                            screenShake = 2;
                        }
                        break; // 單幀一顆球只處理一次磚塊碰撞，避免穿透異常
                    }
                }

                // 球掉落底部邊界
                if (b.y - b.radius > height) {
                    balls.splice(bIdx, 1);
                }
            }

            // 檢查是否所有球都掉落
            if (balls.length === 0) {
                lives--;
                updateHUD();
                screenShake = 8;
                if (lives <= 0) {
                    triggerGameOver();
                } else {
                    sfx.playLoseLife();
                    balls = [createBall()];
                }
                return;
            }

            // 檢查是否清空所有磚塊進入下一關
            if (bricks.length === 0) {
                score += 200 * level;
                updateHUD();
                nextLevel();
            }
        }

        // --- 9. 畫面渲染迴圈 ---
        function draw() {
            ctx.save();
            // 畫面震動效果
            if (screenShake > 0) {
                const dx = (Math.random() - 0.5) * screenShake;
                const dy = (Math.random() - 0.5) * screenShake;
                ctx.translate(dx, dy);
            }

            // 清除畫布並繪製復古網格背景
            ctx.fillStyle = '#0d0d1a';
            ctx.fillRect(0, 0, width, height);

            ctx.strokeStyle = 'rgba(0, 243, 255, 0.04)';
            ctx.lineWidth = 1;
            const gridSize = 32;
            for (let x = 0; x < width; x += gridSize) {
                ctx.beginPath();
                ctx.moveTo(x, 0);
                ctx.lineTo(x, height);
                ctx.stroke();
            }
            for (let y = 0; y < height; y += gridSize) {
                ctx.beginPath();
                ctx.moveTo(0, y);
                ctx.lineTo(width, y);
                ctx.stroke();
            }

            // 繪製磚塊
            bricks.forEach(br => {
                ctx.save();
                ctx.shadowBlur = 10;
                ctx.shadowColor = br.glow;
                ctx.fillStyle = br.color;
                
                // 圓角矩形磚塊
                roundRect(ctx, br.x, br.y, br.width, br.height, 4, true, false);

                // 磚塊高光立體感
                ctx.fillStyle = 'rgba(255, 255, 255, 0.25)';
                roundRect(ctx, br.x + 2, br.y + 2, br.width - 4, br.height * 0.35, 2, true, false);

                // 若為 2 HP 裝甲磚塊，繪製白色外框標示
                if (br.hp > 1) {
                    ctx.strokeStyle = '#ffffff';
                    ctx.lineWidth = 2;
                    roundRect(ctx, br.x + 1, br.y + 1, br.width - 2, br.height - 2, 4, false, true);
                }
                ctx.restore();
            });

            // 繪製掉落道具
            powerups.forEach(p => {
                ctx.save();
                ctx.shadowBlur = 12;
                ctx.shadowColor = p.color;
                ctx.fillStyle = '#111122';
                ctx.strokeStyle = p.color;
                ctx.lineWidth = 2.5;

                ctx.beginPath();
                ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
                ctx.fill();
                ctx.stroke();

                ctx.fillStyle = p.color;
                ctx.font = 'bold 12px "Noto Sans TC", sans-serif';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(p.label, p.x, p.y + 1);
                ctx.restore();
            });

            // 繪製檔板
            if (paddle) {
                ctx.save();
                const paddleColor = paddle.wideTimer > 0 ? '#00ff66' : '#00f3ff';
                ctx.shadowBlur = 15;
                ctx.shadowColor = paddleColor;
                ctx.fillStyle = paddleColor;
                roundRect(ctx, paddle.x, paddle.y, paddle.width, paddle.height, 7, true, false);

                // 檔板中央能量核心裝飾
                ctx.fillStyle = '#ffffff';
                roundRect(ctx, paddle.x + paddle.width * 0.25, paddle.y + 4, paddle.width * 0.5, paddle.height - 8, 3, true, false);
                ctx.restore();
            }

            // 繪製球體與光尾殘影
            balls.forEach(b => {
                // 繪製拖尾
                b.trail.forEach((pos, idx) => {
                    const ratio = (idx + 1) / b.trail.length;
                    ctx.beginPath();
                    ctx.arc(pos.x, pos.y, b.radius * ratio * 0.85, 0, Math.PI * 2);
                    ctx.fillStyle = `rgba(255, 255, 255, ${ratio * 0.25})`;
                    ctx.fill();
                });

                // 繪製球本體
                ctx.save();
                ctx.shadowBlur = 14;
                ctx.shadowColor = '#ffffff';
                ctx.fillStyle = '#ffffff';
                ctx.beginPath();
                ctx.arc(b.x, b.y, b.radius, 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();
            });

            // 繪製粒子爆炸特效
            particles.forEach(p => {
                ctx.save();
                ctx.globalAlpha = Math.max(0, p.alpha);
                ctx.fillStyle = p.color;
                ctx.shadowBlur = 8;
                ctx.shadowColor = p.color;
                ctx.fillRect(p.x - p.size / 2, p.y - p.size / 2, p.size, p.size);
                ctx.restore();
            });

            // 繪製浮動得分/道具提示文字
            floatingTexts.forEach(ft => {
                ctx.save();
                ctx.globalAlpha = Math.max(0, ft.alpha);
                ctx.fillStyle = ft.color;
                ctx.font = 'bold 14px "Noto Sans TC", sans-serif';
                ctx.textAlign = 'center';
                ctx.shadowBlur = 6;
                ctx.shadowColor = '#000';
                ctx.fillText(ft.text, ft.x, ft.y);
                ctx.restore();
            });

            // 關卡過渡提示字樣
            if (gameState === 'LEVEL_TRANSITION') {
                ctx.save();
                ctx.fillStyle = 'rgba(7, 7, 16, 0.65)';
                ctx.fillRect(0, height / 2 - 45, width, 90);
                ctx.fillStyle = '#ffe600';
                ctx.shadowBlur = 15;
                ctx.shadowColor = '#ffe600';
                ctx.font = '900 28px "Noto Sans TC", sans-serif';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(`第 ${level} 關`, width / 2, height / 2 - 8);
                ctx.font = 'bold 13px "Noto Sans TC", sans-serif';
                ctx.fillStyle = '#00f3ff';
                ctx.fillText('速度提升！準備接球', width / 2, height / 2 + 22);
                ctx.restore();
            }

            ctx.restore();
        }

        // 輔助函式：繪製圓角矩形
        function roundRect(ctx, x, y, w, h, r, fill, stroke) {
            ctx.beginPath();
            ctx.moveTo(x + r, y);
            ctx.arcTo(x + w, y, x + w, y + h, r);
            ctx.arcTo(x + w, y + h, x, y + h, r);
            ctx.arcTo(x, y + h, x, y, r);
            ctx.arcTo(x, y, x + w, y, r);
            ctx.closePath();
            if (fill) ctx.fill();
            if (stroke) ctx.stroke();
        }

        // 主動畫迴圈
        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // 頁面載入後初始化
        window.onload = function() {
            resizeCanvas();
            initPaddle();
            initBricks();
            gameLoop();
        };
    </script>
</body>
</html>
