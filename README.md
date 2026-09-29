<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>金錢帝國 2.0 - 終極宇宙版</title>
    <style>
        * { box-sizing: border-box; font-family: 'Segoe UI', Microsoft JhengHei, sans-serif; }
        body { background-color: #0b0f19; color: #f8fafc; margin: 0; padding: 15px; display: flex; justify-content: center; }
        .game-wrapper { width: 100%; max-width: 1100px; display: grid; grid-template-columns: 320px 1fr; gap: 15px; }
        @media (max-width: 850px) { .game-wrapper { grid-template-columns: 1fr; } }
        
        .panel { background-color: #151f30; border-radius: 12px; padding: 18px; border: 1px solid #233554; position: relative; }
        
        h1, h2, h3 { margin-top: 0; color: #38bdf8; text-align: center; }
        .stats-box { background: #0b0f19; padding: 12px; border-radius: 8px; text-align: center; margin-bottom: 12px; border: 1px solid #233554; }
        .money { font-size: 2em; color: #4ade80; font-weight: bold; }
        .sub-stat { font-size: 0.85em; color: #94a3b8; margin-top: 3px; }
        
        /* 點擊按鈕與特效 */
        .click-area { text-align: center; position: relative; margin: 15px 0; }
        .big-btn {
            background: linear-gradient(135deg, #22c55e, #15803d); color: white; border: none;
            width: 140px; height: 140px; border-radius: 50%; font-size: 1.4em; font-weight: bold;
            cursor: pointer; box-shadow: 0 8px 20px rgba(34, 197, 94, 0.4); transition: transform 0.05s;
            user-select: none;
        }
        .big-btn:active { transform: scale(0.92); }
        .floating-text { position: absolute; color: #4ade80; font-weight: bold; pointer-events: none; animation: floatUp 0.8s ease-out forwards; z-index: 100; }
        @keyframes floatUp { 0% { opacity: 1; transform: translateY(0); } 100% { opacity: 0; transform: translateY(-40px); } }

        /* 標籤頁面 */
        .tabs { display: flex; gap: 4px; margin-bottom: 12px; flex-wrap: wrap; }
        .tab-btn { flex: 1; min-width: 70px; padding: 8px 4px; background: #233554; border: none; color: #fff; cursor: pointer; border-radius: 6px; font-weight: bold; font-size: 0.8em; }
        .tab-btn.active { background: #0284c7; }
        .tab-content { display: none; max-height: 520px; overflow-y: auto; padding-right: 5px; }
        .tab-content.active { display: block; }

        /* 列表項目 */
        .item-card {
            background: #0b0f19; border: 1px solid #233554; border-radius: 8px; padding: 8px 10px;
            margin-bottom: 6px; display: flex; justify-content: space-between; align-items: center;
        }
        .item-title { font-weight: bold; color: #f3f4f6; font-size: 0.9em; }
        .item-desc { font-size: 0.78em; color: #9ca3af; }
        .buy-btn {
            background: #0284c7; border: none; color: white; padding: 6px 10px; border-radius: 6px;
            cursor: pointer; font-weight: bold; font-size: 0.8em; transition: 0.2s; min-width: 85px;
        }
        .buy-btn:hover:not(:disabled) { background: #0369a1; }
        .buy-btn:disabled { background: #334155; color: #64748b; cursor: not-allowed; opacity: 0.6; }

        /* 小功能元件 */
        .event-banner { background: #854d0e; color: #fef08a; padding: 6px; border-radius: 6px; text-align: center; font-size: 0.8em; margin-bottom: 8px; display: none; }
        .achieve-badge { display: inline-block; background: #233554; padding: 4px 8px; border-radius: 4px; font-size: 0.75em; margin: 2px; }
        .achieve-badge.unlocked { background: #15803d; color: white; }
        .flex-between { display: flex; justify-content: space-between; gap: 6px; margin-top: 8px; }
        .action-btn { flex: 1; padding: 6px; background: #334155; border: none; color: white; border-radius: 6px; cursor: pointer; font-size: 0.8em; }
        .action-btn:hover { background: #475569; }
        .golden-coin { position: absolute; width: 36px; height: 36px; background: #eab308; border-radius: 50%; border: 3px solid #fef08a; cursor: pointer; display: flex; justify-content: center; align-items: center; font-size: 1.1em; box-shadow: 0 0 12px #eab308; animation: pulse 0.8s infinite alternate; z-index: 99; }
        @keyframes pulse { from { transform: scale(1); } to { transform: scale(1.15); } }

        /* 新功能元件 UI */
        .frenzy-overlay { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; border: 4px solid #ef4444; pointer-events: none; z-index: 999; display: none; animation: flash 0.5s infinite alternate; }
        @keyframes flash { from { opacity: 0.3; } to { opacity: 0.8; } }
        .combo-counter { color: #f59e0b; font-weight: bold; font-size: 1.1em; height: 20px; }
        .stock-box { background: #0b0f19; padding: 8px; border-radius: 6px; margin-bottom: 8px; border: 1px solid #233554; font-size: 0.85em; }
    </style>
</head>
<body>

<div class="frenzy-overlay" id="frenzyOverlay"></div>

<div class="game-wrapper">
    <!-- 左側：控制面板 -->
    <div class="panel">
        <h1>💰 金錢帝國 2.0</h1>
        
        <div id="eventBanner" class="event-banner">特別事件發生中！</div>

        <div class="stats-box">
            <div class="money" id="money">$0</div>
            <div class="sub-stat" id="mps">每秒被動: $0</div>
            <div class="sub-stat" id="mpc">每次點擊: $1</div>
            <div class="sub-stat" id="critStat">暴擊率: 5% (1.5x)</div>
            <!-- 功能：連擊數與熱血狂熱提示 -->
            <div class="combo-counter" id="comboStat">連擊: 0x</div>
        </div>

        <div class="click-area" id="clickArea">
            <button class="big-btn" id="mainBtn" onclick="clickMoney(event)">點擊賺錢</button>
        </div>

        <!-- 轉生機制（根據歷史總收入計算） -->
        <div style="background:#0b0f19; padding:10px; border-radius:8px; border:1px solid #233554; margin-bottom:10px;">
            <div style="font-weight:bold; color:#e2e8f0; font-size:0.85em;">轉生聲望系統</div>
            <div class="sub-stat" id="prestigeStat">持有聲望水晶: 0 (永久 +0%)</div>
            <div class="sub-stat" id="nextPrestigeStat">重置可獲得: +0 水晶</div>
            <button class="buy-btn" style="width: 100%; margin-top: 6px; background: #dc2626;" onclick="prestige()">轉生並領取水晶</button>
        </div>

        <!-- 小工具與系統 -->
        <div class="flex-between">
            <button class="action-btn" onclick="toggleAutoClicker()" id="autoClickerBtn">🤖 自動點擊器: 關</button>
            <button class="action-btn" onclick="saveGame()">💾 存檔</button>
        </div>
        <div class="flex-between">
            <button class="action-btn" onclick="exportSave()">📤 匯出</button>
            <button class="action-btn" onclick="importSave()">📥 匯入</button>
            <button class="action-btn" onclick="resetGame()" style="background:#991b1b;">🗑️ 重置</button>
        </div>
    </div>

    <!-- 右側：分頁面板 -->
    <div class="panel">
        <div class="tabs">
            <button class="tab-btn active" onclick="switchTab('upgrades')">手動升級</button>
            <button class="tab-btn" onclick="switchTab('assets')">被動資產</button>
            <button class="tab-btn" onclick="switchTab('tech')">科技研發</button>
            <button class="tab-btn" onclick="switchTab('presTree')">聲望樹</button>
            <button class="tab-btn" onclick="switchTab('stock')">股市投機</button>
            <button class="tab-btn" onclick="switchTab('achievements')">成就與統計</button>
        </div>

        <!-- 60 個升級與功能分類 -->
        <div id="upgrades" class="tab-content active"></div>
        <div id="assets" class="tab-content"></div>
        <div id="tech" class="tab-content"></div>
        <div id="presTree" class="tab-content"></div>
        
        <!-- 新功能：股市模擬交易 -->
        <div id="stock" class="tab-content">
            <h3>📈 虛擬股市投機</h3>
            <p style="font-size:0.8em; color:#9ca3af;">股價每 3 秒波動一次，低買高賣賺取價差！</p>
            <div id="stockList"></div>
        </div>
        
        <!-- 成就與統計數據 -->
        <div id="achievements" class="tab-content">
            <div style="background:#0b0f19; padding:8px; border-radius:6px; margin-bottom:10px; font-size:0.8em; border:1px solid #233554;" id="statsDetail">
                <!-- 統計數據 -->
            </div>
            <div id="achieveList"></div>
        </div>
    </div>
</div>

<script>
    // 遊戲總狀態資料結構
    let gameState = {
        money: 0,
        totalMoneyEarned: 0, // 用於精確計算轉生水晶
        totalClickCount: 0,
        clickValue: 1,
        mps: 0,
        critChance: 0.05,
        critMulti: 1.5,
        prestigeCrystals: 0,
        lastOnline: Date.now(),
        upgrades: {},
        assets: {},
        techs: {},
        presTechs: {},
        achievements: {},
        hasAutoClicker: false,
        autoClickerActive: false,
        stocks: {
            'TECH': { name: '高科技指數', price: 100, shares: 0, history: [100] },
            'ENERGY': { name: '能源巨頭', price: 50, shares: 0, history: [50] },
            'COIN': { name: '加密概念股', price: 10, shares: 0, history: [10] }
        }
    };

    // 運行時變數
    let eventMultiplier = 1;
    let comboCount = 0;
    let comboTimer = null;
    let isFrenzy = false;
    let frenzyMultiplier = 1;

    // -------------------------------------------------------------
    // 60 個不重複升級清單（原有 30 個 + 新增 30 個）
    // -------------------------------------------------------------

    // 1. 手動點擊升級 (20 個)
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
        { id: 'c10', name: '創世神之手', desc: '點擊力量 +2,000,000', cost: 25000000, val: 2000000 },
        // 新增 10 個手動點擊升級
        { id: 'c11', name: '納米肌術手套', desc: '點擊力量 +10,000,000', cost: 150000000, val: 10000000 },
        { id: 'c12', name: '反物質手指', desc: '點擊力量 +60,000,000', cost: 1000000000, val: 60000000 },
        { id: 'c13', name: '引力波共振點擊', desc: '點擊力量 +400,000,000', cost: 8000000000, val: 400000000 },
        { id: 'c14', name: '超弦維度指壓', desc: '點擊力量 +2.5 B', cost: 50000000000, val: 2500000000 },
        { id: 'c15', name: '奇點爆破打擊', desc: '點擊力量 +15 B', cost: 350000000000, val: 15000000000 },
        { id: 'c16', name: '多元宇宙脈衝', desc: '點擊力量 +100 B', cost: 2500000000000, val: 100000000000 },
        { id: 'c17', name: '時間線重疊敲擊', desc: '點擊力量 +700 B', cost: 20000000000000, val: 700000000000 },
        { id: 'c18', name: '因果律改寫指法', desc: '點擊力量 +5 T', cost: 150000000000000, val: 5000000000000 },
        { id: 'c19', name: '虛無之光觸碰', desc: '點擊力量 +35 T', cost: 1000000000000000, val: 35000000000000 },
        { id: 'c20', name: '全知全能之觸', desc: '點擊力量 +250 T', cost: 8000000000000000, val: 250000000000000 }
    ];

    // 2. 被動資產升級 (20 個)
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
        { id: 'a10', name: '行星小行星採礦', desc: '每秒產出 +4,000,000', cost: 300000000, mps: 4000000 },
        // 新增 10 個被動資產
        { id: 'a11', name: '恆星戴森球基地', desc: '每秒產出 +25,000,000', cost: 2000000000, mps: 25000000 },
        { id: 'a12', name: '黑洞能量抽提站', desc: '每秒產出 +150,000,000', cost: 15000000000, mps: 150000000 },
        { id: 'a13', name: '星系貿易樞紐網', desc: '每秒產出 +1 B', cost: 100000000000, mps: 1000000000 },
        { id: 'a14', name: '暗物質提煉工廠', desc: '每秒產出 +7 B', cost: 800000000000, mps: 7000000000 },
        { id: 'a15', name: '量子並行複製矩陣', desc: '每秒產出 +50 B', cost: 6000000000000, mps: 50000000000 },
        { id: 'a16', name: '時空裂縫資源站', desc: '每秒產出 +350 B', cost: 50000000000000, mps: 350000000000 },
        { id: 'a17', name: '超維度造幣文明', desc: '每秒產出 +2.5 T', cost: 400000000000000, mps: 2500000000000 },
        { id: 'a18', name: '現實法則重構網', desc: '每秒產出 +20 T', cost: 3500000000000000, mps: 20000000000000 },
        { id: 'a19', name: '創世大爆炸發生器', desc: '每秒產出 +150 T', cost: 30000000000000000, mps: 150000000000000 },
        { id: 'a20', name: '無限多元宇宙心臟', desc: '每秒產出 +1,200 T', cost: 250000000000000000, mps: 1200000000000000 }
    ];

    // 3. 科技研發升級 (20 個)
    const techList = [
        { id: 't1', name: '暴擊訓練', desc: '暴擊率 +2%', cost: 200, effect: () => gameState.critChance += 0.02 },
        { id: 't2', name: '槓桿投資', desc: '暴擊傷害倍率 +0.5x', cost: 1000, effect: () => gameState.critMulti += 0.5 },
        { id: 't3', name: '大數據精準行銷', desc: '被動收益提升 10%', cost: 5000, effect: () => gameState.mpsMultiplier = (gameState.mpsMultiplier||1) * 1.1 },
        { id: 't4', name: '微型晶片強化', desc: '暴擊率 +3%', cost: 20000, effect: () => gameState.critChance += 0.03 },
        { id: 't5', name: '自動化物流AI', desc: '被動收益提升 15%', cost: 100000, effect: () => gameState.mpsMultiplier = (gameState.mpsMultiplier||1) * 1.15 },
        { id: 't6', name: '高頻交易演算法', desc: '暴擊傷害倍率 +1.0x', cost: 500000, effect: () => gameState.critMulti += 1.0 },
        { id: 't7', name: '區塊鏈智能合約', desc: '被動收益提升 20%', cost: 2500000, effect: () => gameState.mpsMultiplier = (gameState.mpsMultiplier||1) * 1.20 },
        { id: 't8', name: '金礦幸運磁場', desc: '黃金硬幣出現率翻倍', cost: 12000000, effect: () => window.goldRate = (window.goldRate||0.005) * 2 },
        { id: 't9', name: '超導體電力網', desc: '被動收益提升 30%', cost: 80000000, effect: () => gameState.mpsMultiplier = (gameState.mpsMultiplier||1) * 1.30 },
        { id: 't10', name: '奇點人工智慧', desc: '暴擊率 +10%', cost: 500000000, effect: () => gameState.critChance += 0.10 },
        // 新增 10 個科技升級
        { id: 't11', name: '解鎖硬體自動點擊器', desc: '開啟自動點擊工具列', cost: 5000, effect: () => gameState.hasAutoClicker = true },
        { id: 't12', name: '熱血狂熱強化', desc: '熱血狂熱期間倍率翻倍', cost: 3000000, effect: () => window.frenzyPower = 2 },
        { id: 't13', name: '股市內線消息', desc: '股票交易手續費減半', cost: 15000000, effect: () => window.stockFeeDiscount = true },
        { id: 't14', name: '連擊延遲優化', desc: '連擊計量表衰退速度減半', cost: 50000000, effect: () => window.comboDecayDelay = 2000 },
        { id: 't15', name: '黃金爆發狂潮', desc: '黃金硬幣點擊效果 +100%', cost: 200000000, effect: () => window.goldCoinMulti = 2 },
        { id: 't16', name: '被動收益複利學', desc: '被動收益提升 25%', cost: 1000000000, effect: () => gameState.mpsMultiplier = (gameState.mpsMultiplier||1) * 1.25 },
        { id: 't17', name: '超級連動共振', desc: '連擊加成效果提升至 2 倍', cost: 10000000000, effect: () => window.comboPower = 2 },
        { id: 't18', name: '終極暴擊突破', desc: '暴擊傷害倍率 +3.0x', cost: 100000000000, effect: () => gameState.critMulti += 3.0 },
        { id: 't19', name: '量子金錢複製術', desc: '被動收益提升 50%', cost: 1000000000000, effect: () => gameState.mpsMultiplier = (gameState.mpsMultiplier||1) * 1.5 },
        { id: 't20', name: '全領域經濟帝國', desc: '暴擊率 +15%', cost: 10000000000000, effect: () => gameState.critChance += 0.15 }
    ];

    // 4. 聲望樹天賦升級 (全新 10 個，使用轉生水晶購買)
    const presTechList = [
        { id: 'pt1', name: '星光開局', desc: '轉生後基礎金錢直接給予 $10,000', cost: 1, effect: () => window.startBonus = 10000 },
        { id: 'pt2', name: '水晶共振', desc: '每顆水晶的加成效益提升至 +20%', cost: 3, effect: () => window.crystalBonusRate = 0.20 },
        { id: 'pt3', name: '永恆點擊', desc: '手動點擊基礎值永久 +100', cost: 5, effect: () => window.permClickBase = 100 },
        { id: 'pt4', name: '聲望連擊', desc: '連擊上限提升至 200x', cost: 10, effect: () => window.maxCombo = 200 },
        { id: 'pt5', name: '幸運星雲', desc: '黃金硬幣觸發機率提升 3 倍', cost: 20, effect: () => window.goldRate = (window.goldRate||0.005) * 3 },
        { id: 'pt6', name: '超空時空資產', desc: '所有資產購買成本增長率降低 (1.15x -> 1.12x)', cost: 50, effect: () => window.costGrowthRate = 1.12 },
        { id: 'pt7', name: '自動化之神', desc: '自動點擊器每秒點擊次數 +5 次', cost: 100, effect: () => window.autoClickSpeed = 15 },
        { id: 'pt8', name: '暴擊天界', desc: '暴擊率永久 +10%', cost: 250, effect: () => gameState.critChance += 0.10 },
        { id: 'pt9', name: '股市操盤術', desc: '股市獲利額外增加 30%', cost: 500, effect: () => window.stockBonus = 1.30 },
        { id: 'pt10', name: '無限富豪真諦', desc: '總收益永久翻倍 (x2)', cost: 1000, effect: () => window.godMultiplier = 2 }
    ];

    // 成就清單
    const achievementsList = [
        { id: 'ach1', name: '第一桶金', desc: '累積賺取 $100', check: () => gameState.totalMoneyEarned >= 100 },
        { id: 'ach2', name: '瘋狂點擊者', desc: '手動點擊超過 100 次', check: () => gameState.totalClickCount >= 100 },
        { id: 'ach3', name: '千錘百煉', desc: '達成 50 連擊', check: () => comboCount >= 50 },
        { id: 'ach4', name: '百萬富翁', desc: '持有金額達到 $1,000,000', check: () => gameState.money >= 1000000 },
        { id: 'ach5', name: '轉生者', desc: '完成首次轉生', check: () => gameState.prestigeCrystals > 0 },
        { id: 'ach6', name: '股市狼人', desc: '在股市持股超過 100 股', check: () => Object.values(gameState.stocks).reduce((a,b)=>a+b.shares,0) >= 100 },
        { id: 'ach7', name: '熱血湧現', desc: '進入狂熱模式 (Frenzy)', check: () => isFrenzy }
    ];

    // -------------------------------------------------------------
    // UI 初始化與渲染
    // -------------------------------------------------------------
    function initUI() {
        renderList(clickUpgradesList, 'upgrades', 'buyClickUpgrade');
        renderList(assetsList, 'assets', 'buyAsset');
        renderList(techList, 'tech', 'buyTech');
        renderPresList(presTechList, 'presTree', 'buyPresTech');
        renderStockUI();
        renderAchievements();
    }

    function renderList(list, targetId, buyFuncName) {
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
                    $<span id="cost-${item.id}">${formatNumber(item.cost)}</span>
                </button>
            `;
            container.appendChild(div);
        });
    }

    function renderPresList(list, targetId, buyFuncName) {
        const container = document.getElementById(targetId);
        container.innerHTML = '<h3>✨ 聲望樹 (使用轉生水晶天賦點數)</h3>';
        list.forEach(item => {
            const div = document.createElement('div');
            div.className = 'item-card';
            div.innerHTML = `
                <div class="item-info">
                    <div class="item-title">${item.name}</div>
                    <div class="item-desc">${item.desc}</div>
                </div>
                <button class="buy-btn" id="btn-${item.id}" onclick="${buyFuncName}('${item.id}')" style="background:#7c3aed;">
                    💎 <span id="cost-${item.id}">${item.cost}</span> 水晶
                </button>
            `;
            container.appendChild(div);
        });
    }

    function renderStockUI() {
        const container = document.getElementById('stockList');
        container.innerHTML = '';
        for (let key in gameState.stocks) {
            let s = gameState.stocks[key];
            let div = document.createElement('div');
            div.className = 'stock-box';
            div.innerHTML = `
                <div style="display:flex; justify-content:space-between; font-weight:bold;">
                    <span>${s.name} (${key})</span>
                    <span id="stock-price-${key}">$${s.price.toFixed(2)}</span>
                </div>
                <div style="display:flex; justify-content:space-between; margin-top:4px; font-size:0.8em; color:#94a3b8;">
                    <span>持有股份: <b id="stock-shares-${key}" style="color:#38bdf8">${s.shares}</b></span>
                    <div>
                        <button class="buy-btn" style="padding:2px 6px; font-size:0.75em;" onclick="buyStock('${key}')">買入 1 股</button>
                        <button class="buy-btn" style="padding:2px 6px; font-size:0.75em; background:#dc2626;" onclick="sellStock('${key}')">賣出 1 股</button>
                    </div>
                </div>
            `;
            container.appendChild(div);
        }
    }

    function renderAchievements() {
        const container = document.getElementById('achieveList');
        container.innerHTML = '<h3>🏆 成就勳章</h3>';
        achievementsList.forEach(a => {
            const unlocked = gameState.achievements[a.id];
            const span = document.createElement('span');
            span.className = `achieve-badge ${unlocked ? 'unlocked' : ''}`;
            span.innerText = `${unlocked ? '✓ ' : '🔒 '}${a.name}: ${a.desc}`;
            container.appendChild(span);
        });
    }

    // 數字格式化顯示（支援 K, M, B, T）
    function formatNumber(num) {
        if (num >= 1e15) return (num / 1e15).toFixed(2) + ' Q';
        if (num >= 1e12) return (num / 1e12).toFixed(2) + ' T';
        if (num >= 1e9) return (num / 1e9).toFixed(2) + ' B';
        if (num >= 1e6) return (num / 1e6).toFixed(2) + ' M';
        if (num >= 1e3) return (num / 1e3).toFixed(1) + ' K';
        return Math.floor(num).toLocaleString();
    }

    // -------------------------------------------------------------
    // 轉生聲望系統核心（依據歷史累積總金額公式計算）
    // -------------------------------------------------------------
    function calculatePrestigeCrystals(totalEarned) {
        if (totalEarned < 100000) return 0;
        // 公式: 立方根(總賺取 / 100000)
        return Math.floor(Math.cbrt(totalEarned / 100000));
    }

    function getPrestigeMultiplier() {
        let rate = window.crystalBonusRate || 0.10; // 每顆水晶預設 +10% 收益
        return 1 + (gameState.prestigeCrystals * rate);
    }

    function prestige() {
        let pendingCrystals = calculatePrestigeCrystals(gameState.totalMoneyEarned) - gameState.prestigeCrystals;
        if (pendingCrystals > 0) {
            if (confirm(`確定要轉生嗎？您將獲得 ${pendingCrystals} 顆【聲望水晶】！\n這會重置金額與普通升級，但水晶將給予永久巨大加成！`)) {
                gameState.prestigeCrystals += pendingCrystals;
                
                // 重置部分數據
                gameState.money = window.startBonus || 0;
                gameState.upgrades = {};
                gameState.assets = {};
                gameState.techs = {};
                gameState.mpsMultiplier = 1;
                gameState.critChance = 0.05;
                gameState.critMulti = 1.5;
                
                updateDisplay();
                alert(`轉生成功！獲得 ${pendingCrystals} 顆聲望水晶！`);
            }
        } else {
            alert("目前累積的總收益不足以賺取額外的聲望水晶！請繼續賺錢！");
        }
    }

    // -------------------------------------------------------------
    // 遊戲核心邏輯 (連擊、狂熱、點擊、被動)
    // -------------------------------------------------------------
    function calculateClickValue() {
        let base = 1 + (window.permClickBase || 0);
        clickUpgradesList.forEach(item => {
            let count = gameState.upgrades[item.id] || 0;
            base += item.val * count;
        });
        
        // 連擊加成 (每個 combo +1% ~ 2%)
        let comboMult = 1 + (comboCount * 0.01 * (window.comboPower || 1));
        let godMult = window.godMultiplier || 1;

        return base * getPrestigeMultiplier() * comboMult * frenzyMultiplier * godMult;
    }

    function calculateMPS() {
        let baseMps = 0;
        let growth = window.costGrowthRate || 1.15;
        assetsList.forEach(item => {
            let count = gameState.assets[item.id] || 0;
            baseMps += item.mps * count;
        });
        let mult = (gameState.mpsMultiplier || 1) * getPrestigeMultiplier() * eventMultiplier * (window.godMultiplier || 1);
        return baseMps * mult;
    }

    function clickMoney(e) {
        gameState.totalClickCount++;
        
        // 1. 連擊處理
        comboCount = Math.min(comboCount + 1, window.maxCombo || 100);
        clearTimeout(comboTimer);
        comboTimer = setTimeout(() => {
            comboCount = 0;
            updateDisplay();
        }, window.comboDecayDelay || 1200);

        // 2. 熱血狂熱模式觸發 (50連擊機率觸發狂熱)
        if (comboCount >= 30 && Math.random() < 0.08 && !isFrenzy) {
            triggerFrenzy();
        }

        // 3. 暴擊判定
        let isCrit = Math.random() < gameState.critChance;
        let gained = calculateClickValue();
        if (isCrit) gained *= gameState.critMulti;

        gameState.money += gained;
        gameState.totalMoneyEarned += gained;

        // 4. 特效
        if (e && e.clientX) {
            showFloatingText(e.clientX, e.clientY, `+$${formatNumber(gained)}${isCrit ? ' 暴擊!' : ''}`, isCrit);
        }
        updateDisplay();
    }

    // 熱血狂熱模式 (Frenzy)
    function triggerFrenzy() {
        isFrenzy = true;
        frenzyMultiplier = 7 * (window.frenzyPower || 1);
        document.getElementById('frenzyOverlay').style.display = 'block';
        showFloatingText(window.innerWidth / 2, window.innerHeight / 3, '🔥進入狂熱模式！收益 7 倍！🔥', true);

        setTimeout(() => {
            isFrenzy = false;
            frenzyMultiplier = 1;
            document.getElementById('frenzyOverlay').style.display = 'none';
        }, 8000);
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
        let growth = window.costGrowthRate || 1.15;
        let currentCost = Math.floor(item.cost * Math.pow(growth, count));
        
        if (gameState.money >= currentCost) {
            gameState.money -= currentCost;
            gameState.upgrades[id] = count + 1;
            updateDisplay();
        }
    }

    function buyAsset(id) {
        let item = assetsList.find(x => x.id === id);
        let count = gameState.assets[id] || 0;
        let growth = window.costGrowthRate || 1.15;
        let currentCost = Math.floor(item.cost * Math.pow(growth, count));

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

    function buyPresTech(id) {
        let item = presTechList.find(x => x.id === id);
        if (!gameState.presTechs[id] && gameState.prestigeCrystals >= item.cost) {
            gameState.prestigeCrystals -= item.cost;
            gameState.presTechs[id] = true;
            item.effect();
            updateDisplay();
        }
    }

    // -------------------------------------------------------------
    // 股市模擬與小功能機制
    // -------------------------------------------------------------
    function updateStocks() {
        for (let key in gameState.stocks) {
            let s = gameState.stocks[key];
            let change = (Math.random() - 0.48) * 0.1 * s.price; // 隨機波動
            s.price = Math.max(1, s.price + change);
            s.history.push(s.price);
            if (s.history.length > 20) s.history.shift();

            let priceEl = document.getElementById(`stock-price-${key}`);
            if (priceEl) priceEl.innerText = `$${s.price.toFixed(2)}`;
        }
    }

    function buyStock(key) {
        let s = gameState.stocks[key];
        let fee = window.stockFeeDiscount ? 1.01 : 1.05;
        let cost = s.price * fee;
        if (gameState.money >= cost) {
            gameState.money -= cost;
            s.shares++;
            updateDisplay();
        }
    }

    function sellStock(key) {
        let s = gameState.stocks[key];
        if (s.shares > 0) {
            let gain = s.price * (window.stockBonus || 1.0);
            gameState.money += gain;
            gameState.totalMoneyEarned += gain;
            s.shares--;
            updateDisplay();
        }
    }

    function toggleAutoClicker() {
        if (!gameState.hasAutoClicker) {
            alert("需要先解鎖科技【解鎖硬體自動點擊器】！");
            return;
        }
        gameState.autoClickerActive = !gameState.autoClickerActive;
        document.getElementById('autoClickerBtn').innerText = `🤖 自動點擊器: ${gameState.autoClickerActive ? '開' : '關'}`;
    }

    // -------------------------------------------------------------
    // 隨機事件與硬幣
    // -------------------------------------------------------------
    function spawnGoldenCoin() {
        if (Math.random() < (window.goldRate || 0.005)) {
            if (document.getElementById('goldCoin')) return;
            const coin = document.createElement('div');
            coin.id = 'goldCoin';
            coin.className = 'golden-coin';
            coin.innerText = '💰';
            coin.style.left = `${Math.random() * 75 + 10}%`;
            coin.style.top = `${Math.random() * 65 + 15}%`;
            
            coin.onclick = () => {
                let bonus = (calculateMPS() * 20 + calculateClickValue() * 30 + 100) * (window.goldCoinMulti || 1);
                gameState.money += bonus;
                gameState.totalMoneyEarned += bonus;
                showFloatingText(window.innerWidth / 2, window.innerHeight / 2, `金幣爆發: +$${formatNumber(bonus)}!`, true);
                coin.remove();
            };
            document.body.appendChild(coin);
            setTimeout(() => { if (coin.parentNode) coin.remove(); }, 4500);
        }
    }

    function triggerRandomEvent() {
        if (Math.random() < 0.04) {
            const events = [
                { name: '中央銀行降息！被動收益提升 2 倍 (15秒)', mult: 2, duration: 15000 },
                { name: '全球供應鏈受阻！被動收益降低至 0.5 倍 (10秒)', mult: 0.5, duration: 10000 },
                { name: '超級牛市爆發！被動收益提升 3 倍 (10秒)', mult: 3, duration: 10000 }
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

    // -------------------------------------------------------------
    // 存檔與系統控制
    // -------------------------------------------------------------
    function saveGame() {
        gameState.lastOnline = Date.now();
        localStorage.setItem('clicker_v2_save', JSON.stringify(gameState));
    }

    function loadGame() {
        let saved = localStorage.getItem('clicker_v2_save');
        if (saved) {
            try {
                let loaded = JSON.parse(saved);
                gameState = { ...gameState, ...loaded };
                
                // 離線收益結算
                let now = Date.now();
                let offlineSeconds = Math.floor((now - (gameState.lastOnline || now)) / 1000);
                if (offlineSeconds > 5) {
                    let earned = offlineSeconds * calculateMPS() * 0.5;
                    if (earned > 0) {
                        gameState.money += earned;
                        gameState.totalMoneyEarned += earned;
                        alert(`歡迎回來！離線 ${offlineSeconds} 秒期間獲得了 $${formatNumber(earned)} 收益！`);
                    }
                }
            } catch(e) { console.error("載入失敗", e); }
        }
    }

    function resetGame() {
        if (confirm("⚠️ 確定要徹底重置所有進度嗎？這將無法復原！")) {
            localStorage.removeItem('clicker_v2_save');
            location.reload();
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
                gameState = JSON.parse(atob(str));
                updateDisplay();
                alert("存檔匯入成功！");
            } catch(e) { alert("無效的存檔代碼！"); }
        }
    }

    function switchTab(tabId) {
        document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
        document.querySelectorAll('.tab-btn').forEach(el => el.classList.remove('active'));
        document.getElementById(tabId).classList.add('active');
        event.target.classList.add('active');
    }

    // -------------------------------------------------------------
    // 介面刷寫 (Update Loop)
    // -------------------------------------------------------------
    function updateDisplay() {
        let currentMps = calculateMPS();
        let currentClick = calculateClickValue();

        document.getElementById('money').innerText = '$' + formatNumber(gameState.money);
        document.getElementById('mps').innerText = `每秒被動: $${formatNumber(currentMps)}${eventMultiplier !== 1 ? ` (x${eventMultiplier})` : ''}`;
        document.getElementById('mpc').innerText = `每次點擊: $${formatNumber(currentClick)}`;
        document.getElementById('critStat').innerText = `暴擊率: ${(gameState.critChance * 100).toFixed(0)}% (${gameState.critMulti}x)`;
        document.getElementById('comboStat').innerText = comboCount > 0 ? `連擊: ${comboCount}x (${(comboCount*1).toFixed(0)}%加成)` : '';

        // 轉生資訊 UI
        let pendingCrystals = calculatePrestigeCrystals(gameState.totalMoneyEarned) - gameState.prestigeCrystals;
        document.getElementById('prestigeStat').innerText = `持有聲望水晶: ${gameState.prestigeCrystals} (永久 +${((getPrestigeMultiplier()-1)*100).toFixed(0)}%)`;
        document.getElementById('nextPrestigeStat').innerText = `轉生可獲得: +${Math.max(0, pendingCrystals)} 顆水晶`;

        // 手動升級 UI
        let growth = window.costGrowthRate || 1.15;
        clickUpgradesList.forEach(item => {
            let count = gameState.upgrades[item.id] || 0;
            let cost = Math.floor(item.cost * Math.pow(growth, count));
            document.getElementById(`cost-${item.id}`).innerText = formatNumber(cost);
            document.getElementById(`count-${item.id}`).innerText = count > 0 ? `[Lv.${count}]` : '';
            document.getElementById(`btn-${item.id}`).disabled = gameState.money < cost;
        });

        // 被動資產 UI
        assetsList.forEach(item => {
            let count = gameState.assets[item.id] || 0;
            let cost = Math.floor(item.cost * Math.pow(growth, count));
            document.getElementById(`cost-${item.id}`).innerText = formatNumber(cost);
            document.getElementById(`count-${item.id}`).innerText = count > 0 ? `[x${count}]` : '';
            document.getElementById(`btn-${item.id}`).disabled = gameState.money < cost;
        });

        // 科技 UI
        techList.forEach(item => {
            let bought = gameState.techs[item.id];
            let btn = document.getElementById(`btn-${item.id}`);
            if (bought) {
                btn.disabled = true; btn.innerText = '已解鎖';
            } else {
                btn.disabled = gameState.money < item.cost;
            }
        });

        // 聲望樹 UI
        presTechList.forEach(item => {
            let bought = gameState.presTechs[item.id];
            let btn = document.getElementById(`btn-${item.id}`);
            if (bought) {
                btn.disabled = true; btn.innerText = '已解鎖';
            } else {
                btn.disabled = gameState.prestigeCrystals < item.cost;
            }
        });

        // 股票股份 UI
        for (let key in gameState.stocks) {
            let sEl = document.getElementById(`stock-shares-${key}`);
            if (sEl) sEl.innerText = gameState.stocks[key].shares;
        }

        // 詳細統計 UI
        document.getElementById('statsDetail').innerHTML = `
            📊 <b>數據統計</b><br>
            • 歷史累積總賺取金額: $${formatNumber(gameState.totalMoneyEarned)}<br>
            • 歷史手動點擊次數: ${gameState.totalClickCount.toLocaleString()} 次
        `;

        // 檢查成就
        achievementsList.forEach(a => {
            if (!gameState.achievements[a.id] && a.check()) {
                gameState.achievements[a.id] = true;
                showFloatingText(window.innerWidth / 2, 80, `🏆 解鎖成就: ${a.name}`, true);
                renderAchievements();
            }
        });
    }

    // -------------------------------------------------------------
    // 主遊戲 Tick 循環
    // -------------------------------------------------------------
    initUI();
    loadGame();
    updateDisplay();

    // 每 0.1 秒執行每秒收益與自動點擊
    let autoClickCounter = 0;
    setInterval(() => {
        let mps = calculateMPS();
        let gained = mps / 10;
        gameState.money += gained;
        gameState.totalMoneyEarned += gained;

        // 自動點擊器硬體觸發
        if (gameState.autoClickerActive) {
            autoClickCounter++;
            let speed = window.autoClickSpeed || 10;
            if (autoClickCounter >= (10 / speed)) {
                clickMoney();
                autoClickCounter = 0;
            }
        }

        updateDisplay();
    }, 100);

    // 每 3 秒更新股市與隨機事件
    setInterval(() => {
        updateStocks();
        spawnGoldenCoin();
        triggerRandomEvent();
    }, 3000);

    // 每 10 秒自動存檔
    setInterval(saveGame, 10000);
</script>
</body>
</html>
