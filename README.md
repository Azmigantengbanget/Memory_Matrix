<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Memory Matrix Game</title>
    <style>
        /* CSS untuk styling game */
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh; /* Memastikan body mengisi seluruh tinggi viewport */
            background-color: #2c3e50; /* Warna latar belakang gelap */
            color: #ecf0f1; /* Warna teks terang */
            margin: 0;
        }

        .game-container {
            background-color: #34495e; /* Warna latar belakang kontainer game */
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.5); /* Bayangan untuk efek kedalaman */
            text-align: center;
            width: 90%; /* Lebar kontainer 90% dari parent */
            max-width: 500px; /* Lebar maksimum kontainer */
            
            /* Penambahan kunci untuk stabilitas layout */
            min-height: 400px; /* Memberikan tinggi minimum agar layout tidak menyusut drastis */
            display: flex; /* Menggunakan flexbox untuk tata letak internal */
            flex-direction: column; /* Mengatur item dalam kolom vertikal */
            justify-content: space-between; /* Mendistribusikan item secara merata dengan ruang di antaranya */
            align-items: center; /* Memusatkan item secara horizontal */
        }

        h1 {
            color: #e74c3c; /* Warna judul merah */
            margin-bottom: 20px;
        }

        .info {
            display: flex;
            justify-content: center; /* Memusatkan teks level */
            margin-bottom: 20px;
            font-size: 1.2em;
        }

        button {
            background-color: #27ae60; /* Warna tombol hijau */
            color: white;
            border: none;
            padding: 12px 25px;
            font-size: 1.1em;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s ease; /* Transisi halus saat hover */
            margin-bottom: 20px;
        }

        button:hover {
            background-color: #2ecc71; /* Warna tombol saat di-hover */
        }

        .game-board {
            display: grid; /* Menggunakan CSS Grid untuk tata letak kotak */
            gap: 5px; /* Jarak antar kotak */
            border: 3px solid #3498db; /* Border biru untuk board */
            padding: 5px;
            border-radius: 8px;
            background-color: #2980b9; /* Warna latar belakang board */
            margin-bottom: 20px;
            width: 100%; /* Memastikan board mengisi lebar penuh kontainer */

            /* Perbaikan stabilitas: Menggunakan visibility dan opacity */
            visibility: hidden; /* Sembunyikan board awalnya */
            opacity: 0; /* Mulai dengan transparan */
            transition: visibility 0s, opacity 0.5s ease; /* Transisi untuk opacity */
        }

        .game-board.active {
            visibility: visible; /* Tampilkan board saat aktif */
            opacity: 1; /* Transisi menjadi terlihat */
        }

        .cell {
            background-color: #5d6d7e; /* Warna default kotak */
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.2s ease; /* Transisi warna saat di-hover/diklik */
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 0; /* Sembunyikan angka di dalam kotak */
            aspect-ratio: 1 / 1; /* Menjaga aspek rasio kotak agar selalu persegi */
        }

        .cell.active {
            background-color: #f1c40f; /* Warna saat kotak menyala (kuning) */
        }

        .cell.selected {
            background-color: #3498db; /* Warna saat diklik pemain (biru) */
        }

        .cell.correct {
            background-color: #27ae60; /* Warna saat benar (hijau) */
        }

        .cell.incorrect {
            background-color: #e74c3c; /* Warna saat salah (merah) */
        }

        .message {
            min-height: 1.5em; /* Memberikan tinggi minimum agar tidak "meloncat" saat teks kosong/berubah */
            font-size: 1.3em;
            font-weight: bold;
            color: #ecf0f1;
            margin-top: auto; /* Mendorong pesan ke bagian bawah container */
        }

        /* Penyesuaian ukuran cell berdasarkan jumlah kolom untuk responsiveness */
        .game-board[style*="grid-template-columns"] .cell {
            width: auto; /* Biarkan CSS Grid menentukan lebar */
            height: auto; /* Biarkan CSS Grid menentukan tinggi */
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
        // Mendapatkan elemen-elemen dari DOM
        const gameBoard = document.getElementById('gameBoard');
        const startButton = document.getElementById('startButton');
        const levelSpan = document.getElementById('level');
        const messageDiv = document.getElementById('message');

        // Variabel-variabel state game
        let currentLevel = 1;
        let gridSize = 3; // Ukuran awal grid (misal: 3x3)
        let cellsToLight = 2; // Jumlah kotak yang akan menyala di awal
        let litCells = []; // Menyimpan indeks kotak yang menyala (solusi)
        let playerClicks = []; // Menyimpan indeks kotak yang diklik pemain
        let isPlaying = false; // Status game (berjalan atau tidak)
        let highlightDuration = 1000; // Durasi kotak menyala dalam milidetik
        let gameTimeout; // Variabel untuk menyimpan ID setTimeout agar bisa dibatalkan

        /**
         * Menginisialisasi ulang semua parameter game ke kondisi awal.
         * Dipanggil saat game pertama kali dimuat atau saat restart setelah salah.
         */
        function initializeGame() {
            currentLevel = 1;
            gridSize = 3;
            cellsToLight = 2;
            highlightDuration = 1000; // Reset durasi highlight ke nilai awal
            updateInfo(); // Perbarui tampilan level
            clearBoard(); // Bersihkan semua highlight dari board

            messageDiv.textContent = 'Klik "Mulai Game" untuk memulai!'; // Tampilkan pesan awal
            startButton.textContent = 'Mulai Game'; // Pastikan teks tombol kembali ke 'Mulai Game'
            startButton.style.display = 'block'; // Tampilkan tombol mulai

            // Sembunyikan board secara langsung pada inisialisasi
            gameBoard.classList.remove('active'); // Hapus kelas active untuk menyembunyikan
            gameBoard.style.pointerEvents = 'none'; // Nonaktifkan interaksi klik pada board
            isPlaying = false; // Set status game tidak sedang bermain
        }

        /**
         * Memulai siklus game.
         * Dipanggil saat tombol "Mulai Game" diklik atau saat game restart otomatis.
         */
        function startGame() {
            if (isPlaying) return; // Jika game sudah berjalan, jangan mulai lagi
            isPlaying = true; // Set status game sedang bermain
            startButton.style.display = 'none'; // Sembunyikan tombol mulai
            messageDiv.textContent = ''; // Bersihkan pesan

            // Tampilkan board dan nonaktifkan klik sementara
            gameBoard.classList.add('active'); // Tampilkan board dengan efek fade-in
            gameBoard.style.pointerEvents = 'none'; // Nonaktifkan klik saat pola ditampilkan

            buildBoard(); // Bangun ulang grid (penting jika gridSize berubah)
            // Beri jeda sebentar sebelum menampilkan pola agar user siap
            gameTimeout = setTimeout(showPattern, 1000); 
        }

        /**
         * Membangun atau membangun ulang elemen grid di DOM.
         * Mengatur jumlah kolom CSS Grid berdasarkan gridSize.
         */
        function buildBoard() {
            gameBoard.innerHTML = ''; // Hapus semua sel sebelumnya
            gameBoard.style.gridTemplateColumns = `repeat(${gridSize}, 1fr)`; // Atur jumlah kolom CSS Grid
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
                delay += 150; // Jeda antar kotak menyala (untuk efek sekuensial)
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
                gameBoard.style.pointerEvents = 'none'; // Nonaktifkan klik setelah semua klik terdaftar
                // Setelah semua klik, beri jeda sebentar lalu cek jawaban
                gameTimeout = setTimeout(checkAnswer, 500);
            }
        }

        /**
         * Memeriksa apakah jawaban pemain benar atau salah.
         * Memberikan feedback visual dan menentukan apakah game berlanjut atau restart.
         */
        function checkAnswer() {
            gameBoard.style.pointerEvents = 'none'; // Pastikan board tidak bisa diklik saat proses pengecekan

            // Sortir kedua array untuk perbandingan yang mudah dan memastikan urutan tidak penting
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
                messageDiv.textContent = `SALAH! Game Over.`;
                highlightIncorrectCells(); // Tampilkan yang benar (hijau) dan yang salah (merah)
                isPlaying = false; // Set status game berakhir
                
                // Sembunyikan board untuk transisi yang lebih jelas saat game over
                gameBoard.classList.remove('active'); 

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
            updateInfo(); // Perbarui tampilan level
            buildBoard(); // Bangun ulang board (ukuran grid mungkin berubah)
            gameBoard.style.pointerEvents = 'none'; // Nonaktifkan klik saat mempersiapkan pola
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
