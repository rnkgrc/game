# game
ゲーム
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>Pastel Block Blast Game</title>
    <style>
        /* 全体のスタイル */
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #f0f0f0; /* 背景を薄いグレーに */
            margin: 0;
            font-family: 'Arial', sans-serif;
            flex-direction: column;
        }

        /* メインコンテナ */
        #main-wrapper {
            display: flex;
            gap: 20px;
            position: relative;
        }

        /* ゲームフィールドのスタイル */
        #game-container {
            border: 5px solid #aaa;
            background-color: #ffffff; /* 背景を白に */
            display: inline-block;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
        }

        /* 情報パネル（スコアなど） */
        #info-panel {
            width: 150px;
            padding: 10px;
            background-color: #fff;
            border: 3px solid #aaa;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.05);
            text-align: center;
            height: fit-content;
        }
        #score-display {
            font-size: 2em;
            font-weight: bold;
            color: #333;
            margin-bottom: 20px;
        }
        #controls-info {
            font-size: 0.8em;
            text-align: left;
            padding: 5px;
            color: #555;
            border-top: 1px dashed #ccc;
            padding-top: 10px;
            margin-top: 10px;
        }
        #controls-info p {
            margin: 5px 0;
        }


        /* 各セル（ブロックが存在する可能性のある場所）のスタイル */
        .cell {
            width: 30px;
            height: 30px;
            box-sizing: border-box;
            border: 1px solid #eee; /* セルの境界線は薄く */
            float: left;
            transition: background-color 0.1s; /* 色変化を滑らかに */
        }

        /* パステルカラーの定義 */
        .block-coral { background-color: #ffb3b3; border: 2px outset #ffcccc; box-shadow: 0 0 5px rgba(0, 0, 0, 0.1); } /* パステルレッド/コーラル */
        .block-sky { background-color: #b3ccff; border: 2px outset #cce0ff; box-shadow: 0 0 5px rgba(0, 0, 0, 0.1); } /* パステルブルー/スカイ */
        .block-mint { background-color: #b3ffcc; border: 2px outset #ccffdd; box-shadow: 0 0 5px rgba(0, 0, 0, 0.1); } /* パステルグリーン/ミント */
        .block-lemon { background-color: #ffffb3; border: 2px outset #ffffcc; box-shadow: 0 0 5px rgba(0, 0, 0, 0.1); } /* パステルイエロー/レモン */
        .block-lavender { background-color: #e0b3ff; border: 2px outset #eeddff; box-shadow: 0 0 5px rgba(0, 0, 0, 0.1); } /* パステルパープル/ラベンダー */
        .block-peach { background-color: #ffddb3; border: 2px outset #ffeecc; box-shadow: 0 0 5px rgba(0, 0, 0, 0.1); } /* パステルオレンジ/ピーチ */
        .block-aqua { background-color: #b3fff5; border: 2px outset #ccfffa; box-shadow: 0 0 5px rgba(0, 0, 0, 0.1); } /* パステルシアン/アクア */

        /* セルの行をクリアするための clearfix */
        .row::after {
            content: "";
            clear: both;
            display: table;
        }

        /* ゲームオーバー画面 */
        #game-over-screen {
            display: none; /* 初期状態では非表示 */
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(255, 255, 255, 0.9); /* 半透明の白 */
            z-index: 100;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            text-align: center;
            font-size: 3em;
            color: #d9534f; /* 警告色 */
            font-weight: bold;
            border-radius: 5px;
        }

        #game-over-screen p {
            margin: 10px 0;
        }

        #final-score {
            font-size: 1.5em;
            color: #333;
        }

        #restart-button {
            padding: 10px 20px;
            font-size: 1em;
            cursor: pointer;
            background-color: #5bc0de; /* ボタンの色 */
            color: white;
            border: none;
            border-radius: 5px;
            margin-top: 20px;
            box-shadow: 0 4px #46b8da;
            transition: all 0.1s;
        }

        #restart-button:active {
            box-shadow: 0 1px #46b8da;
            transform: translateY(3px);
        }

    </style>
</head>
<body>

<div id="main-wrapper">
    <div id="game-container">
        <div id="game-over-screen">
            <p>GAME OVER</p>
            <div id="final-score"></div>
            <button id="restart-button">リスタート</button>
        </div>
        </div>

    <div id="info-panel">
        <h2>SCORE</h2>
        <div id="score-display">0</div>
        
        <div id="controls-info">
            <p>操作方法:</p>
            <p>← → : 移動</p>
            <p>↓ : 落下</p>
            <p>↑ または Space : 回転</p>
        </div>
    </div>
</div>

<script>
    // --- ゲーム設定 ---
    const COLS = 10;
    const ROWS = 20;
    const BLOCK_SIZE = 30;
    let DROP_INTERVAL = 1000; // ブロックが自動で落ちる間隔 (ミリ秒)

    // パステルカラーに合わせた色のクラス名
    const COLORS = ['coral', 'sky', 'mint', 'lemon', 'lavender', 'peach', 'aqua'];

    // ブロックの定義（テトリス風に）
    const SHAPES = [
        [[1, 1, 1, 1]],
        [[1, 1], [1, 1]],
        [[0, 1, 0], [1, 1, 1]],
        [[1, 1, 0], [0, 1, 1]],
        [[0, 1, 1], [1, 1, 0]],
        [[1, 0, 0], [1, 1, 1]],
        [[0, 0, 1], [1, 1, 1]]
    ];

    // --- ゲーム変数 ---
    let board = [];
    let currentBlock;
    let currentBlockX;
    let currentBlockY;
    let currentBlockColor; // 色のクラス名 (例: 'coral')
    let gameLoopInterval;
    let score = 0;
    let isGameOver = false;

    // --- DOM要素 ---
    const container = document.getElementById('game-container');
    const scoreDisplay = document.getElementById('score-display');
    const gameOverScreen = document.getElementById('game-over-screen');
    const finalScoreDisplay = document.getElementById('final-score');
    const restartButton = document.getElementById('restart-button');

    // --- 初期化 ---

    /** ゲームの初期状態を設定 */
    function initGame() {
        // 変数をリセット
        board = [];
        score = 0;
        isGameOver = false;
        DROP_INTERVAL = 1000;

        // DOMリセット
        container.innerHTML = '';
        gameOverScreen.style.display = 'none';
        scoreDisplay.textContent = score;

        initBoard();
        spawnBlock();
        // 既存のインターバルをクリアしてから再設定
        if (gameLoopInterval) clearInterval(gameLoopInterval);
        gameLoopInterval = setInterval(dropBlock, DROP_INTERVAL);
    }
    
    /** ゲームフィールドの初期化とDOMの生成 */
    function initBoard() {
        container.style.width = `${COLS * BLOCK_SIZE}px`;
        container.style.height = `${ROWS * BLOCK_SIZE}px`;

        for (let y = 0; y < ROWS; y++) {
            board[y] = [];
            const rowDiv = document.createElement('div');
            rowDiv.className = 'row';
            rowDiv.style.height = `${BLOCK_SIZE}px`;

            for (let x = 0; x < COLS; x++) {
                board[y][x] = 0;
                const cellDiv = document.createElement('div');
                cellDiv.className = 'cell';
                cellDiv.id = `cell-${x}-${y}`;
                rowDiv.appendChild(cellDiv);
            }
            container.appendChild(rowDiv);
        }
        // ゲームオーバー画面を再配置 (field生成後にoverlayとして表示できるように)
        container.appendChild(gameOverScreen);
    }

    /** ランダムな新しいブロックを生成 */
    function spawnBlock() {
        const randomIndex = Math.floor(Math.random() * SHAPES.length);
        currentBlock = SHAPES[randomIndex];
        currentBlockColor = COLORS[randomIndex];
        currentBlockX = Math.floor(COLS / 2) - Math.floor(currentBlock[0].length / 2);
        currentBlockY = 0;

        if (!isValidMove(currentBlock, currentBlockX, currentBlockY)) {
            // ブロックが生成位置に置けない場合はゲームオーバー
            gameOver();
        }
    }

    /** ブロックが移動可能かチェック */
    function isValidMove(shape, dx, dy) {
        for (let y = 0; y < shape.length; y++) {
            for (let x = 0; x < shape[y].length; x++) {
                if (shape[y][x]) {
                    const newX = dx + x;
                    const newY = dy + y;

                    // フィールドの境界外チェック
                    if (newX < 0 || newX >= COLS || newY >= ROWS) {
                        return false;
                    }

                    // 既にブロックが存在する場所との衝突チェック
                    if (newY >= 0 && board[newY] && board[newY][newX]) {
                        return false;
                    }
                }
            }
        }
        return true;
    }

    /** 現在のブロックをフィールドに描画/非描画する */
    function drawBlock(shape, dx, dy, colorClass) {
        for (let y = 0; y < shape.length; y++) {
            for (let x = 0; x < shape[y].length; x++) {
                if (shape[y][x]) {
                    const cell = document.getElementById(`cell-${dx + x}-${dy + y}`);
                    if (cell) {
                        if (colorClass) {
                            cell.className = `cell block-${colorClass}`;
                        } else {
                            // 非描画（クラスを削除して基本のセルに戻す）
                            cell.className = 'cell';
                        }
                    }
                }
            }
        }
    }

    /** フィールド全体を描画する */
    function refreshBoard() {
        if (isGameOver) return;
        
        // 1. 固定されたブロックを描画
        for (let y = 0; y < ROWS; y++) {
            for (let x = 0; x < COLS; x++) {
                const cell = document.getElementById(`cell-${x}-${y}`);
                if (cell) {
                    if (board[y][x] > 0) {
                        // 1以上の値を持つ場合は固定されたブロック
                        const colorIndex = board[y][x] - 1;
                        cell.className = `cell block-${COLORS[colorIndex]}`;
                    } else {
                        // 空のセル
                        cell.className = 'cell';
                    }
                }
            }
        }

        // 2. 現在操作中のブロックを描画 (boardの状態とは別)
        drawBlock(currentBlock, currentBlockX, currentBlockY, currentBlockColor);
    }


    // --- メインゲームループ処理 ---

    /** ブロックを一段下に落とす */
    function dropBlock() {
        if (isGameOver) return;

        if (moveBlock(0, 1)) {
            // 移動成功
            refreshBoard();
        } else {
            // 移動失敗（何かに衝突した）-> ブロックを固定
            lockBlock();
            // 列の削除チェックとスコア更新
            checkAndClearLines();
            // 新しいブロックを生成
            spawnBlock();
            refreshBoard();
        }
    }

    /** ブロックを現在の位置に固定し、board配列に書き込む */
    function lockBlock() {
        const colorId = COLORS.indexOf(currentBlockColor) + 1;

        for (let y = 0; y < currentBlock.length; y++) {
            for (let x = 0; x < currentBlock[y].length; x++) {
                if (currentBlock[y][x]) {
                    if (currentBlockY + y >= 0) {
                        board[currentBlockY + y][currentBlockX + x] = colorId;
                    }
                }
            }
        }
    }

    /** 横一列揃った行をチェックし、削除する */
    function checkAndClearLines() {
        let linesCleared = 0;
        for (let y = ROWS - 1; y >= 0; y--) {
            // 行がすべて埋まっているかチェック
            if (board[y].every(cell => cell > 0)) {
                // 行を削除
                board.splice(y, 1);
                // 新しい空行を一番上に追加
                board.unshift(new Array(COLS).fill(0));
                linesCleared++;
                // 削除した行の分、yの位置を戻して再チェック
                y++;
            }
        }
        if (linesCleared > 0) {
            updateScore(linesCleared);
        }
    }

    /** スコアを更新 */
    function updateScore(linesCleared) {
        // スコアリング (例: 1行100点、2行300点、3行500点、4行800点)
        const scoreMultipliers = [0, 100, 300, 500, 800];
        const addedScore = scoreMultipliers[linesCleared] || 0;
        score += addedScore;
        scoreDisplay.textContent = score;

        // 難易度（落下速度）の調整
        // 5000点ごとに速度アップ（例）
        if (score > 0 && Math.floor(score / 5000) > Math.floor((score - addedScore) / 5000)) {
            DROP_INTERVAL = Math.max(100, DROP_INTERVAL * 0.85); // 速度を速く (最低100ms)
            clearInterval(gameLoopInterval);
            gameLoopInterval = setInterval(dropBlock, DROP_INTERVAL);
        }
    }

    /** ブロックを移動させる（描画はrefreshBoardで行う） */
    function moveBlock(dx, dy) {
        if (isGameOver) return false;
        
        if (isValidMove(currentBlock, currentBlockX + dx, currentBlockY + dy)) {
            // 古い位置から削除
            // drawBlock(currentBlock, currentBlockX, currentBlockY, null); // refreshBoardで全体を描画し直すため不要

            // 新しい位置に更新
            currentBlockX += dx;
            currentBlockY += dy;
            
            return true;
        }
        return false;
    }

    /** ブロックを回転させる */
    function rotateBlock() {
        if (isGameOver) return;
        
        // 新しい形状を生成（行列を入れ替えて逆順にする = 90度回転）
        const newShape = currentBlock[0].map((_, colIndex) =>
            currentBlock.map(row => row[colIndex]).reverse()
        );

        if (isValidMove(newShape, currentBlockX, currentBlockY)) {
            currentBlock = newShape;
            refreshBoard();
        } else {
            // 回転できなかった場合（壁際での調整など、より複雑な処理も可能ですがここではシンプルに）
        }
    }
    
    /** ゲームオーバー処理 */
    function gameOver() {
        isGameOver = true;
        clearInterval(gameLoopInterval);
        
        finalScoreDisplay.textContent = `最終スコア: ${score}`;
        // ゲームオーバー画面を表示
        gameOverScreen.style.display = 'flex';
        
        // キー操作のリスナーを一時的に無効にする (isGameOverフラグで制御)
    }


    // --- イベントリスナー（キー操作） ---
    document.addEventListener('keydown', (e) => {
        if (isGameOver) return;

        let moved = false;
        switch (e.key) {
            case 'ArrowLeft': // 左矢印
                moved = moveBlock(-1, 0);
                break;
            case 'ArrowRight': // 右矢印
                moved = moveBlock(1, 0);
                break;
            case 'ArrowDown': // 下矢印 (1段下げる)
                // 自動落下より早くする
                dropBlock();
                e.preventDefault(); // スクロールを防ぐ
                return;
            case 'ArrowUp': // 上矢印 (回転)
            case ' ': // スペースキー (回転)
                rotateBlock();
                e.preventDefault(); // スクロールを防ぐ
                return;
            default:
                return;
        }

        if (moved) {
            refreshBoard();
        }
    });
    
    // リスタートボタンのイベントリスナー
    restartButton.addEventListener('click', () => {
        initGame();
    });

    // ゲーム開始
    initGame();

</script>

</body>
</html>
