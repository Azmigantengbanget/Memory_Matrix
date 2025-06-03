<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Memory Matrix Game</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #2c3e50;
            color: #ecf0f1;
            margin: 0;
            overflow: hidden; /* Mencegah scroll yang tidak diinginkan */
        }

        .game-container {
            background-color: #34495e;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
            text-align: center;
            width: 90%;
            max-width: 500px;
            
            /* Tetapkan tinggi tetap jika memungkinkan, atau min-height yang cukup besar */
            /* Menggunakan tinggi tetap bisa sangat membantu stabilitas */
            height: 550px; /* Contoh tinggi tetap. Sesuaikan ini agar semua konten muat */
            /* Atau gunakan min-height: 550px; jika Anda ingin sedikit fleksibilitas lebih */
            
            display: flex;
            flex-direction: column;
            /* Pastikan distribusi ruang yang stabil */
            justify-content: space-between; 
            align-items: center;
            box-sizing: border-box; 
        }

        h1 {
            color: #e74c3c;
            margin-bottom: 20px;
        }

        .info {
            display: flex;
            justify-content: center;
            margin-bottom: 20px;
            font-size: 1.2em;
            /* Memberikan tinggi yang pasti agar tidak loncat */
            height: 1.5em; /* Tepat untuk satu baris teks level */
            line-height: 1.5em; /* Pastikan teks berada di tengah vertikal */
        }

        button {
            background-color: #27ae60;
            color: white;
            border: none;
            padding: 12px 25px;
            font-size: 1.1em;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s ease;
            margin-bottom: 20px;
            flex-shrink: 0; /* Pastikan tombol tidak menyusut */
        }

        button:hover {
            background-color: #2ecc71;
        }

        .board-wrapper {
            width: 100%;
            /* Menggunakan max-width pada board-wrapper agar tetap proporsional */
            /* Ini mungkin perlu disesuaikan dengan ukuran grid terbesar (MAX_GRID_SIZE) */
            /* Contoh: Jika MAX_GRID_SIZE = 6, dan cell size ~50px, maka (6*50) + (5*5) = 325px */
            /* Atau biarkan padding-bottom 100% jika lebar container sudah cukup */
            max-width: 400px; /* Maksimum 400px untuk board, sesuaikan jika perlu */
            padding-bottom: 100%; /* Membuat kotak aspek rasio 1:1 relatif terhadap max-width */
            position: relative;
            margin-bottom: 20px;
            border: 3px solid #3498db; 
            border-radius: 8px;
            background-color: #2980b9; 
            overflow: hidden; 
            flex-grow: 1; /* Memungkinkan wrapper mengambil ruang yang tersedia */
            flex-shrink: 1;
            display: flex; /* Tambahkan flex agar game-board bisa ditengah */
            justify-content: center;
            align-items: center;
        }

        .game-board {
            /* Hapus absolute positioning dan biarkan flexbox mengatur */
            /* position: absolute; top: 5px; left: 5px; right: 5px; bottom: 5px; */
            width: calc(100% - 10px); /* Lebar penuh dikurangi padding border wrapper */
            height: calc(100% - 10px); /* Tinggi penuh dikurangi padding border wrapper */
            display: grid;
            gap: 5px;
            
            visibility: hidden;
            opacity: 0;
            transition: visibility 0s, opacity 0.5s ease;
        }

        .game-board.active {
            visibility: visible;
            opacity: 1;
        }

        .cell {
            background-color: #5d6d7e;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.2s ease;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 0;
            aspect-ratio: 1 / 1;
        }

        .cell.active {
            background-color: #f1c40f;
        }

        .cell.selected {
            background-color: #3498db;
        }

        .cell.correct {
            background-color: #27ae60;
        }

        .cell.incorrect {
            background-color: #e74c3c;
        }

        .message {
            /* Ini yang paling penting untuk stabilitas */
            height: 2.5em; /* Set tinggi tetap untuk elemen pesan */
            display: flex; 
            align-items: center; 
            justify-content: center; 
            font-size: 1.3em;
            font-weight: bold;
            color: #ecf0f1;
            margin-top: auto; 
            flex-shrink: 0; /* Pastikan pesan tidak menyusut */
        }
    </style>
