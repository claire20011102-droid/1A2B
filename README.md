# 1A2B
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>1A2B 猜數字學習與策略探索器</title>
    <!-- 引入 Canvas Confetti 彩帶特效庫 -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
        }

        body {
            background-color: #f0f2f5;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
        }

        /* 頂部三階段導覽列 */
        .mode-nav {
            background-color: #ffffff;
            border-radius: 30px;
            padding: 6px;
            display: flex;
            gap: 6px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.06);
            margin-bottom: 20px;
            flex-wrap: wrap;
            justify-content: center;
        }

        .nav-tab {
            padding: 8px 18px;
            border-radius: 20px;
            font-size: 0.9rem;
            font-weight: 600;
            color: #555;
            cursor: pointer;
            transition: all 0.25s ease;
            user-select: none;
        }

        .nav-tab.active {
            background-color: #007bff;
            color: #ffffff;
            box-shadow: 0 2px 6px rgba(0, 123, 255, 0.3);
        }

        /* 主容器：雙欄 Flex 佈局 */
        .game-container {
            background-color: #ffffff;
            border-radius: 12px;
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
            width: 100%;
            max-width: 880px;
            display: flex;
            overflow: hidden;
            position: relative;
            z-index: 1;
        }

        /* 左側：猜題與控制區 */
        .left-panel {
            flex: 1.15;
            padding: 28px;
            display: flex;
            flex-direction: column;
            border-right: 1px solid #eef0f2;
        }

        /* 右側：紀錄與解析區 */
        .right-panel {
            flex: 0.85;
            padding: 28px;
            background-color: #fafbfc;
            display: flex;
            flex-direction: column;
        }

        .mode-header {
            margin-bottom: 16px;
        }

        h1 {
            font-size: 1.35rem;
            color: #1a1a1a;
            margin-bottom: 4px;
        }

        p.subtitle {
            font-size: 0.88rem;
            color: #666;
            line-height: 1.4;
        }

        /* 規則卡片 */
        .rules-card {
            background-color: #f7f9fa;
            border-left: 4px solid #007bff;
            padding: 10px 12px;
            border-radius: 4px;
            margin-bottom: 16px;
            font-size: 0.83rem;
            color: #444;
            line-height: 1.4;
        }

        .rules-card ul {
            margin-top: 4px;
            padding-left: 18px;
        }

        /* AI 提示與演算法說明卡片 */
        .ai-card {
            background-color: #eef9f1;
            border-left: 4px solid #28a745;
            padding: 14px;
            border-radius: 6px;
            margin-bottom: 16px;
            font-size: 0.85rem;
            color: #2e5a35;
        }

        .ai-card-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
        }

        .ai-btn {
            background-color: #28a745;
            padding: 6px 14px;
            font-size: 0.82rem;
            border-radius: 4px;
            color: #fff;
            border: none;
            cursor: pointer;
            font-weight: 600;
        }

        .ai-btn:hover {
            background-color: #218838;
        }

        .ai-result {
            font-size: 0.88rem;
            color: #155724;
            line-height: 1.45;
        }

        .ai-reason {
            margin-top: 8px;
            font-size: 0.82rem;
            color: #3b7044;
            background-color: rgba(255, 255, 255, 0.7);
            padding: 10px;
            border-radius: 4px;
            border: 1px dashed #b2e0bb;
            line-height: 1.5;
        }

        /* 演算法原理小學堂教學區 */
        .algorithm-explainer {
            background-color: #fff8e6;
            border: 1px solid #ffe58f;
            border-radius: 6px;
            padding: 12px;
            margin-bottom: 16px;
            font-size: 0.82rem;
            color: #734a00;
            line-height: 1.5;
        }

        .algorithm-explainer h4 {
            font-size: 0.88rem;
            color: #8c5000;
            margin-bottom: 6px;
            display: flex;
            align-items: center;
            gap: 4px;
        }

        .algorithm-explainer ol {
            padding-left: 18px;
            margin-top: 4px;
        }

        /* 可能答案數動態儀表卡片 */
        .possibility-tracker {
            background-color: #eef4ff;
            border-left: 4px solid #007bff;
            padding: 10px 12px;
            border-radius: 4px;
            margin-bottom: 16px;
            font-size: 0.85rem;
            color: #1a4175;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .input-group {
            display: flex;
            gap: 10px;
            margin-bottom: 12px;
        }

        input[type="text"] {
            flex: 1;
            padding: 10px 12px;
            font-size: 1.1rem;
            border: 2px solid #dcdfe6;
            border-radius: 6px;
            letter-spacing: 4px;
            text-align: center;
            outline: none;
            transition: border-color 0.2s;
        }

        input[type="text"]:focus {
            border-color: #007bff;
        }

        button.submit-btn {
            padding: 10px 18px;
            font-size: 0.95rem;
            font-weight: 600;
            color: #fff;
            background-color: #007bff;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            transition: background-color 0.2s;
        }

        button.submit-btn:hover {
            background-color: #0056b3;
        }

        .restart-btn {
            width: 100%;
            background-color: #6c757d;
            margin-top: auto;
            padding: 10px;
            color: #fff;
            border: none;
            border-radius: 6px;
            font-weight: 600;
            cursor: pointer;
        }

        .restart-btn:hover {
            background-color: #5a6268;
        }

        .message {
            min-height: 36px;
            font-size: 0.9rem;
            color: #d9534f;
            line-height: 1.4;
            margin-bottom: 12px;
        }

        .history-title {
            font-size: 1.1rem;
            color: #333;
            margin-bottom: 14px;
            padding-bottom: 8px;
            border-bottom: 2px solid #eef0f2;
        }

        .history-list-wrapper {
            flex: 1;
            max-height: 360px;
            overflow-y: auto;
            border: 1px solid #eef0f2;
            border-radius: 6px;
            background-color: #fff;
        }

        .history-list {
            list-style: none;
        }

        .history-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px 14px;
            border-bottom: 1px solid #f0f0f0;
            font-size: 0.92rem;
        }

        .history-item:last-child {
            border-bottom: none;
        }

        .history-item:nth-child(even) {
            background-color: #fafafa;
        }

        .result-badge {
            font-weight: bold;
            color: #28a745;
            background-color: #e8f5e9;
            padding: 2px 8px;
            border-radius: 4px;
        }

        .empty-history {
            text-align: center;
            color: #aaa;
            padding: 20px;
            font-size: 0.85rem;
        }

        /* 樹狀推理歷程樣式 */
        .tree-view {
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .tree-node {
            background-color: #fff;
            border: 1px solid #dcdfe6;
            border-radius: 8px;
            padding: 10px 12px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.02);
            position: relative;
        }

        .tree-node-header {
            display: flex;
            justify-content: space-between;
            font-size: 0.88rem;
            margin-bottom: 6px;
        }

        .progress-bg {
            height: 8px;
            background-color: #eef0f2;
            border-radius: 4px;
            overflow: hidden;
        }

        .progress-bar {
            height: 100%;
            background-color: #007bff;
            transition: width 0.4s ease;
        }

        .tree-details {
            font-size: 0.78rem;
            color: #666;
            margin-top: 6px;
        }

        @media (max-width: 640px) {
            .game-container {
                flex-direction: column;
                max-width: 400px;
            }
            .left-panel {
                border-right: none;
                border-bottom: 1px solid #eef0f2;
            }
            .history-list-wrapper {
                max-height: 200px;
            }
        }
    </style>
