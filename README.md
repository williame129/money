<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>金錢帝國 - 終極點擊大亨</title>
    <style>
        * { box-sizing: border-box; font-family: 'Segoe UI', Microsoft JhengHei, sans-serif; }
        body { background-color: #0f172a; color: #f8fafc; margin: 0; padding: 20px; display: flex; justify-content: center; }
        .game-wrapper { width: 100%; max-width: 900px; display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        @media (max-width: 768px) { .game-wrapper { grid-template-columns: 1fr; } }
        
        .panel { background-color: #1e293b; border-radius: 12px; padding: 20px; border: 1px solid #334155; position: relative; }
        .full-width { grid-column: 1 / -1; }
        
        h1, h2, h3 { margin-top: 0; color: #38bdf8; text-align: center; }
        .stats-box { background: #0f172a; padding: 15px; border-radius: 8px; text-align: center; margin-bottom: 15px; border: 1px solid #334155; }
        .money { font-size: 2.2em; color: #4ade80; font-weight: bold; }
        .sub-stat { font-size: 0.9em; color: #94a3b8; margin-top: 4px; }
        
        /* 點擊按鈕與特效 */
        .click-area { text-align: center; position: relative; margin: 20px 0; }
        .big-btn {
            background: linear-gradient(135deg, #22c55e, #15803d); color: white; border: none;
            width: 160px; height: 160px; border-radius: 50%; font-size: 1.5em; font-weight: bold;
            cursor: pointer; box-shadow: 0 10px 25px rgba(34, 197, 94, 0.4); transition: transform 0.05s, box-shadow 0.1s;
            user-select: none;
        }
        .big-btn:active { transform: scale(0.92); box-shadow: 0 5px 10px rgba(34, 197, 94, 0.4); }
        .floating-text { position: absolute; color: #4ade80; font-weight: bold; pointer-events: none; animation: floatUp 0.8s ease-out forwards; }
        @keyframes floatUp { 0% { opacity: 1; transform: translateY(0); } 100% { opacity: 0; transform: translateY(-40px); } }

        /* 標籤頁面 */
        .tabs { display: flex; gap: 5px; margin-bottom: 15px; }
        .tab-btn { flex: 1; padding: 8px; background: #334155; border: none; color: #fff; cursor: pointer; border-radius: 6px; font-weight: bold; }
        .tab-btn.active { background: #0284c7; }
        .tab-content { display: none; max-height: 400px; overflow-y: auto; padding-right: 5px; }
        .tab-content.active { display: block; }

        /* 列表項目 */
        .item-card {
            background: #0f172a; border: 1px solid #334155; border-radius: 8px; padding: 10px 12px;
            margin-bottom: 8px; display: flex; justify-content: space-between; align-items: center;
        }
        .item-info { flex: 1; }
        .item-title { font-weight: bold; color: #f3f4f6; font-size: 0.95em; }
        .item-desc { font-size: 0.8em; color: #9ca3af; }
        .buy-btn {
            background: #0284c7; border: none; color: white; padding: 8px 12px; border-radius: 6px;
            cursor: pointer; font-weight: bold; font-size: 0.85em; transition: 0.2s; min-width: 90px;
        }
        .buy-btn:hover:not(:disabled) { background: #0369a1; }
        .buy-btn:disabled { background: #475569; color: #94a3b8; cursor: not-allowed; opacity: 0.6; }

        /* 小功能元件 */
        .event-banner { background: #854d0e; color: #fef08a; padding: 8px; border-radius: 6px; text-align: center; font-size: 0.85em; margin-bottom: 10px; display: none; }
        .achieve-badge { display: inline-block; background: #334155; padding: 4px 8px; border-radius: 4px; font-size: 0.75em; margin: 2px; }
        .achieve-badge.unlocked { background: #15803d; color: white; }
        .flex-between { display: flex; justify-content: space-between; gap: 10px; }
        .action-btn { flex: 1; padding: 8px; background: #475569; border: none; color: white; border-radius: 6px; cursor: pointer; font-size: 0.85em; }
        .action-btn:hover { background: #64748b; }
        .golden-coin { position: absolute; width: 40px; height: 40px; background: #eab308; border-radius: 50%; border: 3px solid #fef08a; cursor: pointer; display: flex; justify-content: center; align-items: center; font-weight: bold; font-size: 1.2em; box-shadow: 0 0 15px #eab308; animation: pulse 1s infinite alternate; }
        @keyframes pulse { from { transform: scale(1); } to { transform: scale(1.1); } }
    </style>
</head>
<body>

<div class="game-wrapper">
    <!-- 左側：主要操作區 -->
    <div class="panel">
        <h1>💰 金錢帝國</h1>
        
        <!-- 功能 1: 隨機市場事件公告 -->
        <div id="eventBanner" class="event-banner">特別事件發生中！</div>

        <div class="stats-box">
            <div class="money" id="money">$0</div>
            <div class="sub-stat" id="mps">每秒被動收入: $0</div>
            <div class="sub-stat" id="mpc">每次點擊收益: $1</div>
            <!-- 功能 2: 暴擊倍率顯示 -->
            <div class="sub-stat" id="critStat">暴擊機率: 5% (1.5x)</div>
        </div>

        <div class="click-area" id="clickArea">
            <button class="big-btn" id="mainBtn" onclick="clickMoney(event)">點擊賺錢</button>
        </div>

        <!-- 功能 3: 離線收益統計與小工具 -->
        <div class="flex-between">
            <button class="action-btn" onclick="saveGame()">💾 手動存檔</button>
            <button class="action-btn" onclick="exportSave()">📤 匯出存檔</button>
            <button class="action-btn" onclick="importSave()">📥 匯入存檔</button>
        </div>
        
        <!-- 功能 4: 轉生重置系統 (Prestige) -->
        <div style="margin-top: 15px; border-top: 1px solid #334155; pt: 10px;">
            <div class="sub-stat" id="prestigeStat">聲望星級: 0 (永久收益 +0%)</div>
            <button class="buy-btn" style="width: 100%; margin-top: 5px; background: #dc2626;" onclick="prestige()">清空進度重生 (需求: $1,000,000)</button>
        </div>
    </div>

    <!-- 右側：升級與系統區 -->
    <div class="panel">
        <div class="tabs">
            <button class="tab-btn active" onclick="switchTab('upgrades')">手動升級</button>
            <button class="tab-btn" onclick="switchTab('assets')">被動資產</button>
            <button class="tab-btn" onclick="switchTab('tech')">科技研發</button>
            <button class="tab-btn" onclick="switchTab('achievements')">成就</button>
        </div>

        <!-- 30 個升級項目分類顯示 -->
        <div id="upgrades" class="tab-content active"></div>
        <div id="assets" class="tab-content"></div>
        <div id="tech" class="tab-content"></div>
        
        <!-- 功能 5: 成就勳章系統 -->
        <div id="achievements" class="tab-content">
            <div id="achieveList"></div>
        </div>
    </div>
</div>

<script>
    // 遊戲基礎資料
    let gameState = {
        money: 0,
        totalClickCount: 0,
        clickValue: 1,
        mps: 0,
        critChance: 0.05,
        critMulti: 1.5,
        prestigeStars: 0,
        prestigeMulti: 1.0,
        lastOnline: Date.now(),
        upgrades: {},
        assets: {},
        techs: {},
        achievements: {}
    };

    let eventMultiplier = 1;

    // -------------------------------------------------------------
    // 30 個不重複升級清單定義
    // -------------------------------------------------------------

    // 1. 手動點擊升級 (10 個)
    const clickUpgradesList = [
        { id: 'c1', name: '鐵製滑鼠', desc: '點擊力量 +1', cost: 15, val: 1 },
        { id: 'c2', name: '人體工學握把', desc: '點擊力量 +5', cost: 100, val: 5 },
        { id: 'c3', name: '雙重點擊技巧', desc: '點擊力量 +20', cost: 500, val: 20 },
        { id: 'c4', name: '電競機械軸', desc: '點擊力量 +100', cost: 2500, val: 100 },
        { id: 'c5', name: '連點巨集程式', desc: '點擊力量 +500', cost: 10000, val: 500 },
        { id: 'c6', name: '神經傳導連線', desc: '點擊力量 +2,500', cost: 50000, val: 2500 },
        { id: 'c7', name: '量子點擊器', desc: '點擊力量 +12,000', cost: 250000, val: 12000 },
        { id: 'c8', name: '光速雷射感應', desc: '點擊力量 +60,000', cost: 1000000, val: 60000 },
        { id: 'c9', name: '時空裂隙點擊', desc: '點擊力量 +350,000', cost: 5000000, val: 350000 },
        { id: 'c10', name: '創世神之手', desc: '點擊力量 +2,000,000', cost: 25000000, val: 2000000 }
    ];

    // 2. 被動資產升級 (10 個)
    const assetsList = [
        { id: 'a1', name: '自動點擊腳本', desc: '每秒產出 +1', cost: 50, mps: 1 },
        { id: 'a2', name: '街邊路邊攤', desc: '每秒產出 +8', cost: 350, mps: 8 },
        { id: 'a3', name: '加盟手搖飲店', desc: '每秒產出 +40', cost: 2000, mps: 40 },
        { id: 'a4', name: '自動販賣機網', desc: '每秒產出 +200', cost: 10000, mps: 200 },
        { id: 'a5', name: '連鎖超市品牌', desc: '每秒產出 +1,000', cost: 60000, mps: 1000 },
        { id: 'a6', name: '市中心商辦大樓', desc: '每秒產出 +5,000', cost: 350000, mps: 5000 },
        { id: 'a7', name: '跨境電商平台', desc: '每秒產出 +25,000', cost: 2000000, mps: 25000 },
        { id: 'a8', name: '晶圓代工廠', desc: '每秒產出 +120,000', cost: 10000000, mps: 120000 },
        { id: 'a9', name: '商業航天公司', desc: '每秒產出 +700,000', cost: 50000000, mps: 700000 },
        { id: 'a10', name: '行星小行星採礦', desc: '每秒產出 +4,000,000', cost: 300000000, mps: 4000000 }
    ];

    // 3. 科技研發升級 (10 個)
    const techList = [
        { id: 't1', name: '暴擊訓練', desc: '暴擊率 +2%', cost: 200, effect: () => gameState.critChance += 0.02 },
        { id: 't2', name: '槓桿投資', desc: '暴擊傷害倍率 +0.5x', cost: 1000, effect: () => gameState.critMulti += 0.5 },
        { id: 't3', name: '大數據精準行銷', desc: '所有被動收益提升 10%', cost: 5000, effect: () => gameState.mpsMultiplier = (gameState.mpsMultiplier||1) * 1.1 },
        { id: 't4', name: '微型晶片強化', desc: '暴擊率 +3%', cost: 20000, effect: () => gameState.critChance += 0.03 },
        { id: 't5', name: '自動化物流AI', desc: '所有被動收益提升 15%', cost: 100000, effect: () => gameState.mpsMultiplier = (gameState.mpsMultiplier||1) * 1.15 },
        { id: 't6', name: '高頻交易演算法', desc: '暴擊傷害倍率 +1.0x', cost: 500000, effect: () => gameState.critMulti += 1.0 },
        { id: 't7', name: '區塊鏈智能合約', desc: '所有被動收益提升 20%', cost: 2500000, effect: () => gameState.mpsMultiplier = (gameState.mpsMultiplier||1) * 1.20 },
        { id: 't8', name: '金礦幸運磁場', desc: '金幣出現機率提升', cost: 12000000, effect: () => window.goldRate = (window.goldRate||0.005) * 2 },
        { id: 't9', name: '超導體電力網', desc: '所有被動收益提升 30%', cost: 80000000, effect: () => gameState.mpsMultiplier = (gameState.mpsMultiplier||1) * 1.30 },
        { id: 't10', name: '奇點人工智慧', desc: '暴擊率 +10%', cost: 500000000, effect: () => gameState.critChance += 0.10 }
    ];

    // 成就列表 (功能 5)
    const achievementsList = [
        { id: 'ach1', name: '第一桶金', desc: '累積賺取 $100', check: () => gameState.money >= 100 },
        { id: 'ach2', name: '瘋狂點擊者', desc: '手動點擊超過 100 次', check: () => gameState.totalClickCount >= 100 },
        { id: 'ach3', name: '小有資產', desc: '每秒收益達到 $100', check: () => calculateMPS() >= 100 },
        { id: 'ach4', name: '百萬富翁', desc: '持有金額達到 $1,000,000', check: () => gameState.money >= 1000000 },
        { id: 'ach5', name: '科技大亨', desc: '解鎖 5 個科技研發', check: () => Object.keys(gameState.techs).length >= 5 }
    ];

    // -------------------------------------------------------------
    // 初始化與UI繪製
    // -------------------------------------------------------------
    function initUI() {
        renderList(clickUpgradesList, 'upgrades', 'buyClickUpgrade');
        renderList(assetsList, 'assets', 'buyAsset');
        renderList(techList, 'tech', 'buyTech', true);
        renderAchievements();
    }

    function renderList(list, targetId, buyFuncName, isSingle = false) {
        const container = document.getElementById(targetId);
        container.innerHTML = '';
        list.forEach(item => {
            const div = document.createElement('div');
            div.className = 'item-card';
            div.innerHTML = `
                <div class="item-info">
                    <div class="item-title">${item.name} <span id="count-${item.id}" style="color:#38bdf8"></span></div>
                    <div class="item-desc">${item.desc}</div>
                </div>
                <button class="buy-btn" id="btn-${item.id}" onclick="${buyFuncName}('${item.id}')">
                    $<span id="cost-${item.id}">${item.cost}</span>
                </button>
            `;
            container.appendChild(div);
        });
    }

    function renderAchievements() {
        const container = document.getElementById('achieveList');
        container.innerHTML = '';
        achievementsList.forEach(a => {
            const unlocked = gameState.achievements[a.id];
            const span = document.createElement('span');
            span.className = `achieve-badge ${unlocked ? 'unlocked' : ''}`;
            span.innerText = `${unlocked ? '✓ ' : '🔒 '}${a.name}: ${a.desc}`;
            container.appendChild(span);
        });
    }

    // -------------------------------------------------------------
    // 遊戲核心邏輯 (功能 6: 暴擊 / 功能 7: 被動收益計算 / 功能 8: 事件 multiplier)
    // -------------------------------------------------------------
    function calculateClickValue() {
        let base = 1;
        clickUpgradesList.forEach(item => {
            let count = gameState.upgrades[item.id] || 0;
            base += item.val * count;
        });
        return base * gameState.prestigeMulti;
    }

    function calculateMPS() {
        let baseMps = 0;
        assetsList.forEach(item => {
            let count = gameState.assets[item.id] || 0;
            baseMps += item.mps * count;
        });
        let mult = (gameState.mpsMultiplier || 1) * gameState.prestigeMulti * eventMultiplier;
        return baseMps * mult;
    }

    function clickMoney(e) {
        gameState.totalClickCount++;
        let isCrit = Math.random() < gameState.critChance;
        let gained = calculateClickValue();
        
        if (isCrit) {
            gained *= gameState.critMulti;
        }

        gameState.money += gained;

        // 功能 9: 點擊浮動文字特效
        showFloatingText(e.clientX, e.clientY, `+$${Math.floor(gained).toLocaleString()}${isCrit ? ' 暴擊!' : ''}`, isCrit);
        updateDisplay();
    }

    function showFloatingText(x, y, text, isCrit) {
        const el = document.createElement('div');
        el.className = 'floating-text';
        el.innerText = text;
        if (isCrit) el.style.color = '#facc15';
        el.style.left = `${x - 20}px`;
        el.style.top = `${y - 20}px`;
        document.body.appendChild(el);
        setTimeout(() => el.remove(), 800);
    }

    // -------------------------------------------------------------
    // 購買邏輯
    // -------------------------------------------------------------
    function buyClickUpgrade(id) {
        let item = clickUpgradesList.find(x => x.id === id);
        let count = gameState.upgrades[id] || 0;
        let currentCost = Math.floor(item.cost * Math.pow(1.15, count));
        
        if (gameState.money >= currentCost) {
            gameState.money -= currentCost;
            gameState.upgrades[id] = count + 1;
            updateDisplay();
        }
    }

    function buyAsset(id) {
        let item = assetsList.find(x => x.id === id);
        let count = gameState.assets[id] || 0;
        let currentCost = Math.floor(item.cost * Math.pow(1.15, count));

        if (gameState.money >= currentCost) {
            gameState.money -= currentCost;
            gameState.assets[id] = count + 1;
            updateDisplay();
        }
    }

    function buyTech(id) {
        let item = techList.find(x => x.id === id);
        if (!gameState.techs[id] && gameState.money >= item.cost) {
            gameState.money -= item.cost;
            gameState.techs[id] = true;
            item.effect();
            updateDisplay();
        }
    }

    // -------------------------------------------------------------
    // 擴充功能系統
    // -------------------------------------------------------------

    // 功能 10: 隨機黃金硬幣爆發事件 (Golden Coin Event)
    function spawnGoldenCoin() {
        if (Math.random() < (window.goldRate || 0.005)) {
            if (document.getElementById('goldCoin')) return;
            const coin = document.createElement('div');
            coin.id = 'goldCoin';
            coin.className = 'golden-coin';
            coin.innerText = '💰';
            coin.style.left = `${Math.random() * 80 + 10}%`;
            coin.style.top = `${Math.random() * 70 + 15}%`;
            
            coin.onclick = () => {
                let bonus = calculateMPS() * 15 + calculateClickValue() * 20 + 50;
                gameState.money += bonus;
                showFloatingText(window.innerWidth / 2, window.innerHeight / 2, `黃金紅包: +$${Math.floor(bonus).toLocaleString()}!`, true);
                coin.remove();
            };
            document.body.appendChild(coin);
            setTimeout(() => { if (coin.parentNode) coin.remove(); }, 4000);
        }
    }

    // 功能 11: 市場隨機事件 (Market Events)
    function triggerRandomEvent() {
        if (Math.random() < 0.05) { // 5% 觸發率
            const events = [
                { name: '股市牛市升溫！被動收益提升 2 倍 (15秒)', mult: 2, duration: 15000 },
                { name: '通貨膨脹打擊！被動收益降低至 0.5 倍 (10秒)', mult: 0.5, duration: 10000 },
                { name: '科技股大爆發！被動收益提升 3 倍 (10秒)', mult: 3, duration: 10000 }
            ];
            let ev = events[Math.floor(Math.random() * events.length)];
            eventMultiplier = ev.mult;
            
            const banner = document.getElementById('eventBanner');
            banner.innerText = `📢 ${ev.name}`;
            banner.style.display = 'block';

            setTimeout(() => {
                eventMultiplier = 1;
                banner.style.display = 'none';
            }, ev.duration);
        }
    }

    // 功能 4 實作: 轉生/重生系統
    function prestige() {
        let req = 1000000;
        if (gameState.money >= req) {
            if (confirm("確定要執行重置嗎？您將失去金錢與一般升級，但獲得 1 顆聲望之星 (所有收益永久 +50%)！")) {
                gameState.prestigeStars++;
                gameState.prestigeMulti = 1 + (gameState.prestigeStars * 0.5);
                gameState.money = 0;
                gameState.upgrades = {};
                gameState.assets = {};
                gameState.techs = {};
                gameState.mpsMultiplier = 1;
                gameState.critChance = 0.05;
                gameState.critMulti = 1.5;
                updateDisplay();
                alert("重生成功！獲得強力的聲望加成！");
            }
        } else {
            alert(`需要至少 $${req.toLocaleString()} 才能執行重生！`);
        }
    }

    // 功能 12: 自動存檔與離線收益計算
    function saveGame() {
        gameState.lastOnline = Date.now();
        localStorage.setItem('clicker_save', JSON.stringify(gameState));
    }

    function loadGame() {
        let saved = localStorage.getItem('clicker_save');
        if (saved) {
            try {
                let loaded = JSON.parse(saved);
                gameState = { ...gameState, ...loaded };
                
                // 計算離線收益
                let now = Date.now();
                let offlineSeconds = Math.floor((now - (gameState.lastOnline || now)) / 1000);
                if (offlineSeconds > 5) {
                    let offlineMps = calculateMPS();
                    let earned = offlineSeconds * offlineMps * 0.5; // 離線獲得 50% 收益
                    if (earned > 0) {
                        gameState.money += earned;
                        alert(`歡迎回來！您離線了 ${offlineSeconds} 秒，獲得了 $${Math.floor(earned).toLocaleString()} 離線收益！`);
                    }
                }
            } catch(e) { console.error("存檔載入失敗", e); }
        }
    }

    function exportSave() {
        saveGame();
        let str = btoa(JSON.stringify(gameState));
        prompt("請複製下方存檔代碼：", str);
    }

    function importSave() {
        let str = prompt("請貼上存檔代碼：");
        if (str) {
            try {
                let data = JSON.parse(atob(str));
                gameState = data;
                updateDisplay();
                alert("存檔匯入成功！");
            } catch(e) { alert("無效的存檔代碼！"); }
        }
    }

    // 檢查成就
    function checkAchievements() {
        achievementsList.forEach(a => {
            if (!gameState.achievements[a.id] && a.check()) {
                gameState.achievements[a.id] = true;
                showFloatingText(window.innerWidth / 2, 100, `🏆 解鎖成就: ${a.name}`, true);
                renderAchievements();
            }
        });
    }

    // 切換標籤頁
    function switchTab(tabId) {
        document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
        document.querySelectorAll('.tab-btn').forEach(el => el.classList.remove('active'));
        document.getElementById(tabId).classList.add('active');
        event.target.classList.add('active');
    }

    // 更新介面
    function updateDisplay() {
        let currentMps = calculateMPS();
        let currentClick = calculateClickValue();

        document.getElementById('money').innerText = '$' + Math.floor(gameState.money).toLocaleString();
        document.getElementById('mps').innerText = `每秒被動收入: $${Math.floor(currentMps).toLocaleString()}${eventMultiplier !== 1 ? ` (事件倍率 x${eventMultiplier})` : ''}`;
        document.getElementById('mpc').innerText = `每次點擊收益: $${Math.floor(currentClick).toLocaleString()}`;
        document.getElementById('critStat').innerText = `暴擊機率: ${(gameState.critChance * 100).toFixed(0)}% (${gameState.critMulti}x)`;
        document.getElementById('prestigeStat').innerText = `聲望星級: ${gameState.prestigeStars} (永久收益 +${((gameState.prestigeMulti - 1)*100).toFixed(0)}%)`;

        // 更新手動升級列表 UI
        clickUpgradesList.forEach(item => {
            let count = gameState.upgrades[item.id] || 0;
            let cost = Math.floor(item.cost * Math.pow(1.15, count));
            document.getElementById(`cost-${item.id}`).innerText = cost.toLocaleString();
            document.getElementById(`count-${item.id}`).innerText = count > 0 ? `[Lv.${count}]` : '';
            document.getElementById(`btn-${item.id}`).disabled = gameState.money < cost;
        });

        // 更新被動資產列表 UI
        assetsList.forEach(item => {
            let count = gameState.assets[item.id] || 0;
            let cost = Math.floor(item.cost * Math.pow(1.15, count));
            document.getElementById(`cost-${item.id}`).innerText = cost.toLocaleString();
            document.getElementById(`count-${item.id}`).innerText = count > 0 ? `[x${count}]` : '';
            document.getElementById(`btn-${item.id}`).disabled = gameState.money < cost;
        });

        // 更新科技列表 UI
        techList.forEach(item => {
            let bought = gameState.techs[item.id];
            let btn = document.getElementById(`btn-${item.id}`);
            if (bought) {
                btn.disabled = true;
                btn.innerText = '已解鎖';
            } else {
                btn.disabled = gameState.money < item.cost;
            }
        });

        checkAchievements();
    }

    // -------------------------------------------------------------
    // 主遊戲循環 (Tick)
    // -------------------------------------------------------------
    initUI();
    loadGame();
    updateDisplay();

    // 每 0.1 秒執行一次邏輯更新 (順暢計時)
    setInterval(() => {
        let mps = calculateMPS();
        gameState.money += mps / 10;
        updateDisplay();
    }, 100);

    // 每 3 秒檢查事件與黃金硬幣
    setInterval(() => {
        spawnGoldenCoin();
        triggerRandomEvent();
    }, 3000);

    // 每 10 秒自動存檔
    setInterval(saveGame, 10000);
</script>
</body>
</html>
