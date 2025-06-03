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
            
            /* Fleksibilitas dan Stabilitas Utama */
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: center;
            min-height: 450px; /* Tingkatkan min-height untuk menampung grid yang lebih besar dan pesan */
            box-sizing: border-box; /* Pastikan padding termasuk dalam lebar/tinggi */
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
            /* Tambahkan min-height agar tidak loncat */
            min-height: 1.5em; /* Cukup untuk satu baris teks level */
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
        }

        button:hover {
            background-color: #2ecc71;
        }

        /* Container untuk board yang akan mempertahankan ukuran */
        .board-wrapper {
            width: 100%;
            padding-bottom: 100%; /* Membuat kotak aspek rasio 1:1 */
            position: relative;
            margin-bottom: 20px;
            border: 3px solid #3498db; /* Border biru untuk board */
            border-radius: 8px;
            background-color: #2980b9; /* Warna latar belakang board */
            overflow: hidden; /* Pastikan tidak ada overflow dari sel */
        }

        .game-board {
            position: absolute;
            top: 5px; /* Sesuaikan dengan padding wrapper */
            left: 5px; /* Sesuaikan dengan padding wrapper */
            right: 5px; /* Sesuaikan dengan padding wrapper */
            bottom: 5px; /* Sesuaikan dengan padding wrapper */
            display: grid;
            gap: 5px;
            /* Hapus padding dan border dari sini karena sudah di wrapper */
            /* padding: 5px; */
            /* border-radius: 8px; */
            /* background-color: #2980b9; */
            
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
            /* Pastikan tinggi ini selalu stabil */
            min-height: 2.5em; /* Cukup untuk dua baris pesan atau pesan panjang */
            display: flex; /* Untuk memusatkan teks vertikal jika hanya satu baris */
            align-items: center; /* Memusatkan teks secara vertikal */
            justify-content: center; /* Memusatkan teks secara horizontal */
            font-size: 1.3em;
            font-weight: bold;
            color: #ecf0f1;
            margin-top: auto; /* Dorong pesan ke bawah */
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
        const boardWrapper = document.getElementById('boardWrapper'); // Ambil wrapper board
        const startButton = document.getElementById('startButton');
        const levelSpan = document.getElementById('level');
        const messageDiv = document.getElementById('message');

        let currentLevel = 1;
        let gridSize = 3; 
        const MAX_GRID_SIZE = 6; // Tetapkan ukuran grid maksimum yang akan selalu ada di DOM
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
            
            // Perlu clear board sebelum buildBoard di inisialisasi agar semua sel tersembunyi
            clearBoard(); // Memastikan semua highlight dihapus
            buildBoard(); // Pastikan board dibangun ulang dengan ukuran awal

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
            
            buildBoard(); // Pastikan board dibangun ulang untuk level saat ini
            gameBoard.classList.add('active'); 
            gameBoard.style.pointerEvents = 'none'; 

            gameTimeout = setTimeout(showPattern, 1000);
        }

        function buildBoard() {
            // Kita akan selalu membangun board dengan ukuran MAX_GRID_SIZE
            // dan mengatur tampilan sel yang relevan dengan gridSize saat ini.
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
            
            // Sembunyikan sel yang tidak relevan dengan gridSize saat ini
            updateCellVisibility(); 
        }

        function updateCellVisibility() {
            const allCells = document.querySelectorAll('.cell');
            const currentTotalCells = gridSize * gridSize;

            allCells.forEach((cell, index) => {
                if (index < currentTotalCells) {
                    cell.style.display = 'flex'; /* Tampilkan sel yang relevan */
                } else {
                    cell.style.display = 'none'; /* Sembunyikan sel yang tidak digunakan */
                }
            });
        }

        function showPattern() {
            litCells = []; 
            playerClicks = []; 
            const allCells = Array.from(document.querySelectorAll('.cell'));
            const currentTotalCells = gridSize * gridSize; // Hanya pilih dari sel yang aktif

            while (litCells.length < cellsToLight) {
                const randomIndex = Math.floor(Math.random() * currentTotalCells); // Pilih dari sel yang terlihat
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

            // Penting: Pastikan yang diklik adalah sel yang aktif/terlihat
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
            // Tidak perlu buildBoard lagi jika gridnya statis, hanya update visibilitas
            updateCellVisibility(); // Penting untuk menampilkan/menyembunyikan sel sesuai gridSize baru
            gameBoard.style.pointerEvents = 'none'; 
            gameTimeout = setTimeout(showPattern, 1000); 
        }

        function updateDifficulty() {
            cellsToLight++;

            // Jika level sudah cukup tinggi, tingkatkan gridSize
            if (currentLevel % 3 === 0 && gridSize < MAX_GRID_SIZE) {
                gridSize++;
            }

            const currentTotalCells = gridSize * gridSize;
            if (cellsToLight >= currentTotalCells) {
                cellsToLight = Math.floor(currentTotalCells / 2); 
                if (cellsToLight < 2) cellsToLight = 2; 
            }
            
            // Mempersingkat durasi highlight
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

        // Inisialisasi game saat halaman pertama kali dimuat
        initializeGame();
    </script>
</body>
</html>
