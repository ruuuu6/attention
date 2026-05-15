<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>注意力與動作協調訓練器 - 教學專業版</title>
    <script src="https://cdn.tailwindcss.com"></script>
    
    <style>
        :root {
            --bg-dark: #0f172a;
            --success: #10b981;
            --warning: #f59e0b;
            --danger: #f43f5e;
        }

        body {
            background-color: var(--bg-dark);
            color: #f8fafc;
            font-family: 'Inter', "PingFang TC", "Microsoft JhengHei", sans-serif;
            overflow: hidden;
            height: 100vh;
            display: flex;
            flex-direction: column;
        }

        .settings-container {
            transition: transform 0.5s cubic-bezier(0.16, 1, 0.3, 1);
        }
        .settings-collapsed {
            transform: translateY(-100%);
        }

        .instruction-card {
            transition: all 0.3s ease;
            border: 2px solid transparent;
        }
        .instruction-card.active-target {
            border-color: #38bdf8;
            box-shadow: 0 0 20px rgba(56, 189, 248, 0.4);
            transform: scale(1.05);
            background-color: rgba(56, 189, 248, 0.1);
        }

        .game-grid {
            display: grid;
            gap: 12px;
            width: 100%;
            max-width: 500px;
            aspect-ratio: 1 / 1;
            margin: 0 auto;
        }

        .grid-cell {
            background-color: rgba(30, 41, 59, 0.6);
            border-radius: 20px;
            position: relative;
            display: flex;
            align-items: center;
            justify-content: center;
            overflow: hidden;
            box-shadow: inset 0 4px 6px rgba(0, 0, 0, 0.3);
        }

        .pop-item {
            position: absolute;
            width: 85%;
            height: 85%;
            background-color: #1e293b;
            border-radius: 18px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 3rem;
            cursor: pointer;
            transform: scale(0);
            transition: transform 0.15s cubic-bezier(0.34, 1.56, 0.64, 1);
            border: 2px solid rgba(255, 255, 255, 0.1);
        }

        .pop-item.active { transform: scale(1); }

        .hit-success { background-color: rgba(16, 185, 129, 0.2) !important; border-color: #10b981 !important; }
        .hit-error { background-color: rgba(244, 63, 94, 0.2) !important; border-color: #f43f5e !important; }

        #youtubePlayerContainer { position: absolute; left: -1000px; top: -1000px; width: 1px; height: 1px; }
    </style>
</head>
<body>

    <div id="youtubePlayerContainer"><div id="player"></div></div>

    <div id="settingsPanel" class="settings-container fixed top-0 left-0 w-full z-50 p-2">
        <div class="max-w-6xl mx-auto bg-slate-900/95 backdrop-blur-xl p-5 rounded-3xl border border-slate-700 shadow-2xl">
            <div class="grid grid-cols-1 lg:grid-cols-4 gap-6">
                <!-- 標題與模式選擇 -->
                <div class="space-y-3">
                    <h1 class="text-xl font-black text-sky-400">教學動作訓練器</h1>
                    <div>
                        <label class="text-[10px] font-bold text-slate-500 uppercase block mb-2">訓練分級</label>
                        <select id="levelSelect" onchange="changeLevel(this.value)" class="w-full bg-slate-800 border border-slate-600 rounded-xl px-3 py-2 text-sm text-white outline-none">
                            <option value="1">初級：單一指令 (1個動作)</option>
                            <option value="2" selected>中級：二元辨識 (2個動作)</option>
                            <option value="3">高級：多元反應 (3個動作)</option>
                            <option value="4">挑戰：全能專注 (4個動作)</option>
                        </select>
                    </div>
                </div>

                <!-- 參數調整 -->
                <div class="grid grid-cols-2 gap-3 lg:col-span-2">
                    <div class="bg-slate-800/50 p-3 rounded-2xl border border-slate-700">
                        <div class="flex justify-between mb-1">
                            <span class="text-[10px] font-bold text-slate-400">網格大小</span>
                            <span id="gridSizeLabel" class="text-xs text-sky-300">3x3</span>
                        </div>
                        <input type="range" id="gridSlider" min="3" max="5" value="3" oninput="updateGridUI(this.value)" class="w-full h-1 bg-slate-700 rounded-lg appearance-none accent-sky-500">
                    </div>
                    <div class="bg-slate-800/50 p-3 rounded-2xl border border-slate-700">
                        <div class="flex justify-between mb-1">
                            <span class="text-[10px] font-bold text-slate-400">變換速度</span>
                            <span id="speedLabel" class="text-xs text-rose-400">適中</span>
                        </div>
                        <input type="range" id="speedSlider" min="1" max="3" value="2" oninput="updateSpeedUI(this.value)" class="w-full h-1 bg-slate-700 rounded-lg appearance-none accent-rose-500">
                    </div>
                    <div class="bg-slate-800/50 p-3 rounded-2xl border border-slate-700">
                        <span class="text-[10px] font-bold text-slate-400 block mb-1">音效</span>
                        <input type="range" id="sfxVolume" min="0" max="100" value="70" class="w-full h-1 bg-slate-700 rounded-lg appearance-none accent-indigo-500">
                    </div>
                    <div class="bg-slate-800/50 p-3 rounded-2xl border border-slate-700">
                        <span class="text-[10px] font-bold text-slate-400 block mb-1">音樂</span>
                        <input type="range" id="bgmVolume" min="0" max="100" value="30" oninput="updateBGMVolume(this.value)" class="w-full h-1 bg-slate-700 rounded-lg appearance-none accent-emerald-500">
                    </div>
                </div>

                <!-- 動作按鈕 -->
                <div class="flex flex-col justify-center">
                    <button id="startBtn" onclick="toggleGame()" class="w-full py-4 bg-sky-600 hover:bg-sky-500 text-white rounded-2xl font-black text-lg transition-all active:scale-95 shadow-xl shadow-sky-500/20">
                        開始測驗
                    </button>
                </div>
            </div>
            
            <button onclick="toggleSettings()" class="absolute -bottom-8 left-1/2 -translate-x-1/2 bg-slate-800 border border-slate-700 px-8 py-2 rounded-b-2xl text-[10px] font-bold text-slate-400 hover:text-white uppercase tracking-[4px]">
                展開 / 隱藏面板
            </button>
        </div>
    </div>

    <main id="mainContent" class="flex-1 pt-24 pb-6 px-4 flex flex-col items-center justify-center">
        
        <!-- 指令顯示區 (關鍵修改) -->
        <div class="w-full max-w-4xl mb-6">
            <h2 class="text-center text-[10px] font-black text-slate-500 uppercase tracking-widest mb-3">當前指令 (請執行對應動作)</h2>
            <div id="instructionList" class="flex flex-wrap justify-center gap-3">
                <!-- 由 JS 動態生成 -->
            </div>
        </div>

        <div class="w-full max-w-xl flex justify-between items-center mb-4 px-2">
            <div class="flex flex-col">
                <span class="text-[10px] font-bold text-slate-500">時間剩餘</span>
                <span id="timeDisplay" class="text-3xl font-black tabular-nums">60</span>
            </div>
            <div class="text-center">
                <span class="text-[10px] font-bold text-slate-500">得分</span>
                <span id="scoreDisplay" class="text-3xl font-black text-emerald-400 tabular-nums">0</span>
            </div>
            <div class="text-right">
                <span class="text-[10px] font-bold text-slate-500">最高連擊</span>
                <span id="comboDisplay" class="text-3xl font-black text-rose-500 tabular-nums">0</span>
            </div>
        </div>

        <div id="gameGrid" class="game-grid">
            <!-- 網格 -->
        </div>
    </main>

    <div id="toast" class="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 bg-slate-900 border-2 border-white/10 p-8 rounded-3xl shadow-2xl opacity-0 pointer-events-none transition-all z-[100] text-center">
        <div id="toastMsg" class="text-4xl font-black">READY</div>
    </div>

    <script>
        const ACTIONS_DB = [
            { label: '拍手一次', icon: '👏', color: 'text-amber-400' },
            { label: '點一下頭', icon: '👤', color: 'text-sky-400' },
            { label: '舉雙手', icon: '🙌', color: 'text-emerald-400' },
            { label: '摸耳朵', icon: '👂', color: 'text-rose-400' },
            { label: '眨眨眼', icon: '😉', color: 'text-indigo-400' },
            { label: '深呼吸', icon: '😮‍💨', color: 'text-teal-400' }
        ];

        const EMOJIS = ['🍎','🐶','🚗','🍕','⚽️','🎸','🚀','🍦','🐘','🚁','🦄','🍉','🦒','🎹','🚌','🍓'];

        let gameState = {
            isPlaying: false,
            level: 2,
            score: 0,
            combo: 0,
            timeLeft: 60,
            gridSize: 3,
            speedMode: 2,
            instructions: [], // { emoji, action }
            cells: [],
            intervals: { timer: null, spawn: null }
        };

        const dom = {
            levelSelect: document.getElementById('levelSelect'),
            instructionList: document.getElementById('instructionList'),
            grid: document.getElementById('gameGrid'),
            startBtn: document.getElementById('startBtn'),
            settingsPanel: document.getElementById('settingsPanel'),
            time: document.getElementById('timeDisplay'),
            score: document.getElementById('scoreDisplay'),
            combo: document.getElementById('comboDisplay'),
            toast: document.getElementById('toast'),
            toastMsg: document.getElementById('toastMsg')
        };

        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        function playSfx(freq, type = 'sine', dur = 0.1) {
            const vol = document.getElementById('sfxVolume').value / 100;
            if (vol <= 0) return;
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.type = type;
            osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
            gain.gain.setValueAtTime(0.2 * vol, audioCtx.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + dur);
            osc.connect(gain); gain.connect(audioCtx.destination);
            osc.start(); osc.stop(audioCtx.currentTime + dur);
        }

        let player;
        window.onYouTubeIframeAPIReady = () => {
            player = new YT.Player('player', {
                height: '0', width: '0', videoId: '5qap5aO4i9A',
                playerVars: { 'autoplay': 1, 'loop': 1, 'playlist': '5qap5aO4i9A' },
                events: { 'onReady': (e) => e.target.setVolume(gameState.bgmVolume || 30) }
            });
        };
        const tag = document.createElement('script');
        tag.src = "https://www.youtube.com/iframe_api";
        document.body.appendChild(tag);
        function updateBGMVolume(val) { if(player && player.setVolume) player.setVolume(val); }

        function toggleSettings() {
            dom.settingsPanel.classList.toggle('settings-collapsed');
        }

        function updateGridUI(val) {
            document.getElementById('gridSizeLabel').innerText = `${val}x${val}`;
            if(!gameState.isPlaying) buildGrid(val);
        }

        function updateSpeedUI(val) {
            const data = { '1': ['緩慢', 'text-emerald-400'], '2': ['適中', 'text-amber-400'], '3': ['極速', 'text-rose-400'] };
            document.getElementById('speedLabel').innerText = data[val][0];
            document.getElementById('speedLabel').className = `text-xs ${data[val][1]}`;
        }

        function buildGrid(size) {
            gameState.gridSize = parseInt(size);
            dom.grid.style.gridTemplateColumns = `repeat(${size}, 1fr)`;
            dom.grid.innerHTML = '';
            gameState.cells = [];
            for (let i = 0; i < size * size; i++) {
                const cell = document.createElement('div');
                cell.className = 'grid-cell';
                const item = document.createElement('div');
                item.className = 'pop-item';
                item.dataset.index = i;
                item.addEventListener('pointerdown', handleHit);
                cell.appendChild(item);
                dom.grid.appendChild(cell);
                gameState.cells.push({ el: item, content: '', isActive: false, timeout: null });
            }
        }

        function changeLevel(lv) {
            gameState.level = parseInt(lv);
            if (!gameState.isPlaying) refreshInstructions();
        }

        function refreshInstructions() {
            dom.instructionList.innerHTML = '';
            gameState.instructions = [];
            
            // 洗牌 Emoji
            const shuffledEmojis = [...EMOJIS].sort(() => 0.5 - Math.random());
            const shuffledActions = [...ACTIONS_DB].sort(() => 0.5 - Math.random());

            for (let i = 0; i < gameState.level; i++) {
                const mapping = {
                    emoji: shuffledEmojis[i],
                    action: shuffledActions[i]
                };
                gameState.instructions.push(mapping);

                const card = document.createElement('div');
                card.id = `inst-${mapping.emoji}`;
                card.className = `instruction-card bg-slate-800 p-3 rounded-2xl flex items-center gap-3 border-2 border-slate-700`;
                card.innerHTML = `
                    <div class="text-3xl">${mapping.emoji}</div>
                    <div class="flex flex-col">
                        <span class="text-[10px] font-bold text-slate-500 uppercase">動作指令</span>
                        <span class="text-sm font-bold text-white">${mapping.action.label} ${mapping.action.icon}</span>
                    </div>
                `;
                dom.instructionList.appendChild(card);
            }
        }

        function toggleGame() {
            if(audioCtx.state === 'suspended') audioCtx.resume();
            if(gameState.isPlaying) return stopGame();
            startGame();
        }

        function startGame() {
            gameState.isPlaying = true;
            gameState.score = 0;
            gameState.combo = 0;
            gameState.timeLeft = 60;
            gameState.speedMode = parseInt(document.getElementById('speedSlider').value);
            
            dom.startBtn.innerText = '停止測驗';
            dom.startBtn.className = "w-full py-4 bg-rose-600 hover:bg-rose-500 text-white rounded-2xl font-black text-lg transition-all";
            
            if (!dom.settingsPanel.classList.contains('settings-collapsed')) toggleSettings();
            
            refreshInstructions();
            updateScoreUI();
            showToast('READY... GO!');

            gameState.intervals.timer = setInterval(() => {
                gameState.timeLeft--;
                dom.time.innerText = gameState.timeLeft;
                if(gameState.timeLeft <= 0) stopGame();
            }, 1000);

            spawnLoop();
        }

        function stopGame() {
            gameState.isPlaying = false;
            clearInterval(gameState.intervals.timer);
            clearTimeout(gameState.intervals.spawn);
            
            gameState.cells.forEach(c => {
                clearTimeout(c.timeout);
                c.el.classList.remove('active', 'hit-success', 'hit-error');
                c.isActive = false;
            });

            dom.startBtn.innerText = '開始測驗';
            dom.startBtn.className = "w-full py-4 bg-sky-600 hover:bg-sky-500 text-white rounded-2xl font-black text-lg transition-all shadow-xl shadow-sky-500/20";
            
            playSfx(200, 'sawtooth', 0.5);
            showToast(`時間到！<br><span class="text-xl text-emerald-400">總分: ${gameState.score}</span>`, 3000);
        }

        function spawnLoop() {
            if (!gameState.isPlaying) return;
            
            // 決定出生幾個 (難度越高，同時出現越多)
            const count = Math.random() < (0.2 * gameState.level) ? 2 : 1;
            for(let i=0; i<count; i++) spawnItem();

            const base = [1200, 800, 450][gameState.speedMode-1];
            const next = base + Math.random() * (base/2);
            gameState.intervals.spawn = setTimeout(spawnLoop, next);
        }

        function spawnItem() {
            const avail = gameState.cells.filter(c => !c.isActive);
            if (avail.length === 0) return;
            const cell = avail[Math.floor(Math.random() * avail.length)];
            
            // 60% 機率出現「目標動作物品」，40% 出現「干擾物品」
            const isTarget = Math.random() < 0.6;
            let emoji;
            if (isTarget) {
                emoji = gameState.instructions[Math.floor(Math.random() * gameState.instructions.length)].emoji;
                // 強調指令卡
                const card = document.getElementById(`inst-${emoji}`);
                if(card) {
                    card.classList.add('active-target');
                    setTimeout(() => card.classList.remove('active-target'), 800);
                }
            } else {
                // 排除目前指令中的 Emoji，隨機選一個
                const instEmojis = gameState.instructions.map(i => i.emoji);
                const others = EMOJIS.filter(e => !instEmojis.includes(e));
                emoji = others[Math.floor(Math.random() * others.length)];
            }

            cell.content = emoji;
            cell.el.innerText = emoji;
            cell.el.className = 'pop-item active';
            cell.isActive = true;
            
            playSfx(400, 'sine', 0.05);

            const stay = [2500, 1500, 900][gameState.speedMode-1];
            cell.timeout = setTimeout(() => {
                if(cell.isActive) {
                    cell.el.classList.remove('active');
                    cell.isActive = false;
                }
            }, stay);
        }

        function handleHit(e) {
            if (!gameState.isPlaying) return;
            const cell = gameState.cells[e.target.dataset.index];
            if (!cell.isActive) return;

            clearTimeout(cell.timeout);
            const isCorrect = gameState.instructions.some(i => i.emoji === cell.content);

            if (isCorrect) {
                playSfx(880, 'sine', 0.1);
                gameState.score += (10 + gameState.combo);
                gameState.combo++;
                cell.el.classList.add('hit-success');
            } else {
                playSfx(150, 'sawtooth', 0.2);
                gameState.score = Math.max(0, gameState.score - 5);
                gameState.combo = 0;
                cell.el.classList.add('hit-error');
            }

            updateScoreUI();
            setTimeout(() => {
                cell.el.classList.remove('active', 'hit-success', 'hit-error');
                cell.isActive = false;
            }, 100);
        }

        function updateScoreUI() {
            dom.score.innerText = gameState.score;
            dom.combo.innerText = gameState.combo;
        }

        function showToast(msg, dur = 1500) {
            dom.toastMsg.innerHTML = msg;
            dom.toast.style.opacity = '1';
            dom.toast.style.transform = 'translate(-50%, -50%) scale(1)';
            setTimeout(() => {
                dom.toast.style.opacity = '0';
                dom.toast.style.transform = 'translate(-50%, -50%) scale(0.9)';
            }, dur);
        }

        window.onload = () => {
            buildGrid(3);
            refreshInstructions();
        };
    </script>
</body>
</html>
