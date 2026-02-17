<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body { background-color: #fff0f5; text-align: center; font-family: sans-serif; margin: 0; padding: 10px; }
        h1 { color: #ff1493; font-size: 20px; margin: 10px 0; }
        #ui { background: white; padding: 10px; border: 2px solid #ffb6c1; border-radius: 15px; display: inline-block; margin-bottom: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); width: 90%; max-width: 400px; }
        #target-name { font-size: 26px; font-weight: bold; color: #ff1493; display: block; min-height: 35px; }
        #timer { font-size: 16px; color: #666; font-weight: bold; }
        
        #game-board { 
            width: 100%; max-width: 600px; margin: 0 auto; background: white; 
            border-radius: 10px; border: 3px solid #ffb6c1; padding: 5px; 
            min-height: 400px; display: flex; align-items: center; justify-content: center;
        }
        
        /* 都道府県の基本スタイル */
        path, polygon { fill: #f0f0f0; stroke: #ccc; stroke-width: 0.5; cursor: pointer; transition: fill 0.2s; }
        path:hover, polygon:hover { fill: #ffe4e1; }
        
        /* 正解時の色：!importantで上書きを確実に */
        .correct { fill: #ff69b4 !important; stroke: #ff1493 !important; pointer-events: none; }
        
        @keyframes miss-ani { 0% { fill: #ffb6c1; } 100% { fill: #f0f0f0; } }
        .miss { animation: miss-ani 0.5s; }
        
        svg { width: 100%; height: auto; max-height: 75vh; }
    </style>
</head>
<body>

    <h1>🌸 日本地図パズル 🌸</h1>

    <div id="ui">
        タイム: <span id="timer">00:00</span>
        <div style="margin-top:5px;">次にえらぶ：<span id="target-name">地図を読み込み中...</span></div>
    </div>

    <div id="game-board">
        <p>地図データを取得しています...</p>
    </div>

    <script>
        const prefNames = ["北海道", "青森県", "岩手県", "宮城県", "秋田県", "山形県", "福島県", "茨城県", "栃木県", "群馬県", "埼玉県", "千葉県", "東京都", "神奈川県", "新潟県", "富山県", "石川県", "福井県", "山梨県", "長野県", "岐阜県", "静岡県", "愛知県", "三重県", "滋賀県", "京都府", "大阪府", "兵庫県", "奈良県", "和歌山県", "鳥取県", "島根県", "岡山県", "広島県", "山口県", "徳島県", "香川県", "愛媛県", "高知県", "福岡県", "佐賀県", "長崎県", "熊本県", "大分県", "宮崎県", "鹿児島県", "沖縄県"];

        let targetIndex = 0;
        let shuffled = [];
        let startTime;
        let isClear = false;

        const board = document.getElementById('game-board');
        const targetText = document.getElementById('target-name');
        const timerText = document.getElementById('timer');

        async function loadMap() {
            try {
                const response = await fetch('https://geolonia.github.io/japanese-prefectures/map-full.svg');
                if (!response.ok) throw new Error();
                const svgText = await response.text();
                board.innerHTML = svgText;
                initGame();
            } catch (error) {
                board.innerHTML = "<p style='color:red;'>読み込みエラー。ネット接続を確認してください。</p>";
            }
        }

        // 判定ロジック：titleタグから「県名」を正確に抜き出す
        function getCleanName(el) {
            const titleEl = el.querySelector('title') || (el.parentNode && el.parentNode.querySelector('title'));
            if (!titleEl) return "";
            // 「東京都 / Tokyo」から「東京都」だけを抽出
            return titleEl.textContent.split('/')[0].replace(/\s+/g, "").trim();
        }

        function initGame() {
            shuffled = [...prefNames].sort(() => Math.random() - 0.5);
            startTime = Date.now();
            
            // イベントリスナーを1つにまとめて管理（効率化）
            board.addEventListener('click', (e) => {
                if (isClear) return;
                
                const el = e.target.closest('path, polygon');
                if (!el) return;

                const clickedName = getCleanName(el);
                const targetName = shuffled[targetIndex];

                // 柔軟な一致判定（念のため includes を使用）
                if (clickedName !== "" && (clickedName === targetName || clickedName.includes(targetName))) {
                    // 同じ名前を持つすべてのパーツ（離島など）をピンクにする
                    const allPaths = board.querySelectorAll('path, polygon');
                    allPaths.forEach(p => {
                        if (getCleanName(p) === clickedName) {
                            p.classList.add('correct');
                        }
                    });
                    
                    targetIndex++;
                    nextQuestion();
                } else if (clickedName !== "") {
                    el.classList.add('miss');
                    setTimeout(() => el.classList.remove('miss'), 500);
                }
            });

            nextQuestion();
            setInterval(updateTimer, 1000);
        }

        function nextQuestion() {
            if (targetIndex < shuffled.length) {
                targetText.innerText = shuffled[targetIndex];
            } else {
                isClear = true;
                targetText.innerText = "🌸 クリア！ 🌸";
                setTimeout(() => alert("全県クリア！\nタイム: " + timerText.innerText), 100);
            }
        }

        function updateTimer() {
            if (isClear || !startTime) return;
            const now = Math.floor((Date.now() - startTime) / 1000);
            const m = Math.floor(now / 60).toString().padStart(2, '0');
            const s = (now % 60).toString().padStart(2, '0');
            timerText.innerText = m + ":" + s;
        }

        loadMap();
    </script>
</body>
</html>
