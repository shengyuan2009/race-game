<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>卡通賽車大對決</title>
    <style>
        /* CSS 樣式設定：打造可愛的卡通遊戲介面 */
        body {
            margin: 0;
            padding: 0;
            background: #70c5ce; /* 卡通風格的天空藍背景 */
            font-family: 'Arial', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
        }

        h1 {
            color: #fff;
            text-shadow: 3px 3px 0px #ff595e;
            margin-bottom: 10px;
            font-size: 2rem;
        }

        #game-container {
            position: relative;
            box-shadow: 0 10px 25px rgba(0,0,0,0.3);
            border: 6px solid #fff;
            border-radius: 15px;
            background: #333;
        }

        canvas {
            display: block;
            background: #555;
        }

        /* 遊戲選單與控制說明 */
        .ui-panel {
            margin-top: 15px;
            background: #fff;
            padding: 10px 20px;
            border-radius: 10px;
            text-align: center;
            box-shadow: 0 4px 10px rgba(0,0,0,0.2);
        }

        button {
            background: #ffca3a;
            border: none;
            padding: 10px 20px;
            font-size: 1rem;
            font-weight: bold;
            border-radius: 5px;
            cursor: pointer;
            box-shadow: 0 4px 0 #e0a800;
            transition: 0.1s;
            margin: 0 5px;
        }

        button:active {
            transform: translateY(4px);
            box-shadow: 0 0 0 #e0a800;
        }

        .controls-info {
            font-size: 0.85rem;
            color: #555;
            margin-top: 8px;
        }
    </style>
