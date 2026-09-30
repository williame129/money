<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>金錢帝國 3.3 - 終極神階次方膨脹版</title>
    <style>
        * { box-sizing: border-box; font-family: 'Segoe UI', Microsoft JhengHei, sans-serif; }
        body { background-color: #050811; color: #f8fafc; margin: 0; padding: 12px; display: flex; justify-content: center; }
        .game-wrapper { width: 100%; max-width: 1200px; display: grid; grid-template-columns: 340px 1fr; gap: 15px; }
        @media (max-width: 900px) { .game-wrapper { grid-template-columns: 1fr; } }
        
        .panel { background-color: #0f172a; border-radius: 12px; padding: 16px; border: 1px solid #1e293b; position: relative; }
        
        h1, h2, h3 { margin-top: 0; color: #38bdf8; text-align: center; }
        .stats-box { background: #050811; padding: 12px; border-radius: 8px; text-align: center; margin-bottom: 10px; border: 1px solid #1e293b; }
        .money { font-size: 1.8em; color: #4ade80; font-weight: bold; word-break: break-all; }
        .sub-stat { font-size: 0.82em; color: #94a3b8; margin-top: 2px; }
        
        /* 點擊按鈕與特效 */
        .click-area { text-align: center; position: relative; margin: 12px 0; }
        .big-btn {
            background: linear-gradient(135deg, #22c55e, #15803d); color: white; border: none;
            width: 130px; height: 130px; border-radius: 50%; font-size: 1.3em; font-weight: bold;
            cursor: pointer; box-shadow: 0 8px 20px rgba(34, 197, 94, 0.4); transition: transform 0.05s;
            user-select: none;
        }
        .big-btn:active { transform: scale(0.92); }
        .floating-text { position: absolute; color: #4ade80; font-weight: bold; pointer-events: none; animation: floatUp 0.8s ease-out forwards; z-index: 100; font-size: 1.1em; }
        @keyframes floatUp { 0% { opacity: 1; transform: translateY(0); } 100% { opacity: 0; transform: translateY(-40px); } }

        /* 標籤頁面 */
        .tabs { display: flex; gap: 4px; margin-bottom: 10px; flex-wrap: wrap; }
        .tab-btn { flex: 1; min-width: 65px; padding: 6px 3px; background: #1e293b; border: none; color: #fff; cursor: pointer; border-radius: 6px; font-weight: bold; font-size: 0.78em; }
        .tab-btn.active { background: #0284c7; }
        .tab-content { display: none; max-height: 560px; overflow-y: auto; padding-right: 5px; }
        .tab-content.active { display: block; }

        /* 列表項目 */
        .item-card {
            background: #050811; border: 1px solid #1e293b; border-radius: 8px; padding: 8px 10px;
            margin-bottom: 6px; display: flex; justify-content: space-between; align-items: center;
        }
        .item-title { font-weight: bold; color: #f3f4f6; font-size: 0.88em; }
        .item-desc { font-size: 0.75em; color: #9ca3af; }
        .buy-btn {
            background: #0284c7; border: none; color: white; padding: 6px 10px; border-radius: 6px;
            cursor: pointer; font-weight: bold; font-size: 0.78em; transition: 0.2s; min-width: 90px;
        }
        .buy-btn:hover:not(:disabled) { background: #0369a1; }
        .buy-btn:disabled { background: #334155; color: #64748b; cursor: not-allowed; opacity: 0.5; }

        /* 彈出視窗 Modal */
        .modal-overlay {
            position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
            background: rgba(0, 0, 0, 0.85); display: none; justify-content: center; align-items: center; z-index: 1000;
        }
        .story-box {
            background: #0f172a; border: 2px solid #38bdf8; border-radius: 12px; padding: 20px;
            width: 90%; max-width: 500px; box-shadow: 0 0 25px rgba(56, 189, 248, 0.4); text-align: left;
        }
        .story-title { font-size: 1.2em; font-weight: bold; color: #facc15; margin-bottom: 10px; border-bottom: 1px solid #334155; padding-bottom: 5px; }
        .story-text { font-size: 0.95em; line-height: 1.5; color: #e2e8f0; margin-bottom: 20px; }

        .flex-between { display: flex; justify-content: space-between; gap: 6px; margin-top: 6px; }
        .action-btn { flex: 1; padding: 6px; background: #334155; border: none; color: white; border-radius: 6px; cursor: pointer; font-size: 0.78em; }
        .action-btn:hover { background: #475569; }
        
        .mini-progress { width: 100%; background: #1e293b; border-radius: 4px; height: 8px; margin-top: 4px; overflow: hidden; }
        .mini-bar { height: 100%; background: #38bdf8; width: 0%; transition: width 0.1s linear; }

        .toggle-box {
            background: #1e293b; border-radius: 8px; padding: 12px; margin-bottom: 15px; display: flex; justify-content: space-between; align-items: center;
        }
    </style>
</head>
<body>

<!-- 故事劇情 Modal -->
<div class="modal-overlay" id="storyModal">
    <div class="story-box">
        <div class="story-title" id="storyTitle">📖 劇情章節</div>
        <div class="story-text" id="storyText">劇情內容...</div>
        <div style="display:flex; justify-content:space-between;">
            <button class="buy-btn" style="background:#64748b;" onclick="closeStory(true)">跳過劇情 (Skip)</button>
            <button class="buy-btn" style="background:#16a34a;" onclick="closeStory(false)">繼續冒險</button>
        </div>
    </div>
</div>

<!-- 暫停選單 / 設定 Modal -->
<div class="modal-overlay" id="pauseModal">
    <div class="story-box" style="border-color:#facc15;">
        <div class="story-title" style="color:#facc15; text-align:center;">⏸️ 遊戲暫停與設定</div>
        
        <div class="toggle-box">
            <div>
                <div style="font-weight:bold; color:#fff;">數字顯示模式</div>
                <div style="font-size:0.75em; color:#94a3b8;" id="numFormatDesc">當前：單位表示法 (1.50 M, 3.20 T)</div>
            </div>
            <button class="buy-btn" style="background:#eab308; color:#000;" onclick="toggleNumberFormat()">切換模式</button>
        </div>

        <div style="text-align:center; margin-top:15px;">
            <button class="buy-btn" style="background:#22c55e; width:100%; padding:10px; font-size:1em;" onclick="togglePause()">▶️ 繼續遊戲</button>
        </div>
    </div>
</div>

<div class="game-wrapper">
    <!-- 左側：控制面板 -->
    <div class="panel">
        <h1>🌌 金錢帝國 3.3</h1>

        <div class="stats-box">
            <div class="money" id="money">$0</div>
            <div class="sub-stat" id="mps">每秒被動: $0</div>
            <div class="sub-stat" id="mpc">每次點擊: $1</div>
            <div class="sub-stat" id="critStat">暴擊率: 5% (2.0x)</div>
        </div>

        <div class="click-area">
            <button class="big-btn" id="mainBtn" onclick="clickMoney(event)">點擊賺錢</button>
        </div>

        <!-- 重生金幣 / 聲望水晶 -->
        <div style="background:#050811; padding:8px; border-radius:8px; border:1px solid #1e293b; margin-bottom:8px;">
            <div style="font-weight:bold; color:#38bdf8; font-size:0.82em;">💎 重生聲望 (Rebirth)</div>
            <div class="sub-stat" id="prestigeStat">重生金幣: 0 (加成 +0%)</div>
            <div class="sub-stat" id="nextPrestigeStat">重置可得: +0 重生金幣</div>
            <button class="buy-btn" style="width:100%; margin-top:4px; background:#dc2626;" onclick="prestige()">轉生領取重生金幣</button>
        </div>

        <!-- 超越金幣 / 暗物質 -->
        <div style="background:#1e1b4b; padding:8px; border-radius:8px; border:1px solid #4338ca; margin-bottom:8px;">
            <div style="font-weight:bold; color:#c084fc; font-size:0.82em;">✨ 超越神殿 (Transcendence)</div>
            <div class="sub-stat" style="color:#e9d5ff;" id="transStat">超越金幣: 0 (加成 +0%)</div>
            <div class="sub-stat" style="color:#a5b4fc;" id="nextTransStat">需求: $1,000 T 歷史總金額</div>
            <button class="buy-btn" style="width:100%; margin-top:4px; background:#7c3aed;" onclick="transcend()">破除維度：超越升華</button>
        </div>

        <!-- 系統管理 -->
        <div class="flex-between">
            <button class="action-btn" style="background:#eab308; color:#000; font-weight:bold;" onclick="togglePause()">⏸️ 暫停 / 設定</button>
            <button class="action-btn" onclick="toggleAutoClicker()" id="autoClickerBtn">🤖 自動點擊: 關</button>
        </div>
        <div class="flex-between">
            <button class="action-btn" onclick="saveGame()">💾 存檔</button>
            <button class="action-btn" onclick="exportSave()">📤 匯出</button>
            <button class="action-btn" onclick="importSave()">📥 匯入</button>
            <button class="action-btn" onclick="openStoryLog()">📖 劇情</button>
        </div>
    </div>

    <!-- 右側：分頁面板 -->
    <div class="panel">
        <div class="tabs">
            <button class="tab-btn active" onclick="switchTab('upgrades', event)">手動升級(40)</button>
            <button class="tab-btn" onclick="switchTab('assets', event)">被動資產(40)</button>
            <button class="tab-btn" onclick="switchTab('tech', event)">科技研發(20)</button>
            <button class="tab-btn" onclick="switchTab('presTree', event)">重生天賦(30)</button>
            <button class="tab-btn" onclick="switchTab('transTree', event)">超越天賦(30)</button>
            <button class="tab-btn" onclick="switchTab('minigame', event)">金錢挖礦</button>
            <button class="tab-btn" onclick="switchTab('stock', event)">股市投機</button>
        </div>

        <div id="upgrades" class="tab-content active"></div>
        <div id="assets" class="tab-content"></div>
        <div id="tech" class="tab-content"></div>
        <div id="presTree" class="tab-content"></div>
        <div id="transTree" class="tab-content"></div>

        <!-- 金錢挖礦小遊戲 -->
        <div id="minigame" class="tab-content">
            <h3>⛏️ 數字金錢挖礦機</h3>
            <p style="font-size:0.8em; color:#9ca3af;">自動計算哈希值，進度滿即可獲得大量爆發現金！</p>
            <div style="background:#050811; padding:12px; border-radius:8px; border:1px solid #1e293b; text-align:center;">
                <div style="font-size:1.1em; color:#facc15; font-weight:bold; margin-bottom:6px;">當前算力: <span id="minePower">1</span> GH/s</div>
                <div class="mini-progress"><div class="mini-bar" id="mineBar"></div></div>
                <button class="buy-btn" style="margin-top:10px; background:#eab308; color:#000;" onclick="upgradeMiner()">升級鑽頭 (需求: $<span id="mineUpgradeCost">10,000</span>)</button>
            </div>
        </div>

        <!-- 股市 -->
        <div id="stock" class="tab-content">
            <h3>📈 虛擬股市投機</h3>
            <div id="stockList"></div>
        </div>
    </div>
</div>

<script>
    // -------------------------------------------------------------
    // 全局遊戲狀態
    // -------------------------------------------------------------
    let isPaused = false;
    let numberFormatMode = 'short'; // 'short' (單位) 或 'full' (完整數字)

    let gameState = {
        money: 0,
        totalMoneyEarned: 0,
        totalClickCount: 0,
        critChance: 0.05,
        critMulti: 2.0,
        prestigeCrystals: 0,
        darkMatter: 0,
        transcendCount: 0,
        minePower: 1,
        mineProgress: 0,
        storyProgress: 0,
        skipStory: false,
        autoSpeed: 1000,
        hasAutoClicker: false,
        autoClickerActive: false,
        upgrades: {},
        assets: {},
        techs: {},
        presTechs: {},
        transTechs: {},
        stocks: {
            'TECH': { name: '高科技指數', price: 100, shares: 0 },
            'ENERGY': { name: '能源巨頭', price: 50, shares: 0 },
            'COIN': { name: '加密概念股', price: 10, shares: 0 }
        }
    };

    let globalTechMpsMult = 1;
    let globalTechClickMult = 1;

    // -------------------------------------------------------------
    // 數字格式化（支援 單位表示 / 完整數字表示）
    // -------------------------------------------------------------
    function formatNumber(num) {
        if (num === null || num === undefined || isNaN(num)) return '0';
        
        if (numberFormatMode === 'full') {
            return Math.floor(num).toLocaleString('en-US');
        }

        if (num >= 1e36) return (num / 1e36).toFixed(2) + ' Dc';
        if (num >= 1e33) return (num / 1e33).toFixed(2) + ' No';
        if (num >= 1e30) return (num / 1e30).toFixed(2) + ' Oc';
        if (num >= 1e27) return (num / 1e27).toFixed(2) + ' Sp';
        if (num >= 1e24) return (num / 1e24).toFixed(2) + ' Sx';
        if (num >= 1e21) return (num / 1e21).toFixed(2) + ' Qi';
        if (num >= 1e18) return (num / 1e18).toFixed(2) + ' Q';
        if (num >= 1e15) return (num / 1e15).toFixed(2) + ' Quad';
        if (num >= 1e12) return (num / 1e12).toFixed(2) + ' T';
        if (num >= 1e9) return (num / 1e9).toFixed(2) + ' B';
        if (num >= 1e6) return (num / 1e6).toFixed(2) + ' M';
        if (num >= 1e3) return (num / 1e3).toFixed(1) + ' K';
        return Math.floor(num).toLocaleString();
    }

    function togglePause() {
        isPaused = !isPaused;
        document.getElementById('pauseModal').style.display = isPaused ? 'flex' : 'none';
    }

    function toggleNumberFormat() {
        numberFormatMode = (numberFormatMode === 'short') ? 'full' : 'short';
        document.getElementById('numFormatDesc').innerText = `當前：${numberFormatMode === 'short' ? '單位表示法 (1.50 M, 3.20 T)' : '完整數字表示法 (1,500,000)'}`;
        updateDisplay();
    }

    // -------------------------------------------------------------
    // 📖 故事劇情 (Story Chapters)
    // -------------------------------------------------------------
    const storyChapters = [
        { id: 0, req: 0, title: '第一章：起步與街頭', text: '你站在寒風 inclement 的街頭，口袋裡只有一張 1 美元鈔票。你下定決心要在這個資本世界建立自己的金錢帝國！' },
        { id: 1, req: 10000, title: '第二章：第一桶金', text: '累積了上萬資產，並擁有了屬於自己的連鎖小店。街坊鄰居都稱你為商業奇才！' },
        { id: 2, req: 1000000, title: '第三章：金融巨頭', text: '百萬資產達成！你跨足股市與商業地產，金融界的巨頭們開始注意到你的存在。' },
        { id: 3, req: 100000000, title: '第四章：全球供應鏈', text: '億萬帝國成立！你的業務橫跨晶圓與航天，全球經濟隨著你的決策而動盪。' },
        { id: 4, req: 10000000000, title: '第五章：跨星系經濟', text: '戴森球與小行星採礦基地源源不絕將財富輸送到你的帳戶，你成為太陽系的統治者。' },
        { id: 5, req: 1000000000000, title: '第六章：超脫物質界', text: '萬億財富！金錢不再只是數字，而是能撕裂維度的能量。準備進行【超越】升華！' }
    ];

    // 40 個手動升級單位
    const clickUpgradesList = [
        { id: 'c1', name: '鐵製滑鼠', desc: '每次點擊 +1', cost: 15, val: 1 },
        { id: 'c2', name: '人體工學握把', desc: '每次點擊 +5', cost: 100, val: 5 },
        { id: 'c3', name: '雙重點擊技巧', desc: '每次點擊 +25', cost: 500, val: 25 },
        { id: 'c4', name: '電競機械軸', desc: '每次點擊 +120', cost: 2500, val: 120 },
        { id: 'c5', name: '連點巨集程式', desc: '每次點擊 +600', cost: 10000, val: 600 },
        { id: 'c6', name: '神經傳導連線', desc: '每次點擊 +3,000', cost: 50000, val: 3000 },
        { id: 'c7', name: '量子點擊器', desc: '每次點擊 +15,000', cost: 250000, val: 15000 },
        { id: 'c8', name: '光速雷射感應', desc: '每次點擊 +80,000', cost: 1000000, val: 80000 },
        { id: 'c9', name: '時空裂隙點擊', desc: '每次點擊 +400,000', cost: 5000000, val: 400000 },
        { id: 'c10', name: '創世神之手', desc: '每次點擊 +2.5 M', cost: 25000000, val: 2500000 },
        { id: 'c11', name: '反物質觸控板', desc: '每次點擊 +15 M', cost: 1.5e8, val: 15000000 },
        { id: 'c12', name: '超光速脈衝點擊', desc: '每次點擊 +90 M', cost: 1e9, val: 90000000 },
        { id: 'c13', name: '高維度幾何點擊', desc: '每次點擊 +600 M', cost: 8e9, val: 600000000 },
        { id: 'c14', name: '引力波指尖發射器', desc: '每次點擊 +4.5 B', cost: 5e10, val: 4500000000 },
        { id: 'c15', name: '量子糾纏連點器', desc: '每次點擊 +30 B', cost: 4e11, val: 30000000000 },
        { id: 'c16', name: '時空逆轉點擊針', desc: '每次點擊 +200 B', cost: 3e12, val: 200000000000 },
        { id: 'c17', name: '亞原子裂變觸摸', desc: '每次點擊 +1.5 T', cost: 2.5e13, val: 1500000000000 },
        { id: 'c18', name: '現實法則寫入指', desc: '每次點擊 +12 T', cost: 2e14, val: 12000000000000 },
        { id: 'c19', name: '奇點爆破點擊儀', desc: '每次點擊 +90 T', cost: 1.8e15, val: 90000000000000 },
        { id: 'c20', name: '宇宙大爆發觸發按鈕', desc: '每次點擊 +700 T', cost: 1.5e16, val: 700000000000000 },
        { id: 'c21', name: '概率塌縮指套', desc: '每次點擊 +6 Q', cost: 1.2e17, val: 6e15 },
        { id: 'c22', name: '因果律改寫手套', desc: '每次點擊 +50 Q', cost: 1e18, val: 5e16 },
        { id: 'c23', name: '超弦震盪光束', desc: '每次點擊 +400 Q', cost: 9e18, val: 4e17 },
        { id: 'c24', name: '反熵能量點擊器', desc: '每次點擊 +3 Qi', cost: 8e19, val: 3e18 },
        { id: 'c25', name: '平行時空點擊網絡', desc: '每次點擊 +25 Qi', cost: 7e20, val: 2.5e19 },
        { id: 'c26', name: '神聖數學規律觸控', desc: '每次點擊 +200 Qi', cost: 6e21, val: 2e20 },
        { id: 'c27', name: '虛無實體印章', desc: '每次點擊 +1.5 Sx', cost: 5e22, val: 1.5e21 },
        { id: 'c28', name: '高維概念具象手套', desc: '每次點擊 +12 Sx', cost: 4e23, val: 1.2e22 },
        { id: 'c29', name: '多元宇宙打擊艦隊', desc: '每次點擊 +100 Sx', cost: 3.5e24, val: 1e23 },
        { id: 'c30', name: '高靈意識集體點擊', desc: '每次點擊 +800 Sx', cost: 3e25, val: 8e23 },
        { id: 'c31', name: '真理法則塗抹器', desc: '每次點擊 +7 Sp', cost: 2.5e26, val: 7e24 },
        { id: 'c32', name: '始源混沌引力點', desc: '每次點擊 +60 Sp', cost: 2e27, val: 6e25 },
        { id: 'c33', name: '萬物終點按鈕', desc: '每次點擊 +500 Sp', cost: 1.8e28, val: 5e26 },
        { id: 'c34', name: '概念熔煉點擊槍', desc: '每次點擊 +4 Oc', cost: 1.5e29, val: 4e27 },
        { id: 'c35', name: '全知之眼金錢神光', desc: '每次點擊 +35 Oc', cost: 1.2e30, val: 3.5e28 },
        { id: 'c36', name: '創造主思維閃電', desc: '每次點擊 +300 Oc', cost: 1e31, val: 3e29 },
        { id: 'c37', name: '終極富豪概念權杖', desc: '每次點擊 +2.5 No', cost: 9e31, val: 2.5e30 },
        { id: 'c38', name: '超脫幾何擊發器', desc: '每次點擊 +22 No', cost: 8e32, val: 2.2e31 },
        { id: 'c39', name: '存在本質金錢點陣', desc: '每次點擊 +200 No', cost: 7e33, val: 2e32 },
        { id: 'c40', name: '無上創世金錢主按紐', desc: '每次點擊 +1.8 Dc', cost: 6e34, val: 1.8e33 }
    ];

    // 40 個被動資產單位
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
        { id: 'a10', name: '小行星採礦基地', desc: '每秒產出 +4,000,000', cost: 300000000, mps: 4000000 },
        { id: 'a11', name: '恆星戴森球基地', desc: '每秒產出 +25 M', cost: 2000000000, mps: 25000000 },
        { id: 'a12', name: '黑洞能量抽提站', desc: '每秒產出 +150 M', cost: 15000000000, mps: 150000000 },
        { id: 'a13', name: '星系貿易樞紐網', desc: '每秒產出 +1 B', cost: 100000000000, mps: 1000000000 },
        { id: 'a14', name: '暗物質提煉工廠', desc: '每秒產出 +7 B', cost: 800000000000, mps: 7000000000 },
        { id: 'a15', name: '量子並行複製矩陣', desc: '每秒產出 +50 B', cost: 6000000000000, mps: 50000000000 },
        { id: 'a16', name: '時空裂縫資源站', desc: '每秒產出 +350 B', cost: 50000000000000, mps: 350000000000 },
        { id: 'a17', name: '超維度造幣文明', desc: '每秒產出 +2.5 T', cost: 400000000000000, mps: 2500000000000 },
        { id: 'a18', name: '現實法則重構網', desc: '每秒產出 +20 T', cost: 3500000000000000, mps: 20000000000000 },
        { id: 'a19', name: '創世大爆炸發生器', desc: '每秒產出 +150 T', cost: 30000000000000000, mps: 150000000000000 },
        { id: 'a20', name: '無限多元宇宙心臟', desc: '每秒產出 +1,200 T', cost: 250000000000000000, mps: 1200000000000000 },
        { id: 'a21', name: '概率坍縮鑄造廠', desc: '每秒產出 +10 Q', cost: 2e18, mps: 1e16 },
        { id: 'a22', name: '因果律套利中心', desc: '每秒產出 +80 Q', cost: 1.8e19, mps: 8e16 },
        { id: 'a23', name: '超弦震盪汲取器', desc: '每秒產出 +600 Q', cost: 1.5e20, mps: 6e17 },
        { id: 'a24', name: '反熵能源反應爐', desc: '每秒產出 +4.5 Qi', cost: 1.2e21, mps: 4.5e18 },
        { id: 'a25', name: '時間線融合作業區', desc: '每秒產出 +35 Qi', cost: 1e22, mps: 3.5e19 },
        { id: 'a26', name: '神聖數學規律陣', desc: '每秒產出 +280 Qi', cost: 8.5e22, mps: 2.8e20 },
        { id: 'a27', name: '虛無實體貨幣庫', desc: '每秒產出 +2.2 Sx', cost: 7e23, mps: 2.2e21 },
        { id: 'a28', name: '高維概念具象器', desc: '每秒產出 +18 Sx', cost: 6e24, mps: 1.8e22 },
        { id: 'a29', name: '平行宇宙掠奪艦隊', desc: '每秒產出 +150 Sx', cost: 5e25, mps: 1.5e23 },
        { id: 'a30', name: '高靈意識共享網', desc: '每秒產出 +1.2 Sp', cost: 4e26, mps: 1.2e24 },
        { id: 'a31', name: '真理法則寫入儀', desc: '每秒產出 +10 Sp', cost: 3.5e27, mps: 1e25 },
        { id: 'a32', name: '始源混沌引力井', desc: '每秒產出 +85 Sp', cost: 3e28, mps: 8.5e25 },
        { id: 'a33', name: '萬物終點回收站', desc: '每秒產出 +700 Sp', cost: 2.5e29, mps: 7e26 },
        { id: 'a34', name: '概念概念化熔爐', desc: '每秒產出 +6 Oc', cost: 2e30, mps: 6e27 },
        { id: 'a35', name: '全知之眼金錢光束', desc: '每秒產出 +50 Oc', cost: 1.8e31, mps: 5e28 },
        { id: 'a36', name: '創造主思維印記', desc: '每秒產出 +450 Oc', cost: 1.5e32, mps: 4.5e29 },
        { id: 'a37', name: '終極富豪概念圖騰', desc: '每秒產出 +4 No', cost: 1.2e33, mps: 4e30 },
        { id: 'a38', name: '超脫幾何構造體', desc: '每秒產出 +35 No', cost: 1e34, mps: 3.5e31 },
        { id: 'a39', name: '存在本質金錢點陣', desc: '每秒產出 +300 No', cost: 9e34, mps: 3e32 },
        { id: 'a40', name: '絕對無上金錢總源', desc: '每秒產出 +2.5 Dc', cost: 8e35, mps: 2.5e33 }
    ];

    // 20 個高強度科技
    const techList = [
        { id: 't1', name: '暴擊訓練', desc: '暴擊率 +5%', cost: 200, effect: () => gameState.critChance += 0.05 },
        { id: 't2', name: '槓桿投資', desc: '暴擊傷害倍率 +1.5x', cost: 1000, effect: () => gameState.critMulti += 1.5 },
        { id: 't3', name: '大數據行銷', desc: '被動總收益永久 2 倍', cost: 5000, effect: () => globalTechMpsMult *= 2 },
        { id: 't4', name: '解鎖自動點擊器', desc: '開啟自動點擊工具列', cost: 10000, effect: () => gameState.hasAutoClicker = true },
        { id: 't5', name: '極速脈衝點擊', desc: '自動點擊每秒 2 次', cost: 50000, effect: () => gameState.autoSpeed = 500 },
        { id: 't6', name: '點擊增幅陣列', desc: '手動點擊收益 3 倍', cost: 200000, effect: () => globalTechClickMult *= 3 },
        { id: 't7', name: '極限暴擊心法', desc: '暴擊率再 +10%', cost: 1000000, effect: () => gameState.critChance += 0.10 },
        { id: 't8', name: '資本聚變效應', desc: '被動總收益再翻 3 倍', cost: 8000000, effect: () => globalTechMpsMult *= 3 },
        { id: 't9', name: '光速自動點擊', desc: '自動點擊提升至每秒 5 次', cost: 50000000, effect: () => gameState.autoSpeed = 200 },
        { id: 't10', name: '核子暴擊加成', desc: '暴擊傷害倍率 +5.0x', cost: 300000000, effect: () => gameState.critMulti += 5.0 },
        { id: 't11', name: '超導體複利鏈', desc: '被動總收益大幅提升 5 倍', cost: 2e9, effect: () => globalTechMpsMult *= 5 },
        { id: 't12', name: '神經連鎖點擊', desc: '手動點擊收益提升 5 倍', cost: 1.5e10, effect: () => globalTechClickMult *= 5 },
        { id: 't13', name: '極度幸運天賦', desc: '暴擊率額外 +20%', cost: 1e11, effect: () => gameState.critChance += 0.20 },
        { id: 't14', name: '瘋狂自動驅動', desc: '自動點擊達到每秒 10 次', cost: 8e11, effect: () => gameState.autoSpeed = 100 },
        { id: 't15', name: '量子金融風暴', desc: '被動收益提升 10 倍', cost: 5e12, effect: () => globalTechMpsMult *= 10 },
        { id: 't16', name: '次元點擊狂潮', desc: '手動點擊收益提升 10 倍', cost: 4e13, effect: () => globalTechClickMult *= 10 },
        { id: 't17', name: '終極暴擊毀滅', desc: '暴擊傷害倍率 +10x', cost: 3e14, effect: () => gameState.critMulti += 10.0 },
        { id: 't18', name: '反物質收益引擎', desc: '被動收益超級翻倍 20 倍', cost: 2e15, effect: () => globalTechMpsMult *= 20 },
        { id: 't19', name: '創世點擊神權', desc: '手動點擊收益超級翻倍 20 倍', cost: 1.5e16, effect: () => globalTechClickMult *= 20 },
        { id: 't20', name: '絕對真理矩陣', desc: '全域收益再乘 50 倍', cost: 1e17, effect: () => { globalTechMpsMult *= 50; globalTechClickMult *= 50; } }
    ];

    // -------------------------------------------------------------
    // 🔥 30 個 重生金幣 / 聲望天賦 (Prestige Techs)
    // -------------------------------------------------------------
    const presTechList = [
        { id: 'pt1', name: '星光起步', desc: '轉生後初始基礎金額 +$10,000', cost: 1, effect: () => window.startBonus = (window.startBonus||0) + 10000 },
        { id: 'pt2', name: '水晶共振', desc: '每顆重生金幣加成提升至 +20%', cost: 2, effect: () => window.crystalBonusRate = 0.20 },
        { id: 'pt3', name: '連點記憶', desc: '轉生時保留前 5 項手動點擊等級', cost: 3, effect: () => window.keepUpgrades5 = true },
        { id: 'pt4', name: '被動繼承', desc: '轉生時保留前 3 項被動資產', cost: 5, effect: () => window.keepAssets3 = true },
        { id: 'pt5', name: '暴擊靈感', desc: '暴擊率永久 +8%', cost: 8, effect: () => gameState.critChance += 0.08 },
        { id: 'pt6', name: '倍數槓桿', desc: '暴擊倍率 +3.0x', cost: 12, effect: () => gameState.critMulti += 3.0 },
        { id: 'pt7', name: '星辰複利', desc: '被動收益永久 3 倍', cost: 20, effect: () => globalTechMpsMult *= 3 },
        { id: 'pt8', name: '脈衝強化', desc: '點擊收益永久 3 倍', cost: 30, effect: () => globalTechClickMult *= 3 },
        { id: 'pt9', name: '挖礦爆發', desc: '數字金錢挖礦獎勵 5 倍', cost: 50, effect: () => window.minerRewardMult = 5 },
        { id: 'pt10', name: '股市內線', desc: '股市保底最低不低於 $20', cost: 75, effect: () => window.stockFloor = 20 },
        { id: 'pt11', name: '資本狂潮', desc: '被動收益永久 5 倍', cost: 100, effect: () => globalTechMpsMult *= 5 },
        { id: 'pt12', name: '神之右手', desc: '點擊收益永久 5 倍', cost: 150, effect: () => globalTechClickMult *= 5 },
        { id: 'pt13', name: '幸運天神', desc: '暴擊率額外 +15%', cost: 220, effect: () => gameState.critChance += 0.15 },
        { id: 'pt14', name: '核能挖礦機', desc: '算力提升 3 倍', cost: 300, effect: () => gameState.minePower *= 3 },
        { id: 'pt15', name: '聲望增幅矩陣', desc: '每顆重生金幣加成提升至 +35%', cost: 500, effect: () => window.crystalBonusRate = 0.35 },
        { id: 'pt16', name: '富豪降臨', desc: '轉生後初始金額給予 $1,000,000', cost: 800, effect: () => window.startBonus = (window.startBonus||0) + 1000000 },
        { id: 'pt17', name: '星系點擊脈衝', desc: '點擊收益直接提升 10 倍', cost: 1200, effect: () => globalTechClickMult *= 10 },
        { id: 'pt18', name: '恆星收益引擎', desc: '被動收益直接提升 10 倍', cost: 2000, effect: () => globalTechMpsMult *= 10 },
        { id: 'pt19', name: '聲望奇點', desc: '全域收益再乘以 20 倍', cost: 3500, effect: () => { globalTechMpsMult *= 20; globalTechClickMult *= 20; } },
        { id: 'pt20', name: '重生無上主宰', desc: '全域收益與暴擊倍率再暴漲 50 倍', cost: 5000, effect: () => { globalTechMpsMult *= 50; globalTechClickMult *= 50; gameState.critMulti *= 2; } },
        // 新增 10 個神階天賦（價格前一個的 2 倍）
        { id: 'pt21', name: '⚡ 次方起爆·雙重共振', desc: '全域收益提升 100 倍，且暴擊倍率平方提升！', cost: 10000, effect: () => { globalTechMpsMult *= 100; globalTechClickMult *= 100; gameState.critMulti = Math.pow(gameState.critMulti, 1.2); } },
        { id: 'pt22', name: '🌌 被動轉換·點擊超載', desc: '手動點擊獲得相當於【100 秒總被動】的瞬間額外加成！', cost: 20000, effect: () => { window.clickMpsRatio = (window.clickMpsRatio || 0) + 100; } },
        { id: 'pt23', name: '💎 聲望水晶指數躍升', desc: '每顆重生金幣加成效果強制指數提升！(+100% 每顆)', cost: 40000, effect: () => { window.crystalBonusRate = 1.0; } },
        { id: 'pt24', name: '⛏️ 恆星挖礦脈衝陣列', desc: '挖礦算力瞬間乘以 1,000 倍！', cost: 80000, effect: () => { gameState.minePower *= 1000; } },
        { id: 'pt25', name: '🤖 自動萬次極速點擊', desc: '自動點擊頻率極限加倍（全域點擊收益 x500）', cost: 160000, effect: () => { globalTechClickMult *= 500; } },
        { id: 'pt26', name: '🌀 聲望幾何級數膨脹', desc: '全域收益直接進行 1,000 倍超級爆發！', cost: 320000, effect: () => { globalTechMpsMult *= 1000; globalTechClickMult *= 1000; } },
        { id: 'pt27', name: '📈 金融指數崩解壟斷', desc: '股市收益與保底股價直接乘以 10,000 倍！', cost: 640000, effect: () => { window.stockFloor = (window.stockFloor || 1) * 10000; } },
        { id: 'pt28', name: '🔥 次方奇點·倍率次方化', desc: '全域被動與手動倍率直接進行 $x^{1.5}$ 次方運算加成！', cost: 1280000, effect: () => { globalTechMpsMult = Math.pow(globalTechMpsMult, 1.5); globalTechClickMult = Math.pow(globalTechClickMult, 1.5); } },
        { id: 'pt29', name: '👑 創世聲望神王印記', desc: '全域收益爆炸性翻倍 1,000,000 倍！', cost: 2560000, effect: () => { globalTechMpsMult *= 1e6; globalTechClickMult *= 1e6; } },
        { id: 'pt30', name: '🌟 無上聲望·二次方神威', desc: '將當前全域點擊與被動倍率直接【平方 ($x^2$)】！', cost: 5120000, effect: () => { globalTechMpsMult = Math.pow(globalTechMpsMult, 2); globalTechClickMult = Math.pow(globalTechClickMult, 2); } }
    ];

    // -------------------------------------------------------------
    // ✨ 30 個 超越金幣 / 超越神殿天賦 (Transcend Techs)
    // -------------------------------------------------------------
    const transTechList = [
        { id: 'tt1', name: '暗物質點金術', desc: '所有收益永久 10 倍', cost: 1, effect: () => window.transPower = (window.transPower||1) * 10 },
        { id: 'tt2', name: '無盡時空鏈', desc: '重生時保留 20% 的所有資產與升級', cost: 2, effect: () => window.keepAssetsRate = 0.2 },
        { id: 'tt3', name: '宇宙算力爆發', desc: '挖礦機效率提升 5 倍', cost: 3, effect: () => window.minerMult = 5 },
        { id: 'tt4', name: '次元暴擊破限', desc: '暴擊倍率直接 +10.0x', cost: 5, effect: () => gameState.critMulti += 10.0 },
        { id: 'tt5', name: '高維度被動流', desc: '被動總收益 20 倍', cost: 8, effect: () => globalTechMpsMult *= 20 },
        { id: 'tt6', name: '超越點擊神力', desc: '點擊收益 20 倍', cost: 12, effect: () => globalTechClickMult *= 20 },
        { id: 'tt7', name: '百分之百暴擊', desc: '暴擊率 +25%', cost: 18, effect: () => gameState.critChance += 0.25 },
        { id: 'tt8', name: '暗物質重力井', desc: '每顆超越金幣效益提升 3 倍', cost: 25, effect: () => window.transPower = (window.transPower||1) * 3 },
        { id: 'tt9', name: '時空逆轉法則', desc: '重生保留率提升至 50%', cost: 40, effect: () => window.keepAssetsRate = 0.5 },
        { id: 'tt10', name: '無限礦業網', desc: '挖礦自動獲得 100 倍金錢獎勵', cost: 60, effect: () => window.minerRewardMult = (window.minerRewardMult||1) * 100 },
        { id: 'tt11', name: '平行宇宙掠奪', desc: '全域收益翻 50 倍', cost: 100, effect: () => { globalTechMpsMult *= 50; globalTechClickMult *= 50; } },
        { id: 'tt12', name: '超越核聚變', desc: '被動收益提升 100 倍', cost: 150, effect: () => globalTechMpsMult *= 100 },
        { id: 'tt13', name: '神聖點擊閃電', desc: '手動點擊提升 100 倍', cost: 220, effect: () => globalTechClickMult *= 100 },
        { id: 'tt14', name: '毀滅性暴擊', desc: '暴擊倍率再乘以 3 倍', cost: 350, effect: () => gameState.critMulti *= 3 },
        { id: 'tt15', name: '完整完美傳承', desc: '重生時保留 100% 的所有資產等級', cost: 500, effect: () => window.keepAssetsRate = 1.0 },
        { id: 'tt16', name: '宇宙算力奇點', desc: '挖礦算力永久加 1,000 GH/s', cost: 800, effect: () => gameState.minePower += 1000 },
        { id: 'tt17', name: '創世暗物質引擎', desc: '全域收益爆炸性提升 500 倍', cost: 1200, effect: () => { globalTechMpsMult *= 500; globalTechClickMult *= 500; } },
        { id: 'tt18', name: '高維絕對法則', desc: '暴擊率強制達到 100%', cost: 2000, effect: () => gameState.critChance = 1.0 },
        { id: 'tt19', name: '超脫幾何真理', desc: '全域收益提升 2,000 倍', cost: 3500, effect: () => { globalTechMpsMult *= 2000; globalTechClickMult *= 2000; } },
        { id: 'tt20', name: '無上神權·宇宙金錢概念', desc: '全域收益超級暴漲 10,000 倍！', cost: 5000, effect: () => { globalTechMpsMult *= 10000; globalTechClickMult *= 10000; } },
        // 新增 10 個神級次方天賦（價格前一個的 2 倍）
        { id: 'tt21', name: '🌌 超越次方·平方膨脹', desc: '將當前全域被動與點擊倍率進行【平方 ($x^2$)】！', cost: 10000, effect: () => { globalTechMpsMult = Math.pow(globalTechMpsMult, 2); globalTechClickMult = Math.pow(globalTechClickMult, 2); } },
        { id: 'tt22', name: '⏳ 時間流速超撕裂陣列', desc: '遊戲時間流速增快 10 倍（每秒計算 10 次被動與挖礦）！', cost: 20000, effect: () => { window.timeSpeed = (window.timeSpeed || 1) * 10; } },
        { id: 'tt23', name: '💥 次方暴擊·極限撕裂', desc: '暴擊傷害倍率直接進行【平方 ($x^2$)】級數增加！', cost: 40000, effect: () => { gameState.critMulti = Math.pow(gameState.critMulti, 2); } },
        { id: 'tt24', name: '⚛️ 暗物質三次方躍遷', desc: '將當前全域總倍率進行【立方 ($x^3$)】次方化膨脹！', cost: 80000, effect: () => { globalTechMpsMult = Math.pow(globalTechMpsMult, 3); globalTechClickMult = Math.pow(globalTechClickMult, 3); } },
        { id: 'tt25', name: '🌀 維度折疊·無限資源流', desc: '每秒自動觸發 1,000 次超級暴擊手動點擊！', cost: 160000, effect: () => { window.autoClickPower = (window.autoClickPower || 0) + 1000; } },
        { id: 'tt26', name: '🔮 超越四次方宇宙崩解', desc: '全域倍率強勢進行【四次方 ($x^4$)】算力膨脹！', cost: 320000, effect: () => { globalTechMpsMult = Math.pow(globalTechMpsMult, 4); globalTechClickMult = Math.pow(globalTechClickMult, 4); } },
        { id: 'tt27', name: '👑 絕對概念·萬物皆為財富', desc: '所有暗物質超越金幣加成提升 100,000 倍！', cost: 640000, effect: () => { window.transPower = (window.transPower || 1) * 100000; } },
        { id: 'tt28', name: '🌌 五次方神聖概念具象', desc: '全域收益進行【五次方 ($x^5$)】宇宙級爆發！', cost: 1280000, effect: () => { globalTechMpsMult = Math.pow(globalTechMpsMult, 5); globalTechClickMult = Math.pow(globalTechClickMult, 5); } },
        { id: 'tt29', name: '✨ 終極十次方數值毀滅', desc: '全域點擊與被動倍率進行【十次方 ($x^{10}$)** 終極運算！', cost: 2560000, effect: () => { globalTechMpsMult = Math.pow(globalTechMpsMult, 10); globalTechClickMult = Math.pow(globalTechClickMult, 10); } },
        { id: 'tt30', name: '♾️ 無上終極神權·無限金錢帝國', desc: '收益直接賦予【1e100 無限指數】極致神威加成！', cost: 5120000, effect: () => { globalTechMpsMult *= 1e100; globalTechClickMult *= 1e100; } }
    ];

    // -------------------------------------------------------------
    // 數值計算邏輯
    // -------------------------------------------------------------
    function getPrestigeMultiplier() {
        let rate = window.crystalBonusRate || 0.10;
        return 1 + (gameState.prestigeCrystals * rate);
    }

    function getTransMultiplier() {
        return 1 + (gameState.darkMatter * 5) * (window.transPower || 1);
    }

    function calculateClickValue() {
        let base = 1;
        clickUpgradesList.forEach(item => {
            let count = gameState.upgrades[item.id] || 0;
            base += item.val * count;
        });
        let mpsRatioAdd = (window.clickMpsRatio || 0) * calculateMPS();
        return (base + mpsRatioAdd) * globalTechClickMult * getPrestigeMultiplier() * getTransMultiplier();
    }

    function calculateMPS() {
        let baseMps = 0;
        assetsList.forEach(item => {
            let count = gameState.assets[item.id] || 0;
            baseMps += item.mps * count;
        });
        return baseMps * globalTechMpsMult * getPrestigeMultiplier() * getTransMultiplier();
    }

    // -------------------------------------------------------------
    // 點擊與購買觸發按紐
    // -------------------------------------------------------------
    function clickMoney(e) {
        if (isPaused) return;
        gameState.totalClickCount++;
        let isCrit = Math.random() < gameState.critChance;
        let gained = calculateClickValue();
        if (isCrit) gained *= gameState.critMulti;

        gameState.money += gained;
        gameState.totalMoneyEarned += gained;

        if (e && e.clientX) {
            showFloatingText(e.clientX, e.clientY, `+$${formatNumber(gained)}${isCrit ? ' 暴擊!' : ''}`, isCrit);
        }
        updateDisplay();
    }

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

    function buyPresTech(id) {
        let item = presTechList.find(x => x.id === id);
        if (!gameState.presTechs[id] && gameState.prestigeCrystals >= item.cost) {
            gameState.prestigeCrystals -= item.cost;
            gameState.presTechs[id] = true;
            item.effect();
            updateDisplay();
        }
    }

    function buyTransTech(id) {
        let item = transTechList.find(x => x.id === id);
        if (!gameState.transTechs[id] && gameState.darkMatter >= item.cost) {
            gameState.darkMatter -= item.cost;
            gameState.transTechs[id] = true;
            item.effect();
            updateDisplay();
        }
    }

    // 重生與超越
    function prestige() {
        let req = 100000;
        let crystals = Math.floor(Math.cbrt(gameState.totalMoneyEarned / req));
        if (crystals > gameState.prestigeCrystals) {
            let pending = crystals - gameState.prestigeCrystals;
            if (confirm(`確定轉生並獲得 ${pending} 顆【重生金幣】嗎？`)) {
                gameState.prestigeCrystals += pending;
                gameState.money = window.startBonus || 0;
                
                if (!window.keepAssetsRate) {
                    if (!window.keepUpgrades5) {
                        let newUpgrades = {};
                        for(let i=1;i<=5;i++) newUpgrades['c'+i] = gameState.upgrades['c'+i]||0;
                        gameState.upgrades = newUpgrades;
                    }
                    if (!window.keepAssets3) {
                        let newAssets = {};
                        for(let i=1;i<=3;i++) newAssets['a'+i] = gameState.assets['a'+i]||0;
                        gameState.assets = newAssets;
                    }
                } else {
                    Object.keys(gameState.upgrades).forEach(k => gameState.upgrades[k] = Math.floor(gameState.upgrades[k] * window.keepAssetsRate));
                    Object.keys(gameState.assets).forEach(k => gameState.assets[k] = Math.floor(gameState.assets[k] * window.keepAssetsRate));
                }
                updateDisplay();
            }
        } else {
            alert("累積金額不足以獲得額外重生金幣！");
        }
    }

    function transcend() {
        let req = 1e15;
        if (gameState.totalMoneyEarned < req) {
            alert("需要歷史總金額達到 $1,000 T (1 Quad) 才能觸發超越！");
            return;
        }
        let earnedMatter = Math.floor(Math.cbrt(gameState.totalMoneyEarned / req));
        if (confirm(`🌌 確定要【超越升華】嗎？\n這將重置金錢與基礎資產，獲得 ${earnedMatter} 顆【超越金幣】！`)) {
            gameState.darkMatter += earnedMatter;
            gameState.transcendCount++;
            gameState.money = 0;
            gameState.totalMoneyEarned = 0;
            gameState.prestigeCrystals = 0;
            gameState.upgrades = {};
            gameState.assets = {};
            gameState.techs = {};
            gameState.presTechs = {};
            gameState.storyProgress = 0;
            globalTechMpsMult = 1;
            globalTechClickMult = 1;
            
            updateDisplay();
            alert(`✨ 超越成功！獲得 ${earnedMatter} 顆超越金幣！`);
        }
    }

    // -------------------------------------------------------------
    // UI 渲染與刷寫邏輯
    // -------------------------------------------------------------
    function initUI() {
        renderList(clickUpgradesList, 'upgrades', 'buyClickUpgrade');
        renderList(assetsList, 'assets', 'buyAsset');
        renderTechList(techList, 'tech', 'buyTech');
        renderTechList(presTechList, 'presTree', 'buyPresTech', '重生金幣');
        renderTechList(transTechList, 'transTree', 'buyTransTech', '超越金幣');
        renderStock();
    }

    function renderList(list, targetId, buyFuncName) {
        const container = document.getElementById(targetId);
        container.innerHTML = '';
        list.forEach(item => {
            const div = document.createElement('div');
            div.className = 'item-card';
            div.innerHTML = `
                <div>
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

    function renderTechList(list, targetId, buyFuncName, currencyName = '$') {
        const container = document.getElementById(targetId);
        container.innerHTML = '';
        list.forEach(item => {
            const div = document.createElement('div');
            div.className = 'item-card';
            div.innerHTML = `
                <div>
                    <div class="item-title">${item.name}</div>
                    <div class="item-desc">${item.desc}</div>
                </div>
                <button class="buy-btn" id="btn-${item.id}" onclick="${buyFuncName}('${item.id}')">
                    <span id="cost-${item.id}">${currencyName === '$' ? '$' + formatNumber(item.cost) : formatNumber(item.cost) + ' ' + currencyName}</span>
                </button>
            `;
            container.appendChild(div);
        });
    }

    function renderStock() {
        const container = document.getElementById('stockList');
        container.innerHTML = '';
        Object.keys(gameState.stocks).forEach(key => {
            let s = gameState.stocks[key];
            const div = document.createElement('div');
            div.className = 'item-card';
            div.innerHTML = `
                <div>
                    <div class="item-title">${s.name} (${key})</div>
                    <div class="item-desc">當前股價: $${formatNumber(s.price)} | 持有: ${s.shares} 股</div>
                </div>
                <div style="display:flex; gap:4px;">
                    <button class="buy-btn" onclick="buyStock('${key}')">買入</button>
                    <button class="buy-btn" style="background:#dc2626;" onclick="sellStock('${key}')">賣出</button>
                </div>
            `;
            container.appendChild(div);
        });
    }

    function updateDisplay() {
        document.getElementById('money').innerText = '$' + formatNumber(gameState.money);
        document.getElementById('mps').innerText = `每秒被動: $${formatNumber(calculateMPS())}`;
        document.getElementById('mpc').innerText = `每次點擊: $${formatNumber(calculateClickValue())}`;
        document.getElementById('critStat').innerText = `暴擊率: ${(gameState.critChance * 100).toFixed(0)}% (${gameState.critMulti.toFixed(1)}x)`;
        
        let pCrystals = Math.floor(Math.cbrt(gameState.totalMoneyEarned / 100000));
        document.getElementById('prestigeStat').innerText = `重生金幣: ${formatNumber(gameState.prestigeCrystals)} (加成 +${((getPrestigeMultiplier()-1)*100).toFixed(0)}%)`;
        document.getElementById('nextPrestigeStat').innerText = `重置可得: +${formatNumber(Math.max(0, pCrystals - gameState.prestigeCrystals))} 重生金幣`;
        
        document.getElementById('transStat').innerText = `超越金幣: ${formatNumber(gameState.darkMatter)} (加成 +${((getTransMultiplier()-1)*100).toFixed(0)}%)`;

        // 升級按鈕與數量更新
        clickUpgradesList.forEach(item => {
            let count = gameState.upgrades[item.id] || 0;
            let cost = Math.floor(item.cost * Math.pow(1.15, count));
            let btn = document.getElementById(`btn-${item.id}`);
            if (btn) {
                document.getElementById(`cost-${item.id}`).innerText = formatNumber(cost);
                document.getElementById(`count-${item.id}`).innerText = count > 0 ? `[Lv.${count}]` : '';
                btn.disabled = gameState.money < cost;
            }
        });

        assetsList.forEach(item => {
            let count = gameState.assets[item.id] || 0;
            let cost = Math.floor(item.cost * Math.pow(1.15, count));
            let btn = document.getElementById(`btn-${item.id}`);
            if (btn) {
                document.getElementById(`cost-${item.id}`).innerText = formatNumber(cost);
                document.getElementById(`count-${item.id}`).innerText = count > 0 ? `[x${count}]` : '';
                btn.disabled = gameState.money < cost;
            }
        });

        techList.forEach(item => {
            let btn = document.getElementById(`btn-${item.id}`);
            if (btn) {
                if (gameState.techs[item.id]) {
                    btn.innerText = '已研發';
                    btn.disabled = true;
                } else {
                    btn.disabled = gameState.money < item.cost;
                }
            }
        });

        presTechList.forEach(item => {
            let btn = document.getElementById(`btn-${item.id}`);
            if (btn) {
                if (gameState.presTechs[item.id]) {
                    btn.innerText = '已解鎖';
                    btn.disabled = true;
                } else {
                    btn.disabled = gameState.prestigeCrystals < item.cost;
                }
            }
        });

        transTechList.forEach(item => {
            let btn = document.getElementById(`btn-${item.id}`);
            if (btn) {
                if (gameState.transTechs[item.id]) {
                    btn.innerText = '已解鎖';
                    btn.disabled = true;
                } else {
                    btn.disabled = gameState.darkMatter < item.cost;
                }
            }
        });

        checkStoryTrigger();
    }

    // 股市買賣
    function buyStock(key) {
        let s = gameState.stocks[key];
        if (gameState.money >= s.price) {
            gameState.money -= s.price;
            s.shares++;
            renderStock();
            updateDisplay();
        }
    }
    function sellStock(key) {
        let s = gameState.stocks[key];
        if (s.shares > 0) {
            gameState.money += s.price;
            s.shares--;
            renderStock();
            updateDisplay();
        }
    }

    // 挖礦與自動點擊
    function updateMining() {
        gameState.mineProgress += (gameState.minePower * (window.minerMult || 1));
        if (gameState.mineProgress >= 100) {
            gameState.mineProgress = 0;
            let reward = (calculateMPS() * 10 + 500) * (window.minerRewardMult || 1);
            gameState.money += reward;
            gameState.totalMoneyEarned += reward;
            showFloatingText(window.innerWidth / 2, window.innerHeight / 2, `⛏️ 哈希爆破成功: +$${formatNumber(reward)}!`, true);
        }
        let bar = document.getElementById('mineBar');
        if (bar) bar.style.width = `${gameState.mineProgress}%`;
    }

    function upgradeMiner() {
        let cost = gameState.minePower * 10000;
        if (gameState.money >= cost) {
            gameState.money -= cost;
            gameState.minePower++;
            document.getElementById('minePower').innerText = gameState.minePower;
            document.getElementById('mineUpgradeCost').innerText = formatNumber(gameState.minePower * 10000);
            updateDisplay();
        }
    }

    let autoClickTimer = null;
    function toggleAutoClicker() {
        if (!gameState.hasAutoClicker) {
            alert("請先在【科技研發】頁面解鎖【解鎖自動點擊器】！");
            return;
        }
        gameState.autoClickerActive = !gameState.autoClickerActive;
        document.getElementById('autoClickerBtn').innerText = `🤖 自動點擊: ${gameState.autoClickerActive ? '開' : '關'}`;
        
        if (gameState.autoClickerActive) {
            if (autoClickTimer) clearInterval(autoClickTimer);
            autoClickTimer = setInterval(() => { if (!isPaused) clickMoney(); }, gameState.autoSpeed);
        } else {
            if (autoClickTimer) clearInterval(autoClickTimer);
        }
    }

    // 劇情彈出
    function checkStoryTrigger() {
        if (gameState.skipStory || isPaused) return;
        let nextChapter = storyChapters.find(c => c.id === gameState.storyProgress);
        if (nextChapter && gameState.totalMoneyEarned >= nextChapter.req) {
            document.getElementById('storyTitle').innerText = `📖 ${nextChapter.title}`;
            document.getElementById('storyText').innerText = nextChapter.text;
            document.getElementById('storyModal').style.display = 'flex';
        }
    }

    function closeStory(skipAll) {
        document.getElementById('storyModal').style.display = 'none';
        if (skipAll) gameState.skipStory = true;
        gameState.storyProgress++;
    }

    function openStoryLog() {
        let unlocked = storyChapters.filter(c => c.id < gameState.storyProgress);
        let msg = unlocked.map(c => `【${c.title}】\n${c.text}\n`).join('\n-------------------\n');
        alert(msg || "尚未解鎖任何劇情章節！");
    }

    function switchTab(tabId, evt) {
        document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
        document.querySelectorAll('.tab-btn').forEach(el => el.classList.remove('active'));
        document.getElementById(tabId).classList.add('active');
        if (evt && evt.target) evt.target.classList.add('active');
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

    function saveGame() { localStorage.setItem('clicker_v3_3_save', JSON.stringify({ gameState, numberFormatMode })); }
    function loadGame() {
        let saved = localStorage.getItem('clicker_v3_3_save');
        if (saved) {
            let data = JSON.parse(saved);
            if (data.gameState) gameState = { ...gameState, ...data.gameState };
            if (data.numberFormatMode) numberFormatMode = data.numberFormatMode;
        }
    }
    function exportSave() { prompt("複製存檔代碼：", btoa(JSON.stringify({ gameState, numberFormatMode }))); }
    function importSave() {
        let str = prompt("貼上存檔代碼：");
        if (str) {
            let data = JSON.parse(atob(str));
            if (data.gameState) gameState = data.gameState;
            if (data.numberFormatMode) numberFormatMode = data.numberFormatMode;
            updateDisplay();
        }
    }

    // -------------------------------------------------------------
    // 主循環 (Tick Loop)
    // -------------------------------------------------------------
    initUI();
    loadGame();
    updateDisplay();

    setInterval(() => {
        if (isPaused) return;

        let multiplier = (window.timeSpeed || 1);
        let mps = calculateMPS();
        let gained = (mps / 10) * multiplier;
        gameState.money += gained;
        gameState.totalMoneyEarned += gained;

        // 次方級自動維度點擊流
        if (window.autoClickPower) {
            let autoGained = calculateClickValue() * window.autoClickPower * multiplier;
            gameState.money += autoGained;
            gameState.totalMoneyEarned += autoGained;
        }
        
        updateMining();
        
        // 股市隨機波動
        if (Math.random() < 0.05) {
            let floor = window.stockFloor || 1;
            Object.keys(gameState.stocks).forEach(k => {
                let change = (Math.random() - 0.48) * 0.1;
                gameState.stocks[k].price = Math.max(floor, gameState.stocks[k].price * (1 + change));
            });
            renderStock();
        }

        updateDisplay();
    }, 100);

    setInterval(saveGame, 10000);
</script>
</body>
</html>
