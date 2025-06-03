<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Memory Matrix Game</title>
    <style>
        /* CSS untuk styling game */
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #2c3e50;
            color: #ecf0f1;
            margin: 0;
            /* Mencegah overflow yang bisa memicu zoom */
            overflow: hidden;
            /* Mencegah penyesuaian ukuran teks otomatis di mobile */
            -webkit-text-size-adjust: 100%;
        }

        .game-container {
            background-color: #34495e;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
            text-align: center;
            width: 90%;
            max-width: 500px;
            /* Tambahan: Pastikan container tidak melebihi tinggi viewport */
            max-height: 95vh;
            /* Gunakan flexbox untuk mengelola ruang secara vertikal */
            display: flex;
            flex-direction: column;
            justify-content: center; /* Memusatkan konten vertikal */
            align-items: center;
            box-sizing: border-box;
        }

        h1 {
            color: #e74c3c;
            margin-bottom: 20px;
        }

        /* Kelas .info diubah agar hanya menampilkan level */
        .info {
            display: flex;
            justify-content: center; /* Pusat di tengah */
            margin-bottom: 20px;
            font-size: 1.2em;
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

        .game-board {
            display: grid;
            gap: 5px;
            border: 3px solid #3498db;
            padding: 5px;
            border-radius: 8px;
            background-color: #2980b9;
            margin-bottom: 20px;
            opacity: 0; /* Sembunyikan board awalnya */
            transition: opacity 0.5s ease;

            /* --- Perbaikan utama di sini untuk stabilitas --- */
            /* Berikan ukuran pasti agar tidak ada perubahan layout */
            width: calc(100% - 10px); /* 100% lebar container - padding border */
            max-width: 450px; /* Batasi lebar maksimum */
            aspect-ratio: 1 / 1; /* Pastikan selalu persegi */
            box-sizing: border-box; /* Padding dan border termasuk dalam ukuran */

            /* Tambahan: Pastikan elemen ini selalu ada di layout, meskipun opacity 0 */
            /* Ini akan mencegah browser menghitung ulang ukuran saat muncul */
            min-height: 100px; /* Nilai placeholder, akan di-override oleh JS */
            flex-shrink: 0; /* Mencegah elemen ini mengecil jika ruang terbatas */
        }

        .game-board.active {
            opacity: 1; /* Tampilkan saat aktif */
        }

        .cell {
            /* width dan height tetap di auto karena CSS Grid yang mengaturnya */
            background-color: #5d6d7e;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.2s ease;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 0; /* Sembunyikan angka */
            aspect-ratio: 1 / 1; /* Menjaga aspek rasio kotak */
            box-sizing: border-box;

            /* Penting: Mencegah zoom browser pada sentuhan */
            touch-action: manipulation;
            /* Mencegah pemilihan teks yang bisa memicu zoom */
            user-select: none;
            -webkit-user-select: none;
            -moz-user-select: none;
            -ms-user-select: none;
        }

        .cell.active {
            background-color: #f1c40f; /* Warna saat menyala */
        }

        .cell.selected {
            background-color: #3498db; /* Warna saat diklik pemain */
        }

        .cell.correct {
            background-color: #27ae60; /* Warna saat benar */
        }

        .cell.incorrect {
            background-color: #e74c3c; /* Warna saat salah */
        }

        .message {
            font-size: 1.3em;
            font-weight: bold;
            color: #ecf0f1;
            margin-top: 20px; /* Jarak dari board */
            word-wrap: break-word; /* Mencegah teks panjang meluber */
            padding: 0 10px;
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
        <div id="gameBoard" class="game-board">
            </div>
        <div id="message" class="message"></div>
    </div>

    <script>
        // JavaScript untuk logika game
        const gameBoard = document.getElementById('gameBoard');
        const startButton = document.getElementById('startButton');
        const levelSpan = document.getElementById('level');
        const messageDiv = document.getElementById('message');

        let currentLevel = 1;
        let gridSize = 3; // Dimulai dengan 3x3
        let cellsToLight = 2; // Dimulai dengan 2 kotak menyala
        let litCells = []; // Menyimpan indeks kotak yang menyala
        let playerClicks = []; // Menyimpan indeks kotak yang diklik pemain
        let isPlaying = false;
        let highlightDuration = 1000; // Durasi kotak menyala (ms)
        let gameTimeout; // Untuk menyimpan ID timeout agar bisa dibatalkan

        /**
         * Menginisialisasi ulang semua parameter game ke kondisi awal.
         * Dipanggil saat game pertama kali dimuat atau saat restart setelah salah.
         */
        function initializeGame() {
            currentLevel = 1;
            gridSize = 3;
            cellsToLight = 2;
            highlightDuration = 1000; // Reset durasi highlight
            updateInfo();
            clearBoard(); // Pastikan board bersih dari kelas highlight
            messageDiv.textContent = 'Klik "Mulai Game" untuk memulai!';
            startButton.textContent = 'Mulai Game'; // Pastikan teks tombol kembali ke 'Mulai Game'
            startButton.style.display = 'block'; // Tampilkan tombol mulai
            gameBoard.classList.remove('active'); // Sembunyikan board (opacity 0) saat inisialisasi
            gameBoard.style.pointerEvents = 'none'; // Pastikan board tidak bisa diklik saat tidak aktif
            isPlaying = false; // Pastikan status game tidak sedang bermain
        }

        /**
         * Memulai siklus game.
         * Dipanggil saat tombol "Mulai Game" diklik atau saat game restart otomatis.
         */
        function startGame() {
            if (isPlaying) return; // Jika game sudah berjalan, jangan mulai lagi
            isPlaying = true; // Set status sedang bermain
            startButton.style.display = 'none'; // Sembunyikan tombol mulai
            messageDiv.textContent = ''; // Bersihkan pesan
            gameBoard.classList.add('active'); // Tampilkan board dengan efek fade-in
            gameBoard.style.pointerEvents = 'none'; // Pastikan board tidak bisa diklik selama pola ditampilkan

            buildBoard(); // Bangun ulang grid
            // Beri jeda sebentar sebelum menampilkan pola
            gameTimeout = setTimeout(showPattern, 1000);
        }

        /**
         * Membangun atau membangun ulang elemen grid di DOM.
         * Mengatur jumlah kolom CSS Grid berdasarkan gridSize.
         */
        function buildBoard() {
            gameBoard.innerHTML = ''; // Hapus semua sel sebelumnya
            gameBoard.style.gridTemplateColumns = `repeat(${gridSize}, 1fr)`; // Atur kolom CSS Grid
            const totalCells = gridSize * gridSize; // Hitung total sel

            for (let i = 0; i < totalCells; i++) {
                const cell = document.createElement('div');
                cell.classList.add('cell');
                cell.dataset.index = i; // Simpan indeks sel untuk identifikasi
                cell.addEventListener('click', handleCellClick); // Tambahkan event listener klik
                gameBoard.appendChild(cell);
            }
        }

        /**
         * Menampilkan pola kotak yang menyala kepada pemain.
         * Sel dipilih secara acak dan menyala satu per satu.
         */
        function showPattern() {
            litCells = []; // Reset array kotak yang menyala
            playerClicks = []; // Reset array klik pemain
            const allCells = Array.from(document.querySelectorAll('.cell'));
            const totalCells = gridSize * gridSize;

            // Pilih kotak secara acak untuk menyala
            while (litCells.length < cellsToLight) {
                const randomIndex = Math.floor(Math.random() * totalCells);
                if (!litCells.includes(randomIndex)) { // Pastikan tidak ada duplikasi
                    litCells.push(randomIndex);
                }
            }

            // Nyalakan kotak satu per satu dengan jeda
            let delay = 0;
            litCells.forEach((index) => {
                gameTimeout = setTimeout(() => {
                    allCells[index].classList.add('active'); // Tambah kelas 'active' untuk menyala
                    // Matikan setelah durasi highlight
                    gameTimeout = setTimeout(() => {
                        allCells[index].classList.remove('active'); // Hapus kelas 'active'
                    }, highlightDuration);
                }, delay);
                delay += 150; // Jeda antar kotak menyala
            });

            // Setelah semua kotak menyala dan mati, biarkan pemain mengklik
            gameTimeout = setTimeout(() => {
                messageDiv.textContent = 'Klik kotak yang tadi menyala!';
                gameBoard.style.pointerEvents = 'auto'; // Aktifkan klik pada board
            }, delay + highlightDuration + 200); // Tambahan jeda untuk transisi yang mulus
        }

        /**
         * Menangani klik pemain pada sel grid.
         * Menyimpan indeks sel yang diklik dan memicu checkAnswer jika sudah cukup klik.
         */
        function handleCellClick(event) {
            // Pastikan game sedang dimainkan dan belum semua klik terdaftar
            if (!isPlaying || playerClicks.length >= cellsToLight) return;

            const clickedIndex = parseInt(event.target.dataset.index);

            // Mencegah klik ganda pada kotak yang sama
            if (playerClicks.includes(clickedIndex)) return;

            playerClicks.push(clickedIndex); // Tambahkan indeks klik pemain
            event.target.classList.add('selected'); // Tandai kotak yang diklik dengan warna biru

            if (playerClicks.length === cellsToLight) {
                // Setelah semua klik, beri jeda sebentar lalu cek jawaban
                gameTimeout = setTimeout(checkAnswer, 500);
            }
        }

        /**
         * Memeriksa apakah jawaban pemain benar atau salah.
         * Memberikan feedback visual dan menentukan apakah game berlanjut atau restart.
         */
        function checkAnswer() {
            gameBoard.style.pointerEvents = 'none'; // Nonaktifkan klik saat mengecek

            // Sortir kedua array untuk perbandingan yang mudah
            const sortedLitCells = [...litCells].sort((a, b) => a - b);
            const sortedPlayerClicks = [...playerClicks].sort((a, b) => a - b);

            let isCorrect = true;
            // Cek apakah jumlah klik sama dan setiap elemen cocok
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
                highlightCorrectCells(); // Tampilkan kotak yang benar dengan hijau
                gameTimeout = setTimeout(nextLevel, 1500); // Lanjut ke level berikutnya setelah jeda
            } else {
                messageDiv.textContent = `SALAH! Game Over.`; // Pesan skor dihapus
                highlightIncorrectCells(); // Tampilkan yang benar (hijau) dan yang salah (merah)
                isPlaying = false; // Set status game berakhir
                gameBoard.classList.remove('active'); // Sembunyikan board (opacity 0) untuk efek transisi ke restart

                // Setelah menampilkan pesan error sebentar, langsung mulai ulang game
                gameTimeout = setTimeout(() => {
                    initializeGame(); // Reset semua parameter ke level 1
                    startGame();      // Langsung mulai level 1
                }, 2000); // Jeda 2 detik sebelum restart otomatis
            }
            updateInfo(); // Perbarui info level di UI
        }

        /**
         * Memberi highlight hijau pada semua sel yang benar (yang seharusnya diklik).
         */
        function highlightCorrectCells() {
            const allCells = document.querySelectorAll('.cell');
            litCells.forEach(index => {
                allCells[index].classList.add('correct');
                allCells[index].classList.remove('selected'); // Hapus seleksi biru
            });
        }

        /**
         * Memberi highlight hijau pada sel yang benar dan merah pada sel yang salah diklik.
         */
        function highlightIncorrectCells() {
            const allCells = document.querySelectorAll('.cell');
            // Tandai yang benar dengan hijau
            litCells.forEach(index => {
                allCells[index].classList.add('correct');
                allCells[index].classList.remove('selected'); // Hapus seleksi biru
            });
            // Tandai yang salah dengan merah
            playerClicks.forEach(index => {
                if (!litCells.includes(index)) { // Hanya yang diklik tapi salah
                    allCells[index].classList.add('incorrect');
                    allCells[index].classList.remove('selected'); // Hapus seleksi biru
                }
            });
        }

        /**
         * Mempersiapkan game untuk level berikutnya.
         * Meningkatkan level dan menyesuaikan kesulitan.
         */
        function nextLevel() {
            clearBoard(); // Bersihkan semua highlight dari board
            currentLevel++;
            updateDifficulty(); // Sesuaikan kesulitan untuk level baru
            updateInfo();
            buildBoard(); // Bangun ulang board (akan berubah ukuran jika gridSize bertambah)
            gameTimeout = setTimeout(showPattern, 1000); // Mulai pola untuk level baru
        }

        /**
         * Menyesuaikan parameter kesulitan game (jumlah kotak menyala, ukuran grid, durasi highlight)
         * berdasarkan level saat ini.
         */
        function updateDifficulty() {
            // Setiap level, tingkatkan jumlah kotak yang menyala.
            cellsToLight++;

            // Setiap 3 level, tingkatkan ukuran grid (misal: dari 3x3 ke 4x4, lalu ke 5x5).
            if (currentLevel % 3 === 0) {
                gridSize++;
            }

            // Pastikan jumlah kotak yang menyala tidak melebihi total kotak di grid
            // Jika jumlah yang menyala terlalu banyak, reset ke sebagian dari grid baru agar tidak terlalu mudah.
            const maxPossibleLitCells = gridSize * gridSize;
            if (cellsToLight >= maxPossibleLitCells) {
                cellsToLight = Math.floor(maxPossibleLitCells / 2); // Misalnya, setengah dari total sel
                if (cellsToLight < 2) cellsToLight = 2; // Pastikan minimal 2 sel menyala
            }

            // Persingkat durasi highlight di level yang lebih tinggi untuk meningkatkan kesulitan.
            if (currentLevel > 5 && highlightDuration > 300) {
                highlightDuration -= 50;
            } else if (currentLevel > 10 && highlightDuration > 100) { // Lebih agresif di level sangat tinggi
                highlightDuration -= 25;
            }
        }

        /**
         * Memperbarui teks level di antarmuka pengguna.
         */
        function updateInfo() {
            levelSpan.textContent = currentLevel;
        }

        /**
         * Menghapus semua kelas highlight dari sel grid.
         */
        function clearBoard() {
            const cells = document.querySelectorAll('.cell');
            cells.forEach(cell => {
                // Hapus semua kelas yang berhubungan dengan status kotak (menyala, diklik, benar, salah)
                cell.classList.remove('active', 'selected', 'correct', 'incorrect');
            });
        }

        // --- Event Listener ---
        // Menangani klik pada tombol "Mulai Game"
        startButton.addEventListener('click', () => {
            // Hanya mulai game jika status isPlaying adalah false (game belum berjalan)
            if (!isPlaying) {
                startGame(); // Panggil fungsi startGame untuk memulai
            }
        });

        // Inisialisasi game saat halaman pertama kali dimuat
        initializeGame();
    </script>
</body>
</html>