</head>
<body>

    <h1>🏎️ 卡通賽車大對決 🏁</h1>

    <!-- 遊戲主畫面容器 -->
    <div id="game-container">
        <!-- 畫布：用 JavaScript 繪製賽道與賽車 -->
        <canvas id="gameCanvas" width="800" height="500"></canvas>
    </div>

    <!-- 介面控制面板 -->
    <div class="ui-panel">
        <div id="menu">
            <button onclick="startGame('ai')">單人挑戰 AI</button>
            <button onclick="startGame('pvp')">雙人對戰模式</button>
        </div>
        <div class="controls-info">
            <strong>玩家 1 (紅車):</strong> W / S 加速減速, A / D 左右轉<br>
            <strong>玩家 2 (綠車):</strong> 上 / 下 加速減速, 左 / 右 左右轉 (雙人模式啟用)
        </div>
    </div>

    <script>
        // 取得畫布與繪圖上下文
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // 遊戲狀態變數
        let gameMode = null; // 'ai' 或 'pvp'
        let isPlaying = false;

        // 定義車輛類別 (Car Class)
        class Car {
            constructor(x, y, color, isAI = false) {
                this.x = x;          // 車輛 X 座標
                this.y = y;          // 車輛 Y 座標
                this.angle = 0;      // 車輛朝向角度
                this.speed = 0;      // 當前速度
                this.maxSpeed = 6;   // 最高速度
                this.minSpeed = -2;  // 倒車最高速度
                this.accel = 0.15;   // 加速度
                this.friction = 0.05;// 阻力
                this.color = color;  // 車身顏色
                this.isAI = isAI;    // 是否為 AI 操控
                this.laps = 0;       // 已完成圈數
                this.finished = false;
            }

            // 更新車輛狀態
            update(keys, trackPoints) {
                if (this.isAI) {
                    // AI 簡單自動駕駛邏輯：朝著預设軌跡前進
                    this.speed = 3.5;
                    this.angle += 0.02; // 簡單的圓形軌跡模擬
                } else {
                    // 玩家操控邏輯
                    if (keys.up) {
                        this.speed = Math.min(this.speed + this.accel, this.maxSpeed);
                    } else if (keys.down) {
                        this.speed = Math.max(this.speed - this.accel, this.minSpeed);
                    } else {
                        // 自然減速 (摩擦力)
                        if (this.speed > 0) this.speed = Math.max(0, this.speed - this.friction);
                        if (this.speed < 0) this.speed = Math.min(0, this.speed + this.friction);
                    }

                    if (keys.left && Math.abs(this.speed) > 0.5) {
                        this.angle -= 0.04 * (this.speed / this.maxSpeed);
                    }
                    if (keys.right && Math.abs(this.speed) > 0.5) {
                        this.angle += 0.04 * (this.speed / this.maxSpeed);
                    }
                }

                // 根據速度與角度計算新座標
                this.x += Math.cos(this.angle) * this.speed;
                this.y += Math.sin(this.angle) * this.speed;
            }

            // 繪製卡通風格車輛
            draw() {
                ctx.save();
                ctx.translate(this.x, this.y); // 移動到車輛中心
                ctx.rotate(this.angle);        // 旋轉車輛角度

                // 畫車身陰影
                ctx.fillStyle = 'rgba(0,0,0,0.2)';
                ctx.fillRect(-18, -12, 36, 24);

                // 畫車身本體 (卡通圓角風格)
                ctx.fillStyle = this.color;
                ctx.beginPath();
                ctx.roundRect(-16, -10, 32, 20, 6);
                ctx.fill();
                ctx.strokeStyle = '#222';
                ctx.lineWidth = 2;
                ctx.stroke();

                // 畫車窗 (擋風玻璃)
                ctx.fillStyle = '#a0e7e5';
                ctx.fillRect(-2, -7, 10, 14);

                // 畫輪胎
                ctx.fillStyle = '#222';
                ctx.fillRect(-12, -13, 8, 4);
                ctx.fillRect(4, -13, 8, 4);
                ctx.fillRect(-12, 9, 8, 4);
                ctx.fillRect(4, 9, 8, 4);

                ctx.restore();
            }
        }

        // 初始化遊戲角色
        let player1 = new Car(400, 250, '#ff595e', false); // 紅車 (玩家1)
        let player2 = new Car(400, 280, '#4ea8de', false); // 藍/綠車 (玩家2 或 AI)

        // 鍵盤狀態監聽
        const keys1 = { up: false, down: false, left: false, right: false };
        const keys2 = { up: false, down: false, left: false, right: false };

        window.addEventListener('keydown', (e) => {
            if (e.key === 'w') keys1.up = true;
            if (e.key === 's') keys1.down = true;
            if (e.key === 'a') keys1.left = true;
            if (e.key === 'd') keys1.right = true;

            if (e.key === 'ArrowUp') keys2.up = true;
            if (e.key === 'ArrowDown') keys2.down = true;
            if (e.key === 'ArrowLeft') keys2.left = true;
            if (e.key === 'ArrowRight') keys2.right = true;
        });

        window.addEventListener('keyup', (e) => {
            if (e.key === 'w') keys1.up = false;
            if (e.key === 's') keys1.down = false;
            if (e.key === 'a') keys1.left = false;
            if (e.key === 'd') keys1.right = false;

            if (e.key === 'ArrowUp') keys2.up = false;
            if (e.key === 'ArrowDown') keys2.down = false;
            if (e.key === 'ArrowLeft') keys2.left = false;
            if (e.key === 'ArrowRight') keys2.right = false;
        });

        // 啟動遊戲函式
        function startGame(mode) {
            gameMode = mode;
            isPlaying = true;
            player1 = new Car(370, 230, '#ff595e', false);
            
            if (mode === 'ai') {
                player2 = new Car(370, 280, '#38b000', true); // 綠色 AI 對手
            } else {
                player2 = new Car(370, 280, '#4ea8de', false); // 藍色玩家 2
            }
        }

        // 繪製卡通賽道
        function drawTrack() {
            // 草地背景
            ctx.fillStyle = '#80ed99';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // 外賽道
            ctx.fillStyle = '#495057';
            ctx.beginPath();
            ctx.ellipse(400, 250, 350, 200, 0, 0, Math.PI * 2);
            ctx.fill();

            // 內草地（形成甜甜圈賽道）
            ctx.fillStyle = '#80ed99';
            ctx.beginPath();
            ctx.ellipse(400, 250, 200, 90, 0, 0, Math.PI * 2);
            ctx.fill();

            // 起终點線
            ctx.strokeStyle = '#fff';
            ctx.lineWidth = 4;
            ctx.setLineDash([5, 5]);
            ctx.beginPath();
            ctx.moveTo(400, 50);
            ctx.lineTo(400, 160);
            ctx.stroke();
            ctx.setLineDash([]); // 重設虛線
        }

        // 遊戲主循環 (Game Loop)
        function gameLoop() {
            // 清除畫面
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            if (isPlaying) {
                // 1. 繪製賽道
                drawTrack();

                // 2. 更新並繪製玩家 1
                player1.update(keys1);
                player1.draw();

                // 3. 更新並繪製玩家 2 或 AI
                if (gameMode === 'ai') {
                    player2.update({}, null); // AI 自動運行
                } else {
                    player2.update(keys2);
                }
                player2.draw();

                // 4. 顯示計分與模式資訊
                ctx.fillStyle = '#fff';
                ctx.font = 'bold 18px Arial';
                ctx.fillText(`模式: ${gameMode === 'ai' ? '單人 AI 挑戰' : '雙人對戰'}`, 20, 35);
            } else {
                // 尚未開始遊戲時的畫面
                drawTrack();
                ctx.fillStyle = 'rgba(0, 0, 0, 0.4)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);

                ctx.fillStyle = '#fff';
                ctx.font = 'bold 36px Arial';
                ctx.textAlign = 'center';
                ctx.fillText('準備好開始賽車了嗎？', canvas.width / 2, canvas.height / 2 - 20);
                ctx.font = '18px Arial';
                ctx.fillText('請點擊下方按鈕選擇遊戲模式', canvas.width / 2, canvas.height / 2 + 20);
                ctx.textAlign = 'left'; // 恢復預設
            }

            // 循環執行動畫
            requestAnimationFrame(gameLoop);
        }

        // 啟動畫面循環
        gameLoop();
    </script>
</body>
</html>