</head>
<body>
    <div class="game-container">
        <h1>Memory Matrix</h1>
        <div class="info">
            <p>Level: <span id="level">1</span></p>
        </div>
        <button id="startButton">Mulai Game</button>
        <div id="boardWrapper" class="board-wrapper">
            <div id="gameBoard" class="game-board">
                </div>
        </div>
        <div id="message" class="message"></div>
    </div>

    <script>
        const gameBoard = document.getElementById('gameBoard');
        const boardWrapper = document.getElementById('boardWrapper'); 
        const startButton = document.getElementById('startButton');
        const levelSpan = document.getElementById('level');
        const messageDiv = document.getElementById('message');

        let currentLevel = 1;
        let gridSize = 3; 
        const MAX_GRID_SIZE = 6; // Tetapkan ukuran grid maksimum
        let cellsToLight = 2; 
        let litCells = []; 
        let playerClicks = []; 
        let isPlaying = false; 
        let highlightDuration = 1000; 
        let gameTimeout; 

        function initializeGame() {
            currentLevel = 1;
            gridSize = 3;
            cellsToLight = 2;
            highlightDuration = 1000;
            updateInfo();
            
            clearBoard(); 
            buildBoard(); // Bangun ulang dengan MAX_GRID_SIZE dan sembunyikan yang tidak perlu

            messageDiv.textContent = 'Klik "Mulai Game" untuk memulai!';
            startButton.textContent = 'Mulai Game';
            startButton.style.display = 'block';
            
            gameBoard.classList.remove('active'); 
            gameBoard.style.pointerEvents = 'none'; 
            isPlaying = false;
        }

        function startGame() {
            if (isPlaying) return;
            isPlaying = true;
            startButton.style.display = 'none';
            messageDiv.textContent = '';
            
            buildBoard(); // Pastikan board selalu dibangun ulang dengan MAX_GRID_SIZE
            gameBoard.classList.add('active'); 
            gameBoard.style.pointerEvents = 'none'; 

            gameTimeout = setTimeout(showPattern, 1000);
        }

        function buildBoard() {
            gameBoard.innerHTML = ''; 
            gameBoard.style.gridTemplateColumns = `repeat(${MAX_GRID_SIZE}, 1fr)`; 
            const totalMaxCells = MAX_GRID_SIZE * MAX_GRID_SIZE; 

            for (let i = 0; i < totalMaxCells; i++) {
                const cell = document.createElement('div');
                cell.classList.add('cell');
                cell.dataset.index = i; 
                cell.addEventListener('click', handleCellClick); 
                gameBoard.appendChild(cell);
            }
            
            // Perbarui visibilitas sel setelah membangun board
            updateCellVisibility(); 
        }

        function updateCellVisibility() {
            const allCells = document.querySelectorAll('.cell');
            const currentTotalCells = gridSize * gridSize;

            // Mengatur display sel agar tidak terlihat di luar grid aktif
            allCells.forEach((cell, index) => {
                if (index < currentTotalCells) {
                    cell.style.display = 'flex'; 
                } else {
                    cell.style.display = 'none'; 
                }
            });

            // Penting: Atur grid-template-columns untuk gameBoard itu sendiri
            // agar hanya sel yang aktif yang memengaruhi tata letak grid
            gameBoard.style.gridTemplateColumns = `repeat(${gridSize}, 1fr)`;
        }

        function showPattern() {
            litCells = []; 
            playerClicks = []; 
            const allCells = Array.from(document.querySelectorAll('.cell'));
            const currentTotalCells = gridSize * gridSize; 

            while (litCells.length < cellsToLight) {
                const randomIndex = Math.floor(Math.random() * currentTotalCells); 
                if (!litCells.includes(randomIndex)) { 
                    litCells.push(randomIndex);
                }
            }

            let delay = 0;
            litCells.forEach((index) => {
                gameTimeout = setTimeout(() => {
                    allCells[index].classList.add('active'); 
                    gameTimeout = setTimeout(() => {
                        allCells[index].classList.remove('active'); 
                    }, highlightDuration);
                }, delay);
                delay += 150; 
            });

            gameTimeout = setTimeout(() => {
                messageDiv.textContent = 'Klik kotak yang tadi menyala!';
                gameBoard.style.pointerEvents = 'auto'; 
            }, delay + highlightDuration + 200);
        }

        function handleCellClick(event) {
            if (!isPlaying || playerClicks.length >= cellsToLight) return;

            const clickedIndex = parseInt(event.target.dataset.index);

            // Pastikan yang diklik adalah sel yang aktif/terlihat
            if (clickedIndex >= (gridSize * gridSize)) return;

            if (playerClicks.includes(clickedIndex)) return;

            playerClicks.push(clickedIndex); 
            event.target.classList.add('selected'); 

            if (playerClicks.length === cellsToLight) {
                gameBoard.style.pointerEvents = 'none'; 
                gameTimeout = setTimeout(checkAnswer, 500);
            }
        }

        function checkAnswer() {
            gameBoard.style.pointerEvents = 'none'; 

            const sortedLitCells = [...litCells].sort((a, b) => a - b);
            const sortedPlayerClicks = [...playerClicks].sort((a, b) => a - b);

            let isCorrect = true;
            if (sortedLitCells.length !== sortedPlayerClicks.length) {
                isCorrect = false;
            } else {
                for (let i = 0; i < sortedLitCells.length; i++) {
                    if (sortedLitCells[i] !== sortedPlayerClicks[i]) {
                        isCorrect = false;
                        break;
                    }
                }
            }

            if (isCorrect) {
                messageDiv.textContent = 'BENAR!';
                highlightCorrectCells(); 
                gameTimeout = setTimeout(nextLevel, 1500); 
            } else {
                messageDiv.textContent = `SALAH! Game Over.`;
                highlightIncorrectCells(); 
                isPlaying = false;
                
                gameBoard.classList.remove('active'); 

                gameTimeout = setTimeout(() => {
                    initializeGame(); 
                    startGame();      
                }, 2000); 
            }
            updateInfo(); 
        }

        function highlightCorrectCells() {
            const allCells = document.querySelectorAll('.cell');
            litCells.forEach(index => {
                allCells[index].classList.add('correct');
                allCells[index].classList.remove('selected'); 
            });
        }

        function highlightIncorrectCells() {
            const allCells = document.querySelectorAll('.cell');
            litCells.forEach(index => {
                allCells[index].classList.add('correct');
                allCells[index].classList.remove('selected'); 
            });
            playerClicks.forEach(index => {
                if (!litCells.includes(index)) { 
                    allCells[index].classList.add('incorrect');
                    allCells[index].classList.remove('selected'); 
                }
            });
        }

        function nextLevel() {
            clearBoard(); 
            currentLevel++;
            updateDifficulty(); 
            updateInfo(); 
            
            updateCellVisibility(); // Penting: ini akan mengatur `grid-template-columns` gameBoard
            gameBoard.style.pointerEvents = 'none'; 
            gameTimeout = setTimeout(showPattern, 1000); 
        }

        function updateDifficulty() {
            cellsToLight++;

            if (currentLevel % 3 === 0 && gridSize < MAX_GRID_SIZE) {
                gridSize++;
            }

            const currentTotalCells = gridSize * gridSize;
            if (cellsToLight >= currentTotalCells) {
                cellsToLight = Math.floor(currentTotalCells / 2); 
                if (cellsToLight < 2) cellsToLight = 2; 
            }
            
            if (currentLevel > 5 && highlightDuration > 300) {
                highlightDuration -= 50;
            } else if (currentLevel > 10 && highlightDuration > 100) {
                highlightDuration -= 25;
            }
        }

        function updateInfo() {
            levelSpan.textContent = currentLevel;
        }

        function clearBoard() {
            const cells = document.querySelectorAll('.cell');
            cells.forEach(cell => {
                cell.classList.remove('active', 'selected', 'correct', 'incorrect');
            });
        }

        startButton.addEventListener('click', () => {
            if (!isPlaying) {
                startGame();
            }
        });

        initializeGame();
    </script>
</body>
</html>
