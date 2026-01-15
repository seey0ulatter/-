<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>网页版消消乐-多关卡专属元素版</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: Arial, sans-serif;
        }

        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            background-color: #f0f8ff;
            padding: 20px;
        }

        .game-container {
            background-color: #fff;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
            padding: 20px;
            text-align: center;
        }

        .game-info {
            display: flex;
            justify-content: space-between;
            width: 400px;
            margin-bottom: 15px;
            font-size: 18px;
            font-weight: bold;
        }

        .level, .score, .time {
            color: #333;
        }

        #game-board {
            display: grid;
            grid-template-columns: repeat(8, 50px);
            grid-gap: 5px;
            background-color: #e0e0e0;
            padding: 10px;
            border-radius: 5px;
        }

        .tile {
            width: 50px;
            height: 50px;
            border-radius: 5px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 24px;
            cursor: pointer;
            transition: transform 0.2s, background-color 0.2s;
        }

        .tile:hover {
            transform: scale(1.05);
        }

        .tile.selected {
            border: 3px solid #ff6b6b;
        }

        .tile.matched {
            animation: pop 0.5s ease forwards;
        }

        @keyframes pop {
            0% { transform: scale(1); }
            50% { transform: scale(1.2); opacity: 0.8; }
            100% { transform: scale(0); opacity: 0; }
        }

        .controls {
            margin-top: 20px;
        }

        button {
            padding: 10px 20px;
            font-size: 16px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin: 0 10px;
            transition: background-color 0.3s;
        }

        button:hover {
            background-color: #45a049;
        }

        #restart-btn {
            background-color: #ff9800;
        }

        #restart-btn:hover {
            background-color: #e68900;
        }

        .game-over {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            display: none;
            align-items: center;
            justify-content: center;
            z-index: 100;
        }

        .game-over-content {
            background-color: white;
            padding: 40px;
            border-radius: 10px;
            text-align: center;
        }

        .game-over h2 {
            font-size: 36px;
            margin-bottom: 20px;
            color: #ff6b6b;
        }

        .game-over p {
            font-size: 24px;
            margin-bottom: 30px;
        }

        .level-up {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(255, 255, 255, 0.95);
            padding: 30px 60px;
            border-radius: 10px;
            box-shadow: 0 0 30px rgba(0, 0, 0, 0.2);
            font-size: 28px;
            font-weight: bold;
            color: #4CAF50;
            display: none;
            z-index: 99;
        }

        .level-theme {
            font-size: 20px;
            color: #666;
            margin-bottom: 10px;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <div class="game-container">
        <div class="level-theme" id="level-theme">第一关 - 水果主题</div>
        <div class="game-info">
            <div class="level">关卡: <span id="level">1</span></div>
            <div class="score">本关得分: <span id="score">0</span> / <span id="target-score">50</span></div>
            <div class="time">时间: <span id="timer">60</span>秒</div>
        </div>
        <div id="game-board"></div>
        <div class="controls">
            <button id="start-btn">开始游戏</button>
            <button id="restart-btn">重新开始</button>
        </div>
    </div>

    <div class="level-up" id="level-up">
        恭喜进入第 <span id="new-level">2</span> 关！<br>
        <span id="new-theme">蔬菜主题</span>
    </div>

    <div class="game-over" id="game-over">
        <div class="game-over-content">
            <h2 id="game-over-title">游戏结束!</h2>
            <p>最终关卡: <span id="final-level">1</span></p>
            <p>最后一关得分: <span id="final-score">0</span></p>
            <button id="play-again-btn">再来一局</button>
        </div>
    </div>

    <script>
        // 游戏配置 - 按关卡划分专属元素+独立计分
        const config = {
            rows: 8,
            cols: 8,
            minMatch: 3,
            // 关卡配置：每关专属元素、目标分数、时间限制
            levels: [
                { 
                    level: 1, 
                    targetScore: 50, 
                    timeLimit: 60, 
                    theme: "水果主题", 
                    elements: ['🍎', '🍌', '🍇', '🍓', '🍉', '🍒', '🍑', '🥭'] 
                },
                { 
                    level: 2, 
                    targetScore: 100, 
                    timeLimit: 50, 
                    theme: "蔬菜主题", 
                    elements: ['🥕', '🥦', '🥬', '🍅', '🥔', '🌽', '🥑', '🧅'] 
                },
                { 
                    level: 3, 
                    targetScore: 150, 
                    timeLimit: 40, 
                    theme: "动物主题", 
                    elements: ['🐶', '🐱', '🐭', '🐹', '🐰', '🦊', '🐻', '🐼'] 
                },
                { 
                    level: 4, 
                    targetScore: 200, 
                    timeLimit: 35, 
                    theme: "甜品主题", 
                    elements: ['🍦', '🍰', '🍩', '🍪', '🍫', '🍬', '🍭', '🍮'] 
                },
                { 
                    level: 5, 
                    targetScore: 250, 
                    timeLimit: 30, 
                    theme: "交通工具", 
                    elements: ['🚗', '🚕', '🚙', '🚌', '🚎', '🏎️', '🚓', '🚑'] 
                }
            ]
        };

        // 游戏状态 - 新增本关得分（独立计分）
        let gameState = {
            board: [],
            currentLevelScore: 0, // 本关独立分数（非累计）
            timeLeft: config.levels[0].timeLimit,
            isPlaying: false,
            timerInterval: null,
            selectedTile: null,
            currentLevel: 1,
            targetScore: config.levels[0].targetScore,
            currentElements: config.levels[0].elements // 当前关卡专属元素
        };

        // DOM 元素
        const gameBoard = document.getElementById('game-board');
        const scoreDisplay = document.getElementById('score');
        const targetScoreDisplay = document.getElementById('target-score');
        const timerDisplay = document.getElementById('timer');
        const levelDisplay = document.getElementById('level');
        const levelThemeDisplay = document.getElementById('level-theme');
        const startBtn = document.getElementById('start-btn');
        const restartBtn = document.getElementById('restart-btn');
        const gameOverModal = document.getElementById('game-over');
        const finalScoreDisplay = document.getElementById('final-score');
        const finalLevelDisplay = document.getElementById('final-level');
        const playAgainBtn = document.getElementById('play-again-btn');
        const levelUpModal = document.getElementById('level-up');
        const newLevelDisplay = document.getElementById('new-level');
        const newThemeDisplay = document.getElementById('new-theme');
        const gameOverTitle = document.getElementById('game-over-title');

        // 初始化游戏棋盘（适配当前关卡专属元素）
        function initBoard() {
            gameBoard.innerHTML = '';
            gameState.board = [];
            
            // 使用当前关卡的专属元素创建棋盘
            const elements = gameState.currentElements;
            for (let row = 0; row < config.rows; row++) {
                gameState.board[row] = [];
                for (let col = 0; col < config.cols; col++) {
                    const randomElement = elements[Math.floor(Math.random() * elements.length)];
                    gameState.board[row][col] = {
                        value: randomElement,
                        matched: false
                    };

                    const tile = document.createElement('div');
                    tile.className = 'tile';
                    tile.dataset.row = row;
                    tile.dataset.col = col;
                    tile.textContent = randomElement;
                    tile.addEventListener('click', handleTileClick);
                    gameBoard.appendChild(tile);
                }
            }
        }

        // 处理格子点击
        function handleTileClick(e) {
            if (!gameState.isPlaying) return;

            const tile = e.target;
            const row = parseInt(tile.dataset.row);
            const col = parseInt(tile.dataset.col);

            if (!gameState.selectedTile) {
                gameState.selectedTile = { row, col };
                tile.classList.add('selected');
                return;
            }

            if (gameState.selectedTile.row === row && gameState.selectedTile.col === col) {
                tile.classList.remove('selected');
                gameState.selectedTile = null;
                return;
            }

            const isAdjacent = 
                (Math.abs(gameState.selectedTile.row - row) === 1 && gameState.selectedTile.col === col) ||
                (Math.abs(gameState.selectedTile.col - col) === 1 && gameState.selectedTile.row === row);

            if (!isAdjacent) {
                document.querySelector(`.tile[data-row="${gameState.selectedTile.row}"][data-col="${gameState.selectedTile.col}"]`)
                    .classList.remove('selected');
                gameState.selectedTile = { row, col };
                tile.classList.add('selected');
                return;
            }

            swapTiles(gameState.selectedTile.row, gameState.selectedTile.col, row, col);
            const matches = findMatches();
            
            if (matches.length > 0) {
                updateScore(matches.length);
                markMatches(matches);
                document.querySelector(`.tile[data-row="${gameState.selectedTile.row}"][data-col="${gameState.selectedTile.col}"]`)
                    .classList.remove('selected');
                gameState.selectedTile = null;
                
                setTimeout(() => {
                    removeMatchedTiles();
                    fillEmptySpaces();
                }, 500);
            } else {
                setTimeout(() => {
                    swapTiles(row, col, gameState.selectedTile.row, gameState.selectedTile.col);
                    document.querySelector(`.tile[data-row="${gameState.selectedTile.row}"][data-col="${gameState.selectedTile.col}"]`)
                        .classList.remove('selected');
                    gameState.selectedTile = null;
                }, 300);
            }
        }

        // 交换两个格子
        function swapTiles(row1, col1, row2, col2) {
            const temp = gameState.board[row1][col1].value;
            gameState.board[row1][col1].value = gameState.board[row2][col2].value;
            gameState.board[row2][col2].value = temp;

            const tile1 = document.querySelector(`.tile[data-row="${row1}"][data-col="${col1}"]`);
            const tile2 = document.querySelector(`.tile[data-row="${row2}"][data-col="${col2}"]`);
            
            const tempText = tile1.textContent;
            tile1.textContent = tile2.textContent;
            tile2.textContent = tempText;
        }

        // 查找所有匹配
        function findMatches() {
            const matches = [];
            
            // 检查水平匹配
            for (let row = 0; row < config.rows; row++) {
                let currentValue = gameState.board[row][0].value;
                let currentMatch = [{ row, col: 0 }];
                
                for (let col = 1; col < config.cols; col++) {
                    if (gameState.board[row][col].value === currentValue) {
                        currentMatch.push({ row, col });
                    } else {
                        if (currentMatch.length >= config.minMatch) {
                            matches.push(...currentMatch);
                        }
                        currentValue = gameState.board[row][col].value;
                        currentMatch = [{ row, col }];
                    }
                }
                
                if (currentMatch.length >= config.minMatch) {
                    matches.push(...currentMatch);
                }
            }
            
            // 检查垂直匹配
            for (let col = 0; col < config.cols; col++) {
                let currentValue = gameState.board[0][col].value;
                let currentMatch = [{ row: 0, col }];
                
                for (let row = 1; row < config.rows; row++) {
                    if (gameState.board[row][col].value === currentValue) {
                        currentMatch.push({ row, col });
                    } else {
                        if (currentMatch.length >= config.minMatch) {
                            matches.push(...currentMatch);
                        }
                        currentValue = gameState.board[row][col].value;
                        currentMatch = [{ row, col }];
                    }
                }
                
                if (currentMatch.length >= config.minMatch) {
                    matches.push(...currentMatch);
                }
            }
            
            return matches;
        }

        // 标记匹配的格子
        function markMatches(matches) {
            matches.forEach(match => {
                gameState.board[match.row][match.col].matched = true;
                const tile = document.querySelector(`.tile[data-row="${match.row}"][data-col="${match.col}"]`);
                tile.classList.add('matched');
            });
        }

        // 移除匹配的格子
        function removeMatchedTiles() {
            for (let row = 0; row < config.rows; row++) {
                for (let col = 0; col < config.cols; col++) {
                    if (gameState.board[row][col].matched) {
                        gameState.board[row][col].value = '';
                        gameState.board[row][col].matched = false;
                    }
                }
            }
        }

        // 填充空的格子（使用当前关卡专属元素）
        function fillEmptySpaces() {
            const elements = gameState.currentElements;
            for (let col = 0; col < config.cols; col++) {
                const values = [];
                for (let row = 0; row < config.rows; row++) {
                    if (gameState.board[row][col].value !== '') {
                        values.push(gameState.board[row][col].value);
                    }
                }
                
                while (values.length < config.rows) {
                    values.unshift(elements[Math.floor(Math.random() * elements.length)]);
                }
                
                for (let row = 0; row < config.rows; row++) {
                    gameState.board[row][col].value = values[row];
                    const tile = document.querySelector(`.tile[data-row="${row}"][data-col="${col}"]`);
                    tile.textContent = values[row];
                    tile.classList.remove('matched');
                }
            }
            
            const newMatches = findMatches();
            if (newMatches.length > 0) {
                updateScore(newMatches.length);
                markMatches(newMatches);
                setTimeout(() => {
                    removeMatchedTiles();
                    fillEmptySpaces();
                }, 500);
            }
        }

        // 更新分数 - 本关独立计分（不累计）
        function updateScore(matchCount) {
            // 仅累加本关分数，过关后重置
            gameState.currentLevelScore += matchCount * 10;
            scoreDisplay.textContent = gameState.currentLevelScore;
            
            // 检查是否达到当前关卡目标分数
            if (gameState.currentLevelScore >= gameState.targetScore) {
                if (gameState.currentLevel < config.levels.length) {
                    // 进入下一关：重置本关分数，加载新关卡配置
                    gameState.currentLevel++;
                    const currentLevelConfig = config.levels[gameState.currentLevel - 1];
                    gameState.targetScore = currentLevelConfig.targetScore;
                    gameState.currentElements = currentLevelConfig.elements;
                    
                    // 显示关卡升级提示（含新主题）
                    newLevelDisplay.textContent = gameState.currentLevel;
                    newThemeDisplay.textContent = `${currentLevelConfig.theme}`;
                    levelUpModal.style.display = 'block';
                    
                    // 2秒后关闭提示，初始化新关卡
                    setTimeout(() => {
                        levelUpModal.style.display = 'none';
                        // 更新UI
                        levelDisplay.textContent = gameState.currentLevel;
                        levelThemeDisplay.textContent = `第${gameState.currentLevel}关 - ${currentLevelConfig.theme}`;
                        targetScoreDisplay.textContent = currentLevelConfig.targetScore;
                        // 重置本关分数和时间，保留游戏状态
                        gameState.currentLevelScore = 0;
                        scoreDisplay.textContent = 0;
                        gameState.timeLeft = currentLevelConfig.timeLimit;
                        timerDisplay.textContent = gameState.timeLeft;
                        // 生成新关卡专属元素的棋盘
                        initBoard();
                    }, 2000);
                } else {
                    // 所有关卡通关
                    gameOver(true);
                }
            }
        }

        // 开始游戏 - 初始化当前关卡配置
        function startGame() {
            // 获取当前关卡配置
            const currentLevelConfig = config.levels[gameState.currentLevel - 1];
            
            // 重置游戏状态（仅本关分数、时间）
            gameState.currentLevelScore = 0;
            gameState.timeLeft = currentLevelConfig.timeLimit;
            gameState.isPlaying = true;
            gameState.selectedTile = null;
            gameState.currentElements = currentLevelConfig.elements;
            
            // 更新UI显示
            scoreDisplay.textContent = 0;
            targetScoreDisplay.textContent = currentLevelConfig.targetScore;
            timerDisplay.textContent = currentLevelConfig.timeLimit;
            levelDisplay.textContent = gameState.currentLevel;
            levelThemeDisplay.textContent = `第${gameState.currentLevel}关 - ${currentLevelConfig.theme}`;
            
            // 隐藏弹窗
            gameOverModal.style.display = 'none';
            levelUpModal.style.display = 'none';
            
            // 初始化当前关卡的棋盘
            initBoard();
            
            // 启动计时器
            if (gameState.timerInterval) {
                clearInterval(gameState.timerInterval);
            }
            
            gameState.timerInterval = setInterval(() => {
                gameState.timeLeft--;
                timerDisplay.textContent = gameState.timeLeft;
                
                if (gameState.timeLeft <= 0) {
                    gameOver(false);
                }
            }, 1000);
            
            // 禁用开始按钮
            startBtn.disabled = true;
            startBtn.style.opacity = '0.5';
        }

        // 结束游戏
        function gameOver(isWon = false) {
            gameState.isPlaying = false;
            clearInterval(gameState.timerInterval);
            
            // 更新结束弹窗内容
            finalScoreDisplay.textContent = gameState.currentLevelScore;
            finalLevelDisplay.textContent = gameState.currentLevel;
            
            if (isWon) {
                gameOverTitle.textContent = '恭喜通关！';
                gameOverTitle.style.color = '#4CAF50';
            } else {
                gameOverTitle.textContent = '时间到！';
                gameOverTitle.style.color = '#ff6b6b';
            }
            
            // 显示结束弹窗
            gameOverModal.style.display = 'flex';
            
            // 启用开始按钮
            startBtn.disabled = false;
            startBtn.style.opacity = '1';
        }

        // 重新开始游戏（重置为第一关）
        function restartGame() {
            gameState.currentLevel = 1;
            gameState.targetScore = config.levels[0].targetScore;
            gameState.currentElements = config.levels[0].elements;
            gameOver(false);
            setTimeout(startGame, 300);
        }

        // 事件监听
        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', restartGame);
        playAgainBtn.addEventListener('click', () => {
            gameState.currentLevel = 1;
            gameState.targetScore = config.levels[0].targetScore;
            gameState.currentElements = config.levels[0].elements;
            startGame();
        });

        // 初始化
        initBoard();
        // 初始化目标分数和主题显示
        targetScoreDisplay.textContent = config.levels[0].targetScore;
        levelThemeDisplay.textContent = `第1关 - ${config.levels[0].theme}`;
    </script>
</body>
</html>