</head>
<body>

    <!-- 頂部頁籤切換：三階段學習步驟 -->
    <div class="mode-nav">
        <div class="nav-tab active" id="tabStudent" onclick="switchMode('student')">階段一：自主探索策略</div>
        <div class="nav-tab" id="tabTree" onclick="switchMode('tree')">階段二：電腦思考脈絡圖</div>
        <div class="nav-tab" id="tabAi" onclick="switchMode('ai')">階段三：AI 演算法決策</div>
    </div>

    <div class="game-container">
        <!-- 左側：猜題區 -->
        <div class="left-panel">
            <div class="mode-header">
                <h1 id="panelTitle">🎮 階段一：學生自主探索</h1>
                <p class="subtitle" id="panelSub">嘗試靠自己的直覺與邏輯，找出推算答案的最快策略！</p>
            </div>

            <!-- 固定顯示：遊戲規則說明 -->
            <div class="rules-card">
                <strong>📖 遊戲規則：</strong>
                <ul>
                    <li><strong>A</strong>：數字正確且位置正確。｜ <strong>B</strong>：數字正確但位置錯誤。</li>
                    <li>目標：透過提示推算出正確組合 (<strong>4A0B</strong>)。</li>
                </ul>
            </div>

            <!-- 階段二與階段三專屬：剩餘可能答案即時統計 -->
            <div class="possibility-tracker" id="possibilityTracker" style="display: none;">
                <span>🎯 當前符合所有提示的可能答案數：</span>
                <strong id="possibleCount" style="font-size: 1.1rem; color: #007bff;">5,040 種</strong>
            </div>

            <!-- 階段三專屬：Minimax 演算法核心原理教學卡片 -->
            <div class="algorithm-explainer" id="algoExplainer" style="display: none;">
                <h4>💡 什麼是 Minimax (極小化極大) 演算法？</h4>
                <div>概念：<strong>「假設最壞情況，並選擇讓最壞情況損失最小的策略」</strong></div>
                <ol>
                    <li><strong>模擬預測</strong>：AI 嘗試模擬每個猜測數字（如 `1234`）。</li>
                    <li><strong>評估最壞結果</strong>：計算若拿到最差反饋，會剩幾種可能答案。</li>
                    <li><strong>最佳選擇</strong>：選擇「最壞狀況下，剩餘可能性最少」的組合，確保每一猜都能最快砍掉搜尋空間！</li>
                </ol>
            </div>

            <!-- 階段三專屬：AI 解題助手卡片 -->
            <div class="ai-card" id="aiCard" style="display: none;">
                <div class="ai-card-header">
                    <strong>🤖 Minimax 演算法解析器</strong>
                    <button class="ai-btn" id="aiHintBtn">推薦下一猜</button>
                </div>
                <div class="ai-result" id="aiResultContainer">
                    點擊按鈕查看 AI 如何進行最佳決策分析。
                </div>
            </div>

            <div class="input-group">
                <input type="text" id="guessInput" maxlength="4" placeholder="1234" autofocus autocomplete="off">
                <button class="submit-btn" id="submitBtn">提交猜測</button>
            </div>

            <div class="message" id="message"></div>

            <button class="restart-btn" id="restartBtn">重新開始遊戲</button>
        </div>

        <!-- 右側：歷史紀錄／思考脈絡圖 -->
        <div class="right-panel">
            <div class="history-title" id="rightPanelTitle">歷史紀錄</div>
            
            <!-- 歷史紀錄面板 (階段一、三顯示) -->
            <div class="history-list-wrapper" id="historyWrapper">
                <ul class="history-list" id="historyList">
                    <li class="empty-history" id="emptyMsg">尚無紀錄</li>
                </ul>
            </div>

            <!-- 階段二專屬：思考脈絡樹狀圖面板 -->
            <div class="history-list-wrapper" id="treeWrapper" style="display: none; padding: 12px; background-color: #fafafa;">
                <div class="tree-view" id="treeContainer">
                    <div class="empty-history">遊戲進行中，將在此動態繪製答案範圍（5,040種）逐步縮減的脈絡...</div>
                </div>
            </div>
        </div>
    </div>

    <script>
        let secret = '';
        let attempts = 0;
        let isGameOver = false;
        let userHistory = []; 
        let allPossibleCodes = []; 
        let currentMode = 'student'; 

        const guessInput = document.getElementById('guessInput');
        const submitBtn = document.getElementById('submitBtn');
        const restartBtn = document.getElementById('restartBtn');
        const messageEl = document.getElementById('message');
        const historyList = document.getElementById('historyList');
        
        const tabStudent = document.getElementById('tabStudent');
        const tabTree = document.getElementById('tabTree');
        const tabAi = document.getElementById('tabAi');
        
        const panelTitle = document.getElementById('panelTitle');
        const panelSub = document.getElementById('panelSub');
        const rightPanelTitle = document.getElementById('rightPanelTitle');
        
        const aiCard = document.getElementById('aiCard');
        const algoExplainer = document.getElementById('algoExplainer');
        const possibilityTracker = document.getElementById('possibilityTracker');
        const possibleCount = document.getElementById('possibleCount');
        const aiHintBtn = document.getElementById('aiHintBtn');
        const aiResultContainer = document.getElementById('aiResultContainer');

        const historyWrapper = document.getElementById('historyWrapper');
        const treeWrapper = document.getElementById('treeWrapper');
        const treeContainer = document.getElementById('treeContainer');

        function generateAllPossible() {
            const list = [];
            for (let i = 0; i <= 9999; i++) {
                let str = i.toString().padStart(4, '0');
                if (new Set(str).size === 4) {
                    list.push(str);
                }
            }
            return list;
        }

        // 切換教學階段頁籤
        function switchMode(mode) {
            currentMode = mode;
            tabStudent.classList.remove('active');
            tabTree.classList.remove('active');
            tabAi.classList.remove('active');

            if (mode === 'student') {
                tabStudent.classList.add('active');
                panelTitle.textContent = '🎮 階段一：學生自主探索';
                panelSub.textContent = '靠自己的直覺與邏輯猜測，感受摸索策略的過程。';
                rightPanelTitle.textContent = '歷史紀錄';
                aiCard.style.display = 'none';
                algoExplainer.style.display = 'none';
                possibilityTracker.style.display = 'none';
                historyWrapper.style.display = 'block';
                treeWrapper.style.display = 'none';
            } else if (mode === 'tree') {
                tabTree.classList.add('active');
                panelTitle.textContent = '🌳 階段二：電腦思考脈絡解析';
                panelSub.textContent = '觀察每一次提示如何把 5,040 種可能快速縮減，視覺化理解排除法！';
                rightPanelTitle.textContent = '可能性遞減脈絡圖 (5040漸遞減)';
                aiCard.style.display = 'none';
                algoExplainer.style.display = 'none';
                possibilityTracker.style.display = 'flex';
                historyWrapper.style.display = 'none';
                treeWrapper.style.display = 'block';
                updatePossibilityCount();
                renderTree();
            } else if (mode === 'ai') {
                tabAi.classList.add('active');
                panelTitle.textContent = '🧠 階段三：AI 演算法與最佳決策';
                panelSub.textContent = '運用 Minimax 資訊量最大化演算法，計算最能幫你縮減範圍的最佳下一猜。';
                rightPanelTitle.textContent = '歷史紀錄';
                aiCard.style.display = 'block';
                algoExplainer.style.display = 'block';
                possibilityTracker.style.display = 'flex';
                historyWrapper.style.display = 'block';
                treeWrapper.style.display = 'none';
                updatePossibilityCount();
            }
        }

        function initGame() {
            allPossibleCodes = generateAllPossible();
            const randomIndex = Math.floor(Math.random() * allPossibleCodes.length);
            secret = allPossibleCodes[randomIndex];

            attempts = 0;
            isGameOver = false;
            userHistory = [];

            historyList.innerHTML = '<li class="empty-history" id="emptyMsg">尚無紀錄</li>';
            treeContainer.innerHTML = '<div class="empty-history">初始狀態：共有 <strong>5,040</strong> 種可能答案。<br>請輸入第一猜開始觀察電腦的推理壓縮過程！</div>';
            
            messageEl.textContent = '';
            messageEl.style.color = '#d9534f';
            aiResultContainer.innerHTML = '點擊按鈕查看 AI 如何進行最佳決策分析。';
            possibleCount.textContent = '5,040 種';
            guessInput.value = '';
            guessInput.disabled = false;
            submitBtn.disabled = false;
            aiHintBtn.disabled = false;
            guessInput.focus();
        }

        function calculateAB(secretStr, guessStr) {
            let a = 0;
            let b = 0;
            for (let i = 0; i < 4; i++) {
                if (guessStr[i] === secretStr[i]) {
                    a++;
                } else if (secretStr.includes(guessStr[i])) {
                    b++;
                }
            }
            return { a, b };
        }

        function getRemainingCandidates() {
            return allPossibleCodes.filter(code => {
                return userHistory.every(h => {
                    const res = calculateAB(code, h.guess);
                    return res.a === h.a && res.b === h.b;
                });
            });
        }

        function updatePossibilityCount() {
            const candidates = getRemainingCandidates();
            possibleCount.textContent = `${candidates.length.toLocaleString()} 種`;
        }

        function validateInput(guess) {
            if (guess.length !== 4 || !/^\d{4}$/.test(guess)) {
                return '請輸入完整的 4 位數字！';
            }
            const uniqueDigits = new Set(guess);
            if (uniqueDigits.size !== 4) {
                return '數字不能重複，請重新輸入！';
            }
            return null;
        }

        function triggerConfetti() {
            if (typeof confetti !== 'function') return;

            const duration = 2.5 * 1000;
            const animationEnd = Date.now() + duration;

            const interval = setInterval(function() {
                const timeLeft = animationEnd - Date.now();
                if (timeLeft <= 0) return clearInterval(interval);

                const particleCount = 50 * (timeLeft / duration);

                confetti({
                    particleCount,
                    startVelocity: 30,
                    spread: 360,
                    ticks: 60,
                    origin: { x: Math.random() * 0.2 + 0.1, y: Math.random() - 0.2 },
                    colors: ['#28a745', '#007bff', '#ffc107', '#dc3545', '#17a2b8']
                });
                confetti({
                    particleCount,
                    startVelocity: 30,
                    spread: 360,
                    ticks: 60,
                    origin: { x: Math.random() * 0.2 + 0.7, y: Math.random() - 0.2 },
                    colors: ['#28a745', '#007bff', '#ffc107', '#dc3545', '#17a2b8']
                });
            }, 200);
        }

        // 渲染階段二：樹狀思考脈絡與遞減柱狀圖
        function renderTree() {
            if (userHistory.length === 0) {
                treeContainer.innerHTML = '<div class="empty-history">初始狀態：共有 <strong>5,040</strong> 種可能答案。<br>請輸入第一猜開始觀察電腦的推理壓縮過程！</div>';
                return;
            }

            let html = `
                <div class="tree-node">
                    <div class="tree-node-header">
                        <span>🌱 初始搜尋空間</span>
                        <strong style="color: #007bff;">5,040 種可能 (100%)</strong>
                    </div>
                    <div class="progress-bg"><div class="progress-bar" style="width: 100%;"></div></div>
                </div>
            `;

            userHistory.forEach((h, index) => {
                const percent = ((h.remainingCount / 5040) * 100).toFixed(1);
                const excluded = index === 0 ? 5040 - h.remainingCount : userHistory[index-1].remainingCount - h.remainingCount;
                
                html += `
                    <div style="text-align: center; color: #888; font-size: 0.8rem; margin: -4px 0;">↓ 排除 ${excluded.toLocaleString()} 種不可能答案</div>
                    <div class="tree-node">
                        <div class="tree-node-header">
                            <span>#${index + 1} 猜 <strong>${h.guess}</strong> ➔ <span class="result-badge">${h.a}A${h.b}B</span></span>
                            <strong style="color: ${h.remainingCount === 1 ? '#28a745' : '#007bff'};">${h.remainingCount.toLocaleString()} 種 (${percent}%)</strong>
                        </div>
                        <div class="progress-bg">
                            <div class="progress-bar" style="width: ${Math.max(percent, 2)}%; background-color: ${h.remainingCount === 1 ? '#28a745' : '#007bff'};"></div>
                        </div>
                        <div class="tree-details">
                            此猜測成功幫你縮減了 <strong>${((1 - h.remainingCount / (index === 0 ? 5040 : userHistory[index-1].remainingCount)) * 100).toFixed(1)}%</strong> 的搜尋範圍。
                        </div>
                    </div>
                `;
            });

            treeContainer.innerHTML = html;
        }

        function handleGuess() {
            if (isGameOver) return;

            const guess = guessInput.value.trim();
            const errorMsg = validateInput(guess);

            if (errorMsg) {
                messageEl.style.color = '#d9534f';
                messageEl.textContent = errorMsg;
                return;
            }

            messageEl.textContent = '';
            attempts++;

            const emptyMsg = document.getElementById('emptyMsg');
            if (emptyMsg) emptyMsg.remove();

            const { a, b } = calculateAB(secret, guess);
            
            userHistory.push({ guess, a, b, remainingCount: 0 });
            const currentCandidates = getRemainingCandidates();
            userHistory[userHistory.length - 1].remainingCount = currentCandidates.length;

            const li = document.createElement('li');
            li.className = 'history-item';
            li.innerHTML = `
                <span>#${attempts}：<strong>${guess}</strong></span>
                <span class="result-badge">${a}A${b}B</span>
            `;
            historyList.prepend(li);

            aiResultContainer.innerHTML = '點擊按鈕查看 AI 如何進行最佳決策分析。';
            updatePossibilityCount();
            renderTree();

            if (a === 4) {
                messageEl.style.color = '#28a745';
                messageEl.textContent = `🎉 恭喜答對！答案是 ${secret}，共猜了 ${attempts} 次！`;
                isGameOver = true;
                guessInput.disabled = true;
                submitBtn.disabled = true;
                aiHintBtn.disabled = true;
                triggerConfetti();
            } else {
                guessInput.value = '';
                guessInput.focus();
            }
        }

        // AI 策略分析算子 (Minimax)
        function getBestGuess() {
            if (isGameOver) return;

            let candidates = getRemainingCandidates();

            if (candidates.length === 0) {
                aiResultContainer.innerHTML = '⚠️ 找不到符合歷史紀錄的答案，請檢查先前是否有輸入錯誤。';
                return;
            }

            if (candidates.length === 1) {
                aiResultContainer.innerHTML = `
                    <div>💡 推薦猜測：<strong>${candidates[0]}</strong></div>
                    <div class="ai-reason">🔍 <strong>策略原因：</strong>可能性已壓縮至唯一確定解答！</div>
                `;
                guessInput.value = candidates[0];
                return;
            }

            if (userHistory.length === 0) {
                aiResultContainer.innerHTML = `
                    <div>💡 開局最佳推薦：<strong>1234</strong></div>
                    <div class="ai-reason">🔍 <strong>策略原因：</strong>目前有 5,040 種可能。選用 4 個不重複數字能最大化取得初始資訊量。</div>
                `;
                guessInput.value = '1234';
                return;
            }

            aiResultContainer.innerHTML = '🤖 AI 正在計算 Minimax 最壞狀況之剩餘可能數...';

            setTimeout(() => {
                let bestGuess = candidates[0];
                let minMaxRemaining = Infinity;

                const testPool = candidates.length > 500 ? candidates.slice(0, 300) : candidates;

                for (let guess of testPool) {
                    let scoreGroups = {};
                    for (let code of candidates) {
                        const { a, b } = calculateAB(code, guess);
                        const key = `${a}A${b}B`;
                        scoreGroups[key] = (scoreGroups[key] || 0) + 1;
                    }

                    let maxRemaining = Math.max(...Object.values(scoreGroups));

                    if (maxRemaining < minMaxRemaining) {
                        minMaxRemaining = maxRemaining;
                        bestGuess = guess;
                    }
                }

                const isCandidate = candidates.includes(bestGuess);
                let reasonText = `當前剩餘 <strong>${candidates.length}</strong> 種答案組合。AI 計算後選擇 <strong>${bestGuess}</strong>，因為這組數字能在最壞狀況下，將剩餘可能數壓縮到剩 <strong>${minMaxRemaining}</strong> 種以下，確保長條圖砍得最深！`;
                
                if (!isCandidate) {
                    reasonText += ` (此數字雖已確定不是答案，但作為工具能取得最大資訊量)`;
                }

                aiResultContainer.innerHTML = `
                    <div>💡 AI 推薦猜測：<strong>${bestGuess}</strong></div>
                    <div class="ai-reason">🔍 <strong>演算法推理原因：</strong><br>${reasonText}</div>
                `;
                guessInput.value = bestGuess;
            }, 50);
        }

        submitBtn.addEventListener('click', handleGuess);
        guessInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') handleGuess();
        });
        restartBtn.addEventListener('click', initGame);
        aiHintBtn.addEventListener('click', getBestGuess);

        initGame();
    </script>
</body>
</html>
