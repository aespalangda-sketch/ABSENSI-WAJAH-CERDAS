(https://github.com/user-attachments/files/32646054/sistem_absensi_pengenalan_wajah_3_html.html)
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Absensi Pengenalan Wajah</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        // Konfigurasi Tailwind untuk mengaktifkan class-based dark mode
        tailwind.config = {
            darkMode: 'class',
        }
    </script>
    <script src="https://cdn.jsdelivr.net/npm/@vladmandic/face-api/dist/face-api.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .video-container { position: relative; width: 100%; max-width: 640px; margin: 0 auto; border-radius: 0.75rem; overflow: hidden; background: #000; box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1); }
        video, canvas { position: absolute; top: 0; left: 0; width: 100%; height: 100%; object-fit: cover; }
        .video-placeholder { padding-top: 75%; }
        
        .toast {
            visibility: hidden; min-width: 250px; background-color: #10B981; color: #fff;
            text-align: center; border-radius: 8px; padding: 16px; position: fixed;
            z-index: 50; left: 50%; bottom: 30px; transform: translateX(-50%);
            font-size: 17px; box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            opacity: 0; transition: opacity 0.3s, bottom 0.3s;
        }
        .toast.show { visibility: visible; opacity: 1; bottom: 50px; }
        .toast.error { background-color: #EF4444; }

        /* Sembunyikan elemen UI saat dicetak (Ctrl+P) */
        @media print {
            body { background: white !important; color: black !important; }
            #app-container > div > header, 
            .video-container, 
            #registerControls, 
            #attendanceControls, 
            .flex.border-b,
            .top-controls-container,
            #registeredListSection,
            .mb-4.flex.flex-col.gap-3 { display: none !important; }
            #attendanceListSection { display: block !important; width: 100% !important; }
            * { color: black !important; background-color: transparent !important; }
        }
    </style>
    <script>
        // Inisialisasi Mode Gelap agar tidak ada flash saat reload (Default ke Dark Mode)
        if (localStorage.theme === 'dark' || !('theme' in localStorage)) {
            document.documentElement.classList.add('dark');
        } else {
            document.documentElement.classList.remove('dark');
        }
    </script>
</head>
<body class="bg-gray-100 dark:bg-gray-900 text-gray-800 dark:text-gray-200 transition-colors duration-300">

    <div id="loginScreen" class="fixed inset-0 bg-gray-900 z-50 flex flex-col items-center justify-center text-white p-4">
        <div class="bg-white dark:bg-gray-800 p-8 rounded-2xl shadow-2xl w-full max-w-md border dark:border-gray-700 text-gray-800 dark:text-gray-200 text-center relative overflow-hidden">
            <div class="absolute top-0 left-0 w-full h-2 bg-gradient-to-r from-blue-500 to-purple-600"></div>
            <div class="w-16 h-16 bg-blue-100 dark:bg-blue-900/50 rounded-full flex items-center justify-center mx-auto mb-4 text-blue-600 dark:text-blue-400 text-2xl shadow-inner">
                <i class="fas fa-shield-alt"></i>
            </div>
            <h2 class="text-2xl font-extrabold mb-1 text-gray-800 dark:text-white">Akses Terbatas</h2>
            <p class="text-xs text-gray-500 dark:text-gray-400 mb-6">Masukkan kata sandi aplikasi untuk melanjutkan ke sistem absensi.</p>
            
            <div class="mb-4 text-left">
                <label class="block text-xs font-semibold text-gray-600 dark:text-gray-300 mb-1 uppercase tracking-wider">Kata Sandi Akses</label>
                <div class="relative">
                    <input type="password" id="loginPasswordInput" placeholder="Masukkan Password..." class="w-full px-4 py-3 bg-gray-50 dark:bg-gray-700 border border-gray-300 dark:border-gray-600 text-gray-800 dark:text-white rounded-xl focus:ring-2 focus:ring-blue-500 outline-none transition-colors pr-10 text-sm">
                    <button type="button" onclick="togglePasswordVisibility()" class="absolute right-3 top-1/2 -translate-y-1/2 text-gray-400 hover:text-gray-600 dark:hover:text-gray-200">
                        <i class="fas fa-eye" id="eyeIcon"></i>
                    </button>
                </div>
            </div>

            <button onclick="verifyLoginPassword()" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-4 rounded-xl shadow-lg flex items-center justify-center transition-colors mb-4">
                <i class="fas fa-sign-in-alt mr-2"></i> Masuk Aplikasi
            </button>

            <!-- Keterangan & Kontak Admin -->
            <div class="border-t border-gray-200 dark:border-gray-700 pt-4 mt-2">
                <p class="text-xs text-gray-500 dark:text-gray-400 mb-3">
                    Hubungi admin untuk mendapatkan password:
                </p>
                <div class="flex justify-center items-center gap-4 text-xl">
                    <!-- WhatsApp Admin -->
                    <a href="https://wa.me/6285397772345" target="_blank" class="w-10 h-10 rounded-full bg-green-100 dark:bg-green-900/40 text-green-600 dark:text-green-400 flex items-center justify-center hover:scale-110 transition-transform shadow-sm" title="WhatsApp: 085397772345">
                        <i class="fab fa-whatsapp"></i>
                    </a>
                    <!-- Instagram Admin -->
                    <a href="https://instagram.com/aes_435" target="_blank" class="w-10 h-10 rounded-full bg-pink-100 dark:bg-pink-900/40 text-pink-600 dark:text-pink-400 flex items-center justify-center hover:scale-110 transition-transform shadow-sm" title="Instagram: @aes_435">
                        <i class="fab fa-instagram"></i>
                    </a>
                    <!-- TikTok Admin -->
                    <a href="https://tiktok.com/@aes_435" target="_blank" class="w-10 h-10 rounded-full bg-gray-100 dark:bg-gray-700 text-gray-800 dark:text-gray-200 flex items-center justify-center hover:scale-110 transition-transform shadow-sm" title="TikTok: @aes_435">
                        <i class="fab fa-tiktok"></i>
                    </a>
                </div>
                
                <div class="mt-4 text-[10px] text-gray-400 dark:text-gray-500 font-medium tracking-wide uppercase">
                    Aplikasi ini dibuat oleh AES
                </div>
            </div>
        </div>
    </div>

    <div id="app-container" class="hidden">
        <!-- Loading Overlay -->
        <div id="loadingOverlay" class="fixed inset-0 bg-gray-900 bg-opacity-90 z-50 flex flex-col items-center justify-center text-white">
            <i class="fas fa-spinner fa-spin text-5xl mb-4 text-blue-500"></i>
            <h2 class="text-2xl font-bold mb-2">Memuat Model AI...</h2>
            <p class="text-gray-300 text-center px-4">Sistem sedang memuat model pengenalan wajah.<br>Mohon tunggu sebentar.</p>
        </div>

        <div class="max-w-6xl mx-auto px-4 py-8">
            <div class="top-controls-container flex justify-end mb-2 gap-2">
                <button onclick="toggleDarkMode()" id="darkModeBtn" class="bg-gray-200 dark:bg-gray-800 hover:bg-gray-300 dark:hover:bg-gray-700 text-gray-700 dark:text-gray-300 border border-transparent dark:border-gray-700 px-3 py-2 rounded-lg text-sm font-semibold transition-colors flex items-center shadow-sm">
                    <i class="fas fa-moon mr-2" id="darkModeIcon"></i> <span id="darkModeText">Dark Mode</span>
                </button>
                <button onclick="toggleFullScreen()" id="fullscreenBtn" class="bg-gray-200 dark:bg-gray-800 hover:bg-gray-300 dark:hover:bg-gray-700 text-gray-700 dark:text-gray-300 border border-transparent dark:border-gray-700 px-3 py-2 rounded-lg text-sm font-semibold transition-colors flex items-center shadow-sm">
                    <i class="fas fa-expand mr-2"></i> Fullscreen
                </button>
            </div>
            
            <header class="text-center mb-8">
                <h1 class="text-3xl font-extrabold text-blue-700 dark:text-blue-400 mb-2"><i class="fas fa-id-badge mr-2"></i>Absensi Wajah Cerdas</h1>
                <p class="text-gray-600 dark:text-gray-400">Sistem presensi otomatis dengan Export/Import Data Kelas — <span class="text-xs font-bold text-purple-600 dark:text-purple-400">Created by AES</span></p>
            </header>

            <div class="bg-white dark:bg-gray-800 border dark:border-gray-700 rounded-xl shadow-lg overflow-hidden transition-colors duration-300">
                <div class="flex border-b dark:border-gray-700">
                    <button id="tabRegister" class="flex-1 py-4 text-center font-semibold text-blue-600 dark:text-blue-400 border-b-2 border-blue-600 dark:border-blue-400 bg-blue-50 dark:bg-blue-900/30 transition-colors focus:outline-none" onclick="switchTab('register')">
                        <i class="fas fa-database mr-2"></i> Kelola Data Siswa
                    </button>
                    <button id="tabAttendance" class="flex-1 py-4 text-center font-semibold text-gray-500 dark:text-gray-400 border-b-2 border-transparent hover:bg-gray-50 dark:hover:bg-gray-700 transition-colors focus:outline-none" onclick="switchTab('attendance')">
                        <i class="fas fa-camera mr-2"></i> Mode Absensi
                    </button>
                    <button id="tabRecap" class="flex-1 py-4 text-center font-semibold text-gray-500 dark:text-gray-400 border-b-2 border-transparent hover:bg-gray-50 dark:hover:bg-gray-700 transition-colors focus:outline-none" onclick="switchTab('recap')">
                        <i class="fas fa-chart-bar mr-2"></i> Rekap Laporan
                    </button>
                </div>

                <div class="p-6 md:p-8 flex flex-col md:flex-row gap-8">
                    <div id="leftPanel" class="w-full md:w-1/2 flex flex-col items-center">
                        
                        <!-- Video Container with Manual Start Overlay -->
                        <div class="w-full mb-4">
                            <div class="video-container relative border dark:border-gray-700" id="videoContainer">
                                <div class="video-placeholder"></div>
                                <video id="video" autoplay muted playsinline></video>
                                <canvas id="canvas"></canvas>
                                
                                <div id="cameraOverlay" class="absolute inset-0 bg-gray-800 flex flex-col items-center justify-center z-10">
                                    <i class="fas fa-video-slash text-4xl text-gray-400 mb-4"></i>
                                    <p class="text-gray-300 text-sm mb-4 px-4 text-center">Atur kelas & JSON di panel kanan terlebih dahulu.<br>Klik tombol di bawah jika siap merekam wajah.</p>
                                    <button onclick="turnOnCamera()" class="bg-blue-600 hover:bg-blue-700 text-white px-6 py-2 rounded-lg shadow font-semibold transition-colors">
                                        <i class="fas fa-power-off mr-2"></i> Nyalakan Kamera
                                    </button>
                                </div>
                            </div>
                        </div>

                        <div id="registerControls" class="w-full max-w-sm flex flex-col gap-3">
                            <input type="text" id="studentName" placeholder="Nama Lengkap Siswa" class="w-full px-4 py-3 bg-white dark:bg-gray-700 border border-gray-300 dark:border-gray-600 text-gray-800 dark:text-white rounded-lg focus:ring-2 focus:ring-blue-500 outline-none transition-colors">
                            <input type="text" id="studentClass" placeholder="Kelas (Cth: X RPL 1)" class="w-full px-4 py-3 bg-white dark:bg-gray-700 border border-gray-300 dark:border-gray-600 text-gray-800 dark:text-white rounded-lg focus:ring-2 focus:ring-blue-500 outline-none transition-colors">
                            <button onclick="registerStudent()" id="btnRegister" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-4 rounded-lg shadow flex items-center justify-center transition-colors">
                                <i class="fas fa-camera-retro mr-2"></i> Rekam Wajah
                            </button>
                        </div>

                        <div id="attendanceControls" class="w-full max-w-sm hidden flex-col items-center gap-3">
                            <div class="bg-green-100 dark:bg-green-900/30 border border-green-400 dark:border-green-800 text-green-700 dark:text-green-400 px-4 py-3 rounded-lg w-full text-center font-medium animate-pulse">
                                <i class="fas fa-expand mr-2"></i> Deteksi Wajah Aktif...
                            </div>
                            <!-- Slider Sensitivitas -->
                            <div class="w-full bg-gray-50 dark:bg-gray-700/50 border border-gray-200 dark:border-gray-600 p-3 rounded-lg mt-2 transition-colors">
                                <label for="accuracySlider" class="block text-xs font-bold text-gray-600 dark:text-gray-300 mb-2 flex justify-between">
                                    <span><i class="fas fa-sliders-h mr-1"></i> Pengaturan Ketatnya Deteksi</span>
                                    <span id="accuracyLabel" class="text-blue-600 dark:text-blue-400">0.43</span>
                                </label>
                                <input type="range" id="accuracySlider" min="0.30" max="0.60" step="0.01" value="0.43" class="w-full h-2 bg-gray-300 dark:bg-gray-600 rounded-lg appearance-none cursor-pointer" onchange="updateAccuracy(this.value)">
                                <div class="flex justify-between text-[10px] text-gray-500 dark:text-gray-400 mt-1">
                                    <span>Lebih Ketat (Sulit tertukar)</span>
                                    <span>Lebih Longgar (Mudah Dikenali)</span>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div id="rightPanel" class="w-full md:w-1/2 flex flex-col">
                        
                        <!-- Panel: Siswa Terdaftar -->
                        <div id="registeredListSection" class="h-full flex flex-col">
                            <h3 class="text-xl font-bold mb-4 text-gray-700 dark:text-gray-200 flex items-center justify-between">
                                <span><i class="fas fa-users mr-2 text-blue-500 dark:text-blue-400"></i>Siswa Terdaftar</span>
                                <span id="registeredCount" class="bg-blue-100 dark:bg-blue-900/50 text-blue-800 dark:text-blue-300 text-sm font-medium px-2.5 py-0.5 rounded-full">0</span>
                            </h3>
                            
                            <!-- Backup/Restore Controls -->
                            <div class="mb-3 grid grid-cols-2 gap-2">
                                <button onclick="exportJSON()" class="bg-gray-800 dark:bg-gray-700 hover:bg-gray-900 dark:hover:bg-gray-600 text-white py-2 px-3 rounded-lg text-sm font-medium transition-colors flex justify-center items-center" title="Backup Siswa & Log Absensi">
                                    <i class="fas fa-file-export mr-2"></i> Backup JSON
                                </button>
                                <button onclick="document.getElementById('importJSON').click()" class="bg-yellow-500 dark:bg-yellow-600 hover:bg-yellow-600 dark:hover:bg-yellow-500 text-white py-2 px-3 rounded-lg text-sm font-medium transition-colors flex justify-center items-center" title="Restore Siswa & Log Absensi">
                                    <i class="fas fa-file-import mr-2"></i> Restore JSON
                                </button>
                                <input type="file" id="importJSON" accept=".json" class="hidden" onchange="importJSON(event)">
                            </div>
                            
                            <div class="mb-3">
                                <select id="filterClassReg" onchange="updateUIList()" class="w-full px-3 py-2 bg-white dark:bg-gray-700 border border-gray-300 dark:border-gray-600 text-gray-800 dark:text-white rounded-lg text-sm outline-none focus:ring-2 focus:ring-blue-500 transition-colors">
                                    <option value="ALL">Semua Kelas</option>
                                </select>
                            </div>
                            
                            <div class="flex-1 bg-gray-50 dark:bg-gray-900 border dark:border-gray-700 rounded-lg overflow-y-auto max-h-[400px] relative transition-colors">
                                <table class="w-full text-sm text-left text-gray-500 dark:text-gray-400">
                                    <thead class="text-xs text-gray-700 dark:text-gray-300 uppercase bg-gray-200 dark:bg-gray-700 sticky top-0 z-10 transition-colors">
                                        <tr>
                                            <th scope="col" class="px-4 py-3 w-16 text-center">No</th>
                                            <th scope="col" class="px-4 py-3">Nama Siswa</th>
                                            <th scope="col" class="px-4 py-3">Kelas</th>
                                            <th scope="col" class="px-4 py-3 text-center w-20">Aksi</th>
                                        </tr>
                                    </thead>
                                    <tbody id="registeredStudentsList">
                                        <!-- Data tabel siswa akan dimuat di sini via JavaScript -->
                                    </tbody>
                                </table>
                                <div id="emptyRegistered" class="text-center text-gray-400 dark:text-gray-500 py-8 hidden absolute w-full top-10">
                                    <i class="fas fa-inbox text-4xl mb-2"></i><p>Belum ada siswa.</p>
                                </div>
                            </div>
                        </div>

                        <!-- Panel: Log Absensi Hari Ini -->
                        <div id="attendanceListSection" class="h-full hidden flex-col">
                            <h3 class="text-xl font-bold mb-4 text-gray-700 dark:text-gray-200 flex items-center justify-between">
                                <span><i class="fas fa-clipboard-list mr-2 text-green-500 dark:text-green-400"></i>Log Kehadiran</span>
                            </h3>
                            
                            <div class="mb-3 grid grid-cols-2 gap-2">
                                <input type="date" id="inputDate" onchange="updateUIList()" class="w-full px-3 py-2 bg-white dark:bg-gray-700 border border-gray-300 dark:border-gray-600 text-gray-800 dark:text-white rounded-lg text-sm outline-none focus:ring-2 focus:ring-green-500 transition-colors">
                                <input type="text" id="inputMapel" onchange="updateUIList()" onkeyup="updateUIList()" placeholder="Mata Pelajaran (Opsional)" value="Umum" class="w-full px-3 py-2 bg-white dark:bg-gray-700 border border-gray-300 dark:border-gray-600 text-gray-800 dark:text-white rounded-lg text-sm outline-none focus:ring-2 focus:ring-green-500 transition-colors">
                            </div>
                            
                            <div class="mb-3 flex gap-2">
                                <select id="filterClassAtt" onchange="updateUIList()" class="w-full px-3 py-2 bg-white dark:bg-gray-700 border border-gray-300 dark:border-gray-600 text-gray-800 dark:text-white rounded-lg text-sm outline-none focus:ring-2 focus:ring-green-500 transition-colors">
                                    <option value="ALL">Semua Kelas</option>
                                </select>
                                <button onclick="downloadWord()" class="bg-blue-600 hover:bg-blue-700 text-white px-3 py-2 rounded-lg text-sm font-medium transition-colors flex items-center shadow-sm whitespace-nowrap" title="Unduh Daftar Hadir (Word)">
                                    <i class="fas fa-file-word"></i>
                                </button>
                                <button onclick="sendToWhatsApp()" class="bg-green-600 hover:bg-green-700 text-white px-3 py-2 rounded-lg text-sm font-medium transition-colors flex items-center shadow-sm whitespace-nowrap" title="Kirim Laporan ke WhatsApp">
                                    <i class="fab fa-whatsapp"></i>
                                </button>
                            </div>
                            
                            <div class="flex-1 bg-gray-50 dark:bg-gray-900 border dark:border-gray-700 rounded-lg overflow-y-auto max-h-[400px] relative transition-colors shadow-inner">
                                <table class="w-full text-sm text-left text-gray-500 dark:text-gray-400">
                                    <thead class="text-xs text-gray-700 dark:text-gray-300 uppercase bg-gray-200 dark:bg-gray-700 sticky top-0 z-10 transition-colors">
                                        <tr>
                                            <th scope="col" class="px-4 py-3 w-16 text-center">No</th>
                                            <th scope="col" class="px-4 py-3">Nama Siswa</th>
                                            <th scope="col" class="px-4 py-3">Kelas</th>
                                            <th scope="col" class="px-4 py-3 text-center w-24">Waktu</th>
                                        </tr>
                                    </thead>
                                    <tbody id="attendanceLogList">
                                        <!-- Log absensi akan dimuat di sini -->
                                    </tbody>
                                </table>
                                <div id="emptyAttendance" class="text-center text-gray-400 dark:text-gray-500 py-8 hidden absolute w-full top-10">
                                    <i class="fas fa-clipboard-check text-4xl mb-2"></i><p>Belum ada data kehadiran<br>untuk tanggal/mapel ini.</p>
                                </div>
                            </div>
                        </div>

                        <!-- Panel: Rekapitulasi Kehadiran -->
                        <div id="recapSection" class="h-full hidden flex-col w-full">
                            <h3 class="text-xl font-bold mb-4 text-gray-700 dark:text-gray-200 flex items-center">
                                <i class="fas fa-chart-bar mr-2 text-purple-500 dark:text-purple-400"></i> Rekapitulasi Kehadiran Mingguan/Bulanan
                            </h3>
                            
                            <div class="mb-4 flex flex-col md:flex-row gap-3 bg-purple-50 dark:bg-purple-900/20 p-4 rounded-lg border border-purple-100 dark:border-purple-800/30 transition-colors items-end">
                                <div class="flex-1 w-full">
                                    <label class="text-xs font-semibold text-gray-600 dark:text-gray-400 mb-1 block">Pilih Kelas</label>
                                    <select id="recapClassFilter" onchange="generateRecapTable()" class="w-full px-3 py-2 bg-white dark:bg-gray-700 border border-gray-300 dark:border-gray-600 text-gray-800 dark:text-white rounded outline-none focus:ring-2 focus:ring-purple-500"></select>
                                </div>
                                <div class="flex-1 w-full">
                                    <label class="text-xs font-semibold text-gray-600 dark:text-gray-400 mb-1 block">Pilih Mata Pelajaran</label>
                                    <select id="recapMapelFilter" onchange="generateRecapTable()" class="w-full px-3 py-2 bg-white dark:bg-gray-700 border border-gray-300 dark:border-gray-600 text-gray-800 dark:text-white rounded outline-none focus:ring-2 focus:ring-purple-500"></select>
                                </div>
                                <div class="w-full md:w-auto flex gap-2">
                                    <button onclick="downloadRecapWord()" class="w-full md:w-auto bg-purple-600 hover:bg-purple-700 text-white px-4 py-2 rounded font-bold transition-colors shadow flex items-center justify-center whitespace-nowrap">
                                        <i class="fas fa-file-word mr-2"></i> Unduh Rekap Word
                                    </button>
                                </div>
                            </div>
                            
                            <div class="flex-1 bg-white dark:bg-gray-900 border dark:border-gray-700 rounded-lg overflow-x-auto overflow-y-auto max-h-[450px] relative transition-colors shadow-inner">
                                <table class="w-full text-sm text-left text-gray-500 dark:text-gray-400 min-w-max">
                                    <thead id="recapTableHeader" class="text-xs text-gray-700 dark:text-gray-300 uppercase bg-gray-200 dark:bg-gray-700 sticky top-0 z-20 shadow-sm">
                                        <!-- Header Dinamis -->
                                    </thead>
                                    <tbody id="recapTableBody">
                                        <!-- Data Dinamis -->
                                    </tbody>
                                </table>
                                <div id="emptyRecap" class="text-center text-gray-400 dark:text-gray-500 py-12 hidden">
                                    <i class="fas fa-folder-open text-5xl mb-3"></i><p>Pilih kelas dan mata pelajaran untuk melihat rekap.</p>
                                </div>
                            </div>
                        </div>

                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Modal Edit Siswa -->
    <div id="editModal" class="fixed inset-0 bg-gray-900 bg-opacity-60 z-50 hidden flex-col items-center justify-center backdrop-blur-sm transition-opacity">
        <div class="bg-white dark:bg-gray-800 p-6 rounded-xl shadow-2xl w-full max-w-md border dark:border-gray-700">
            <h3 class="text-xl font-bold mb-4 text-gray-800 dark:text-gray-200"><i class="fas fa-user-edit text-blue-500 mr-2"></i>Edit Data Siswa</h3>
            <input type="hidden" id="editStudentId">
            <div class="mb-4">
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Nama Lengkap</label>
                <input type="text" id="editStudentName" class="w-full px-4 py-3 bg-white dark:bg-gray-700 border border-gray-300 dark:border-gray-600 text-gray-800 dark:text-white rounded-lg focus:ring-2 focus:ring-blue-500 outline-none transition-colors">
            </div>
            <div class="mb-6">
                <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Kelas</label>
                <input type="text" id="editStudentClass" class="w-full px-4 py-3 bg-white dark:bg-gray-700 border border-gray-300 dark:border-gray-600 text-gray-800 dark:text-white rounded-lg focus:ring-2 focus:ring-blue-500 outline-none transition-colors">
            </div>
            <div class="flex justify-end gap-3">
                <button onclick="closeEditModal()" class="px-4 py-2 bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-gray-200 rounded-lg hover:bg-gray-300 dark:hover:bg-gray-600 transition-colors font-medium">Batal</button>
                <button onclick="saveEditStudent()" class="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors font-medium shadow-md">Simpan Perubahan</button>
            </div>
        </div>
    </div>

    <!-- Modal Konfirmasi Hapus -->
    <div id="deleteModal" class="fixed inset-0 bg-gray-900 bg-opacity-60 z-50 hidden flex-col items-center justify-center backdrop-blur-sm transition-opacity">
        <div class="bg-white dark:bg-gray-800 p-6 rounded-xl shadow-2xl w-full max-w-sm border dark:border-gray-700 text-center">
            <div class="w-16 h-16 bg-red-100 dark:bg-red-900/30 rounded-full flex items-center justify-center mx-auto mb-4 text-red-500 text-3xl">
                <i class="fas fa-trash-alt"></i>
            </div>
            <h3 class="text-xl font-bold mb-2 text-gray-800 dark:text-gray-200">Hapus Data Siswa?</h3>
            <p class="text-gray-600 dark:text-gray-400 mb-6 text-sm">Profil wajah siswa akan dihapus dari sistem. Tindakan ini tidak dapat dibatalkan.</p>
            <input type="hidden" id="deleteStudentId">
            <div class="flex justify-center gap-3">
                <button onclick="closeDeleteModal()" class="px-4 py-2 bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-gray-200 rounded-lg hover:bg-gray-300 dark:hover:bg-gray-600 transition-colors font-medium w-full">Batal</button>
                <button onclick="confirmDeleteStudent()" class="px-4 py-2 bg-red-600 text-white rounded-lg hover:bg-red-700 transition-colors font-medium shadow-md w-full">Ya, Hapus</button>
            </div>
        </div>
    </div>

    <!-- Toast Notification -->
    <div id="toast" class="toast"><i id="toastIcon" class="fas fa-check-circle mr-2"></i><span id="toastMsg">Pesan</span></div>

    <script>
        const video = document.getElementById('video');
        const canvas = document.getElementById('canvas');
        const loadingOverlay = document.getElementById('loadingOverlay');
        const toast = document.getElementById('toast');
        const toastMsg = document.getElementById('toastMsg');
        const toastIcon = document.getElementById('toastIcon');

        let currentMode = 'register'; 
        let registeredStudents = [];
        let attendanceLog = {}; 
        let faceMatcher = null;
        let detectionInterval = null;
        let synthVoices = [];
        
        const MODEL_URL = 'https://cdn.jsdelivr.net/npm/@vladmandic/face-api/model/';
        let MATCH_THRESHOLD = 0.43; // Variabel sekarang dinamis

        function verifyLoginPassword() {
            const passInput = document.getElementById('loginPasswordInput').value.trim();
            if (passInput === "AES123AB") {
                document.getElementById('loginScreen').classList.add('hidden');
                document.getElementById('app-container').classList.remove('hidden');
                showToast("Akses Diberikan. Selamat Datang!");
            } else {
                showToast("Password salah! Silakan hubungi admin.", true);
            }
        }

        function togglePasswordVisibility() {
            const input = document.getElementById('loginPasswordInput');
            const icon = document.getElementById('eyeIcon');
            if (input.type === 'password') {
                input.type = 'text';
                icon.className = 'fas fa-eye-slash';
            } else {
                input.type = 'password';
                icon.className = 'fas fa-eye';
            }
        }

        // Support Enter key for login
        document.addEventListener('DOMContentLoaded', () => {
            const passInput = document.getElementById('loginPasswordInput');
            if (passInput) {
                passInput.addEventListener('keypress', (e) => {
                    if (e.key === 'Enter') verifyLoginPassword();
                });
            }
        });

        async function init() {
            loadDataFromStorage();
            updateDarkModeIcon(document.documentElement.classList.contains('dark'));
            
            // Set today's date
            const today = new Date();
            const offset = today.getTimezoneOffset() * 60000;
            const localISOTime = (new Date(today - offset)).toISOString().split('T')[0];
            document.getElementById('inputDate').value = localISOTime;

            updateUIList();
            initSpeechSynthesis();
            
            try {
                await Promise.all([
                    faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URL),
                    faceapi.nets.faceLandmark68Net.loadFromUri(MODEL_URL),
                    faceapi.nets.faceRecognitionNet.loadFromUri(MODEL_URL)
                ]);
                
                buildFaceMatcher();
                loadingOverlay.style.display = 'none';
                
                video.addEventListener('loadedmetadata', () => {
                    const displaySize = { width: video.videoWidth, height: video.videoHeight };
                    faceapi.matchDimensions(canvas, displaySize);
                    if(currentMode === 'attendance') startDetectionLoop();
                });
            } catch (err) {
                console.error(err);
                loadingOverlay.innerHTML = `
                    <i class="fas fa-exclamation-triangle text-5xl mb-4 text-red-500"></i>
                    <h2 class="text-2xl font-bold mb-2">Gagal Memuat</h2>
                    <p class="text-gray-300 text-center px-4">Pastikan perangkat memiliki kamera dan izin diberikan.</p>
                `;
            }
        }

        function toggleDarkMode() {
            const html = document.documentElement;
            const isDark = html.classList.toggle('dark');
            localStorage.setItem('theme', isDark ? 'dark' : 'light');
            updateDarkModeIcon(isDark);
        }

        function updateDarkModeIcon(isDark) {
            const icon = document.getElementById('darkModeIcon');
            const text = document.getElementById('darkModeText');
            if(icon && text) {
                if (isDark) {
                    icon.className = 'fas fa-sun text-yellow-400 mr-2';
                    text.textContent = 'Light Mode';
                } else {
                    icon.className = 'fas fa-moon mr-2';
                    text.textContent = 'Dark Mode';
                }
            }
        }

        async function startVideo() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" }, audio: false });
                video.srcObject = stream;
            } catch (err) {
                showToast("Gagal mengakses kamera!", true);
                throw err;
            }
        }

        async function turnOnCamera() {
            const btn = document.querySelector('#cameraOverlay button');
            const icon = document.querySelector('#cameraOverlay i');
            btn.innerHTML = '<i class="fas fa-spinner fa-spin mr-2"></i> Menghubungkan...';
            
            try {
                await startVideo();
                document.getElementById('cameraOverlay').style.display = 'none';
            } catch (err) {
                btn.innerHTML = '<i class="fas fa-power-off mr-2"></i> Coba Lagi';
                icon.className = 'fas fa-exclamation-triangle text-4xl text-red-500 mb-4';
            }
        }

        function switchTab(mode) {
            currentMode = mode;
            const tabRegister = document.getElementById('tabRegister');
            const tabAttendance = document.getElementById('tabAttendance');
            const tabRecap = document.getElementById('tabRecap');
            
            const regControls = document.getElementById('registerControls');
            const attControls = document.getElementById('attendanceControls');
            const regList = document.getElementById('registeredListSection');
            const attList = document.getElementById('attendanceListSection');
            const recapList = document.getElementById('recapSection');
            
            const leftPanel = document.getElementById('leftPanel');
            const rightPanel = document.getElementById('rightPanel');

            const activeClasses = ['text-blue-600', 'border-blue-600', 'bg-blue-50', 'dark:text-blue-400', 'dark:border-blue-400', 'dark:bg-blue-900/30'];
            const activeRecapClasses = ['text-purple-600', 'border-purple-600', 'bg-purple-50', 'dark:text-purple-400', 'dark:border-purple-400', 'dark:bg-purple-900/30'];
            const inactiveClasses = ['text-gray-500', 'border-transparent', 'hover:bg-gray-50', 'dark:text-gray-400', 'dark:hover:bg-gray-700'];

            // Reset tab styles
            [tabRegister, tabAttendance, tabRecap].forEach(tab => {
                tab.className = `flex-1 py-4 text-center font-semibold text-gray-500 dark:text-gray-400 border-b-2 border-transparent hover:bg-gray-50 dark:hover:bg-gray-700 transition-colors focus:outline-none`;
            });

            if (mode === 'register') {
                tabRegister.classList.remove(...inactiveClasses);
                tabRegister.classList.add(...activeClasses);
                
                leftPanel.style.display = 'flex';
                leftPanel.className = "w-full md:w-1/2 flex flex-col items-center";
                rightPanel.className = "w-full md:w-1/2 flex flex-col";

                regControls.classList.replace('hidden', 'flex');
                attControls.classList.replace('flex', 'hidden');
                regList.classList.replace('hidden', 'flex');
                attList.classList.replace('flex', 'hidden');
                recapList.classList.replace('flex', 'hidden');
                
                stopDetectionLoop();
                clearCanvas();
            } else if (mode === 'attendance') {
                tabAttendance.classList.remove(...inactiveClasses);
                tabAttendance.classList.add(...activeClasses);
                
                leftPanel.style.display = 'flex';
                leftPanel.className = "w-full md:w-1/2 flex flex-col items-center";
                rightPanel.className = "w-full md:w-1/2 flex flex-col";

                attControls.classList.replace('hidden', 'flex');
                regControls.classList.replace('flex', 'hidden');
                attList.classList.replace('hidden', 'flex');
                regList.classList.replace('flex', 'hidden');
                recapList.classList.replace('flex', 'hidden');

                if (registeredStudents.length === 0) {
                    showToast("Sistem kosong. Silakan Restore Data JSON atau Daftar Baru.", true);
                    switchTab('register');
                    return;
                }
                
                buildFaceMatcher();
                startDetectionLoop();
            } else if (mode === 'recap') {
                tabRecap.classList.remove(...inactiveClasses);
                tabRecap.classList.add(...activeRecapClasses);
                
                leftPanel.style.display = 'none';
                rightPanel.className = "w-full flex flex-col";

                recapList.classList.replace('hidden', 'flex');
                regList.classList.replace('flex', 'hidden');
                attList.classList.replace('flex', 'hidden');
                regControls.classList.replace('flex', 'hidden');
                attControls.classList.replace('flex', 'hidden');
                
                stopDetectionLoop();
                clearCanvas();
                populateRecapFilters();
            }
        }

        function loadDataFromStorage() {
            const storedStudents = localStorage.getItem('faceAttendance_students');
            if (storedStudents) {
                registeredStudents = JSON.parse(storedStudents);
            } else {
                // Bersih total secara default agar tidak ada data siswa bawaan di GitHub
                registeredStudents = [];
            }

            const storedLogs = localStorage.getItem('faceAttendance_logs');
            if (storedLogs) {
                let parsedLogs = JSON.parse(storedLogs);
                let migrated = false;
                for (let key in parsedLogs) {
                    if (key.length === 10 && key.indexOf('-') === 4) {
                        parsedLogs[key + '_Umum'] = parsedLogs[key];
                        delete parsedLogs[key];
                        migrated = true;
                    }
                }
                attendanceLog = parsedLogs;
                if (migrated) saveDataToStorage();
            } else {
                // Bersih total riwayat absensi secara default
                attendanceLog = {};
            }
        }

        function saveDataToStorage() {
            localStorage.setItem('faceAttendance_students', JSON.stringify(registeredStudents));
            localStorage.setItem('faceAttendance_logs', JSON.stringify(attendanceLog));
        }

        function exportJSON() {
            if (registeredStudents.length === 0 && Object.keys(attendanceLog).length === 0) return showToast("Tidak ada data untuk diekspor!", true);
            
            // Format v2: Simpan siswa sekaligus data rekap absen
            const exportData = {
                version: "2.0",
                students: registeredStudents,
                logs: attendanceLog
            };
            
            const dataStr = JSON.stringify(exportData, null, 2);
            const blob = new Blob([dataStr], { type: "application/json" });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            
            const dateStr = new Date().toISOString().split('T')[0];
            const mapel = document.getElementById('inputMapel').value.trim() || 'Umum';
            a.download = `Backup_Data_${mapel}_${dateStr}.json`;
            
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
            showToast("Data lengkap (Siswa & Absen) berhasil dibackup!");
        }

        function importJSON(event) {
            const file = event.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const importedData = JSON.parse(e.target.result);
                    if (Array.isArray(importedData)) {
                        // Versi lama (hanya array siswa)
                        registeredStudents = importedData;
                        showToast("Data siswa berhasil di-restore (Versi Lama)!");
                    } else if (importedData.version === "2.0") {
                        // Versi baru (siswa + log absensi)
                        registeredStudents = importedData.students || [];
                        attendanceLog = importedData.logs || {};
                        showToast("Data lengkap (Siswa & Rekap) berhasil di-restore!");
                    } else {
                        showToast("Format file JSON tidak sesuai!", true);
                        return;
                    }
                    saveDataToStorage();
                    buildFaceMatcher();
                    updateUIList();
                    if(currentMode === 'recap') populateRecapFilters();
                } catch (err) {
                    showToast("Gagal membaca file JSON!", true);
                }
            };
            reader.readAsText(file);
            event.target.value = ''; 
        }

        function initSpeechSynthesis() {
            if ('speechSynthesis' in window) {
                const updateVoices = () => { synthVoices = window.speechSynthesis.getVoices(); };
                updateVoices();
                if (speechSynthesis.onvoiceschanged !== undefined) speechSynthesis.onvoiceschanged = updateVoices;
            }
        }

        function speakName(name) {
            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel(); 
                const utterance = new SpeechSynthesisUtterance(`${name}, hadir.`);
                utterance.lang = 'id-ID';
                utterance.pitch = 1.3;
                utterance.rate = 0.95; 
                const idVoice = synthVoices.find(v => v.lang.includes('id-ID') || v.lang.includes('id_ID'));
                if (idVoice) utterance.voice = idVoice;
                window.speechSynthesis.speak(utterance);
            }
        }

        async function registerStudent() {
            const nameInput = document.getElementById('studentName');
            const classInput = document.getElementById('studentClass');
            const name = nameInput.value.trim();
            const studentClass = classInput.value.trim();
            const btn = document.getElementById('btnRegister');

            if (!name || !studentClass) {
                showToast("Nama dan Kelas tidak boleh kosong!", true);
                return;
            }

            if(document.getElementById('cameraOverlay').style.display !== 'none') {
                showToast("Nyalakan kamera terlebih dahulu!", true);
                return;
            }

            btn.disabled = true;
            btn.innerHTML = '<i class="fas fa-spinner fa-spin mr-2"></i> Memindai...';

            try {
                const detection = await faceapi.detectSingleFace(video, new faceapi.TinyFaceDetectorOptions()).withFaceLandmarks().withFaceDescriptor();
                if (!detection) {
                    showToast("Wajah tidak terdeteksi! Pastikan terang dan jelas.", true);
                    btn.disabled = false;
                    btn.innerHTML = '<i class="fas fa-camera-retro mr-2"></i> Rekam Wajah';
                    return;
                }

                registeredStudents.push({
                    id: Date.now().toString(),
                    name: name,
                    className: studentClass,
                    descriptor: Array.from(detection.descriptor),
                    dateRegistered: new Date().toLocaleDateString('id-ID')
                });

                saveDataToStorage();
                buildFaceMatcher();
                updateUIList();
                nameInput.value = '';
                showToast(`Berhasil mendaftarkan: ${name}`);
            } catch (err) {
                showToast("Terjadi kesalahan saat memindai wajah.", true);
            }

            btn.disabled = false;
            btn.innerHTML = '<i class="fas fa-camera-retro mr-2"></i> Rekam Wajah';
        }

        function buildFaceMatcher() {
            if (registeredStudents.length === 0) { faceMatcher = null; return; }
            const labeledDescriptors = registeredStudents.map(student => {
                return new faceapi.LabeledFaceDescriptors(student.id, [new Float32Array(student.descriptor)]);
            });
            faceMatcher = new faceapi.FaceMatcher(labeledDescriptors, MATCH_THRESHOLD);
        }

        function startDetectionLoop() {
            if (detectionInterval) clearInterval(detectionInterval);
            if(document.getElementById('cameraOverlay').style.display !== 'none') return; // Do not loop if camera is off

            const displaySize = { width: video.videoWidth, height: video.videoHeight };
            if(displaySize.width === 0) return;
            faceapi.matchDimensions(canvas, displaySize);

            detectionInterval = setInterval(async () => {
                if (currentMode !== 'attendance' || !faceMatcher) return;

                const selectedDateStr = document.getElementById('inputDate').value;
                const mapelStr = document.getElementById('inputMapel').value.trim() || 'Umum';
                if(!selectedDateStr) return;
                const logKey = `${selectedDateStr}_${mapelStr}`;

                const detections = await faceapi.detectAllFaces(video, new faceapi.TinyFaceDetectorOptions()).withFaceLandmarks().withFaceDescriptors();
                const resizedDetections = faceapi.resizeResults(detections, displaySize);
                
                clearCanvas();
                faceapi.draw.drawDetections(canvas, resizedDetections);

                if (!attendanceLog[logKey]) attendanceLog[logKey] = [];

                resizedDetections.forEach(detection => {
                    const match = faceMatcher.findBestMatch(detection.descriptor);
                    let student = null;
                    let displayLabel = "Tidak Dikenal";
                    
                    if (match.label !== 'unknown' && match.distance < MATCH_THRESHOLD) {
                        student = registeredStudents.find(s => s.id === match.label);
                        if (student) displayLabel = `${student.name} (${student.className || '-'})`;
                    }
                    
                    const box = detection.detection.box;
                    new faceapi.draw.DrawBox(box, { label: displayLabel }).draw(canvas);

                    if (student) {
                        const alreadyAttended = attendanceLog[logKey].find(log => log.id === student.id);
                        if (!alreadyAttended) {
                            attendanceLog[logKey].unshift({
                                id: student.id,
                                name: student.name,
                                className: student.className || 'Tanpa Kelas',
                                time: new Date().toLocaleTimeString('id-ID')
                            });
                            
                            saveDataToStorage();
                            updateUIList();
                            showToast(`Hadir: ${student.name}`, false);
                            speakName(student.name);
                            new faceapi.draw.DrawBox(box, { label: "Tercatat!", boxColor: '#10B981' }).draw(canvas);
                        }
                    }
                });
            }, 500); 
        }

        function stopDetectionLoop() {
            if (detectionInterval) { clearInterval(detectionInterval); detectionInterval = null; }
        }

        function updateAccuracy(val) {
            MATCH_THRESHOLD = parseFloat(val);
            document.getElementById('accuracyLabel').textContent = val;
            buildFaceMatcher(); 
            showToast(`Toleransi deteksi diubah ke ${val}`);
        }

        function clearCanvas() {
            canvas.getContext('2d').clearRect(0, 0, canvas.width, canvas.height);
        }

        function updateUIList() {
            const allClasses = [...new Set(registeredStudents.map(s => s.className || 'Tanpa Kelas'))].sort();
            const filterReg = document.getElementById('filterClassReg');
            const filterAtt = document.getElementById('filterClassAtt');
            
            if (filterReg && filterAtt) {
                const currReg = filterReg.value || 'ALL';
                const currAtt = filterAtt.value || 'ALL';
                let options = `<option value="ALL">Semua Kelas</option>`;
                allClasses.forEach(c => { options += `<option value="${c}">${c}</option>`; });
                filterReg.innerHTML = options; filterAtt.innerHTML = options;
                filterReg.value = allClasses.includes(currReg) ? currReg : 'ALL';
                filterAtt.value = allClasses.includes(currAtt) ? currAtt : 'ALL';
            }

            const activeReg = filterReg ? filterReg.value : 'ALL';
            let filteredStudents = activeReg !== 'ALL' ? registeredStudents.filter(s => (s.className || '') === activeReg) : registeredStudents;
            document.getElementById('registeredCount').textContent = filteredStudents.length;
            const regList = document.getElementById('registeredStudentsList');
            regList.innerHTML = '';
            document.getElementById('emptyRegistered').classList.toggle('hidden', filteredStudents.length > 0);
            
            [...filteredStudents].reverse().forEach((student, index) => {
                const tr = document.createElement('tr');
                tr.className = "bg-white dark:bg-gray-800 border-b dark:border-gray-700 hover:bg-gray-50 dark:hover:bg-gray-700 transition-colors";
                tr.innerHTML = `
                    <td class="px-4 py-3 text-center font-medium text-gray-900 dark:text-gray-200">${index + 1}</td>
                    <td class="px-4 py-3 font-semibold text-gray-800 dark:text-gray-100 flex items-center gap-2">
                        <div class="w-8 h-8 rounded-full bg-blue-100 dark:bg-blue-900/50 flex items-center justify-center text-blue-600 dark:text-blue-400 font-bold shrink-0">${student.name.charAt(0).toUpperCase()}</div>
                        ${student.name}
                    </td>
                    <td class="px-4 py-3">
                        <span class="text-xs bg-blue-100 dark:bg-blue-900/50 text-blue-800 dark:text-blue-300 px-2 py-1 rounded font-medium whitespace-nowrap">${student.className || '-'}</span>
                    </td>
                    <td class="px-4 py-3 text-center flex justify-center gap-2">
                        <button onclick="openEditModal('${student.id}')" class="text-blue-500 dark:text-blue-400 hover:text-blue-700 dark:hover:text-blue-300 p-2 transition-colors rounded hover:bg-blue-50 dark:hover:bg-blue-900/20" title="Edit Siswa"><i class="fas fa-edit"></i></button>
                        <button onclick="openDeleteModal('${student.id}')" class="text-red-500 dark:text-red-400 hover:text-red-700 dark:hover:text-red-300 p-2 transition-colors rounded hover:bg-red-50 dark:hover:bg-red-900/20" title="Hapus Siswa"><i class="fas fa-trash"></i></button>
                    </td>`;
                regList.appendChild(tr);
            });

            const activeAtt = filterAtt ? filterAtt.value : 'ALL';
            const selectedDateStr = document.getElementById('inputDate').value;
            const mapelStr = document.getElementById('inputMapel').value.trim() || 'Umum';
            const logKey = `${selectedDateStr}_${mapelStr}`;
            let logsToDisplay = attendanceLog[logKey] || [];
            
            if (activeAtt !== 'ALL') logsToDisplay = logsToDisplay.filter(log => (log.className || '') === activeAtt);
            
            const attList = document.getElementById('attendanceLogList');
            attList.innerHTML = '';
            document.getElementById('emptyAttendance').classList.toggle('hidden', logsToDisplay.length > 0);
            
            logsToDisplay.forEach((log, index) => {
                const tr = document.createElement('tr');
                tr.className = "bg-green-50 dark:bg-green-900/20 border-b border-green-100 dark:border-green-800 hover:bg-green-100 dark:hover:bg-green-900/40 transition-colors";
                tr.innerHTML = `
                    <td class="px-4 py-3 text-center font-medium text-gray-900 dark:text-gray-200">${index + 1}</td>
                    <td class="px-4 py-3 font-semibold text-gray-800 dark:text-gray-100 flex items-center gap-2">
                        <i class="fas fa-check-circle text-green-500 dark:text-green-400 text-lg shrink-0"></i>
                        ${log.name}
                    </td>
                    <td class="px-4 py-3 text-gray-600 dark:text-gray-300 font-medium whitespace-nowrap">${log.className}</td>
                    <td class="px-4 py-3 text-center">
                        <span class="text-xs font-bold text-gray-600 dark:text-gray-300 bg-white dark:bg-gray-700 px-2 py-1 rounded border border-gray-200 dark:border-gray-600 shadow-sm whitespace-nowrap">${log.time}</span>
                    </td>`;
                attList.appendChild(tr);
            });
        }

        function downloadWord() {
            if (registeredStudents.length === 0) return showToast("Sistem kosong. Tambah data siswa terlebih dahulu.", true);

            const selectedDate = document.getElementById('inputDate').value; 
            const mapel = document.getElementById('inputMapel').value.trim() || 'Umum';
            const filterAtt = document.getElementById('filterClassAtt').value;
            
            if(!selectedDate) return showToast("Pilih tanggal absen terlebih dahulu!", true);
            const logKey = `${selectedDate}_${mapel}`;

            let studentsToExport = registeredStudents;
            let classLabel = "Semua Kelas";
            if (filterAtt !== 'ALL') {
                studentsToExport = registeredStudents.filter(s => (s.className || '') === filterAtt);
                classLabel = filterAtt;
            }

            if (studentsToExport.length === 0) return showToast(`Tidak ada data siswa di kelas ${classLabel}.`, true);

            studentsToExport.sort((a, b) => a.name.localeCompare(b.name));
            const logsForDate = attendanceLog[logKey] || [];

            const dateObj = new Date(selectedDate);
            const formattedDateStr = dateObj.toLocaleDateString('id-ID', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });

            let htmlContent = `
            <html xmlns:o='urn:schemas-microsoft-com:office:office' xmlns:w='urn:schemas-microsoft-com:office:word' xmlns='http://www.w3.org/TR/REC-html40'>
            <head>
                <meta charset='utf-8'>
                <title>Laporan Kehadiran</title>
                <style>
                    body { font-family: 'Times New Roman', serif; color: #000; }
                    .header-title { text-align: center; font-size: 16pt; font-weight: bold; margin-bottom: 20px; text-transform: uppercase; }
                    .info-text { font-size: 12pt; margin-bottom: 5px; }
                    table { border-collapse: collapse; width: 100%; margin-top: 15px; }
                    th, td { border: 1px solid black; padding: 8px; font-size: 12pt; }
                    th { background-color: #e0e0e0; font-weight: bold; text-align: center; }
                    .text-center { text-align: center; }
                    .check-mark { font-family: Arial, sans-serif; font-weight: bold; font-size: 14pt; color: #000; }
                </style>
            </head>
            <body>
                <div class="header-title">Daftar Hadir Siswa</div>
                <div class="info-text"><strong>Mata Pelajaran :</strong> ${mapel}</div>
                <div class="info-text"><strong>Tanggal        :</strong> ${formattedDateStr}</div>
                <div class="info-text"><strong>Kelas          :</strong> ${classLabel}</div>
                
                <table>
                    <thead>
                        <tr>
                            <th width="10%">No</th>
                            <th width="65%">Nama Lengkap</th>
                            <th width="25%">Keterangan</th>
                        </tr>
                    </thead>
                    <tbody>
            `;

            studentsToExport.forEach((student, index) => {
                const isPresent = logsForDate.some(log => log.id === student.id);
                const status = isPresent ? "<span class='check-mark'>&#10003;</span>" : "&nbsp;"; 
                
                htmlContent += `
                        <tr>
                            <td class="text-center">${index + 1}</td>
                            <td>${student.name}</td>
                            <td class="text-center">${status}</td>
                        </tr>
                `;
            });

            htmlContent += `
                    </tbody>
                </table>
                <br><br>
                <table style="border: none; width: 100%;">
                    <tr>
                        <td style="border: none; width: 60%;"></td>
                        <td style="border: none; width: 40%; text-align: center;">
                            Mengetahui,<br>Guru Mata Pelajaran
                            <br><br><br><br><br>
                            _________________________
                        </td>
                    </tr>
                </table>
            </body>
            </html>
            `;

            const blob = new Blob(['\ufeff', htmlContent], { type: 'application/msword' });

            let formattedDate = selectedDate;
            const dateParts = selectedDate.split('-');
            if(dateParts.length === 3) formattedDate = `${dateParts[2]}_${dateParts[1]}_${dateParts[0]}`; 
            
            const safeMapel = mapel.replace(/[^a-zA-Z0-9]/g, '_');
            const fileName = `${formattedDate}_${safeMapel}.doc`;

            const url = URL.createObjectURL(blob);
            const a = document.createElement("a");
            a.href = url;
            a.download = fileName;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        }

        function sendToWhatsApp() {
            if (registeredStudents.length === 0) return showToast("Sistem kosong. Tambah data siswa terlebih dahulu.", true);

            const selectedDate = document.getElementById('inputDate').value; 
            const mapel = document.getElementById('inputMapel').value.trim() || 'Umum';
            const filterAtt = document.getElementById('filterClassAtt').value;
            
            if(!selectedDate) return showToast("Pilih tanggal absen terlebih dahulu!", true);
            const logKey = `${selectedDate}_${mapel}`;

            let studentsToExport = registeredStudents;
            let classLabel = "Semua Kelas";
            if (filterAtt !== 'ALL') {
                studentsToExport = registeredStudents.filter(s => (s.className || '') === filterAtt);
                classLabel = filterAtt;
            }

            if (studentsToExport.length === 0) return showToast(`Tidak ada data siswa di kelas ${classLabel}.`, true);

            studentsToExport.sort((a, b) => a.name.localeCompare(b.name));
            const logsForDate = attendanceLog[logKey] || [];
            const dateObj = new Date(selectedDate);
            const formattedDateStr = dateObj.toLocaleDateString('id-ID', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });

            let message = `📊 *LAPORAN KEHADIRAN SISWA* 📊\n\n`;
            message += `📅 *Tanggal:* ${formattedDateStr}\n`;
            message += `📚 *Mapel:* ${mapel}\n`;
            message += `🏫 *Kelas:* ${classLabel}\n\n`;

            let presentList = [];
            let absentList = [];

            studentsToExport.forEach(student => {
                const isPresent = logsForDate.some(log => log.id === student.id);
                if (isPresent) {
                    const logDetail = logsForDate.find(log => log.id === student.id);
                    presentList.push(`${student.name} - *${logDetail.time}*`);
                } else {
                    absentList.push(`${student.name}`);
                }
            });

            message += `✅ *HADIR (${presentList.length} Siswa):*\n`;
            if (presentList.length > 0) {
                presentList.forEach((text, i) => {
                    message += `${i + 1}. ${text}\n`;
                });
                message += `\n`;
            } else {
                message += `- Tidak ada\n\n`;
            }

            message += `❌ *BELUM HADIR (${absentList.length} Siswa):*\n`;
            if (absentList.length > 0) {
                absentList.forEach((text, i) => {
                    message += `${i + 1}. ${text}\n`;
                });
                message += `\n`;
            } else {
                message += `- Tidak ada\n\n`;
            }
            
            message += `📝 _Dibuat otomatis oleh Sistem Absensi_`;

            const encodedMessage = encodeURIComponent(message);
            window.open(`https://wa.me/?text=${encodedMessage}`, '_blank');
        }

        /* Fitur Rekapitulasi Matriks */
        function populateRecapFilters() {
            const classFilter = document.getElementById('recapClassFilter');
            const mapelFilter = document.getElementById('recapMapelFilter');
            
            const allClasses = [...new Set(registeredStudents.map(s => s.className || 'Tanpa Kelas'))].sort();
            let classOptions = '';
            if(allClasses.length === 0) {
                classOptions = '<option value="">Belum ada kelas</option>';
            } else {
                allClasses.forEach(c => { classOptions += `<option value="${c}">${c}</option>`; });
            }
            classFilter.innerHTML = classOptions;

            const allMapels = new Set();
            Object.keys(attendanceLog).forEach(key => {
                if(key.length > 10) {
                    const mapel = key.substring(11);
                    if(mapel) allMapels.add(mapel);
                }
            });
            
            const sortedMapels = [...allMapels].sort();
            let mapelOptions = '';
            if(sortedMapels.length === 0) {
                mapelOptions = '<option value="">Belum ada data absensi</option>';
            } else {
                sortedMapels.forEach(m => { mapelOptions += `<option value="${m}">${m}</option>`; });
            }
            mapelFilter.innerHTML = mapelOptions;
            
            generateRecapTable();
        }

        function generateRecapTable() {
            const selectedClass = document.getElementById('recapClassFilter').value;
            const selectedMapel = document.getElementById('recapMapelFilter').value;
            
            const thead = document.getElementById('recapTableHeader');
            const tbody = document.getElementById('recapTableBody');
            const emptyMsg = document.getElementById('emptyRecap');
            
            if(!selectedClass || !selectedMapel) {
                thead.innerHTML = ''; tbody.innerHTML = '';
                emptyMsg.classList.remove('hidden');
                return;
            }
            
            let datesForMapel = [];
            Object.keys(attendanceLog).forEach(key => {
                if(key.substring(11) === selectedMapel) datesForMapel.push(key.substring(0, 10));
            });
            datesForMapel.sort();
            
            let students = registeredStudents.filter(s => (s.className || 'Tanpa Kelas') === selectedClass);
            students.sort((a, b) => a.name.localeCompare(b.name));
            
            if(students.length === 0 || datesForMapel.length === 0) {
                thead.innerHTML = ''; tbody.innerHTML = '';
                emptyMsg.classList.remove('hidden');
                return;
            }
            
            emptyMsg.classList.add('hidden');
            
            let theadHTML = `<tr>
                <th class="px-4 py-3 text-center bg-gray-200 dark:bg-gray-700 sticky left-0 z-30 min-w-[50px]">No</th>
                <th class="px-4 py-3 bg-gray-200 dark:bg-gray-700 sticky left-[50px] z-30 whitespace-nowrap min-w-[200px]">Nama Siswa</th>`;
                
            datesForMapel.forEach((date, i) => {
                const dParts = date.split('-');
                const shortDate = dParts.length === 3 ? `${dParts[2]}/${dParts[1]}` : date;
                theadHTML += `<th class="px-3 py-3 text-center whitespace-nowrap" title="${date}">P${i+1}<br><span class="text-[10px] text-gray-500 dark:text-gray-400 font-normal">${shortDate}</span></th>`;
            });
            theadHTML += `<th class="px-4 py-3 text-center sticky right-0 bg-gray-200 dark:bg-gray-700 z-30">Total</th></tr>`;
            thead.innerHTML = theadHTML;
            
            let tbodyHTML = '';
            students.forEach((student, index) => {
                let totalHadir = 0;
                let rowHTML = `<tr class="bg-white dark:bg-gray-800 border-b dark:border-gray-700 hover:bg-gray-50 dark:hover:bg-gray-700">
                    <td class="px-4 py-3 text-center font-medium sticky left-0 bg-white dark:bg-gray-800 z-10 border-r dark:border-gray-700 min-w-[50px]">${index + 1}</td>
                    <td class="px-4 py-3 font-semibold whitespace-nowrap sticky left-[50px] bg-white dark:bg-gray-800 z-10 border-r dark:border-gray-700 min-w-[200px]">${student.name}</td>`;
                    
                datesForMapel.forEach(date => {
                    const logKey = `${date}_${selectedMapel}`;
                    const isPresent = attendanceLog[logKey] && attendanceLog[logKey].some(log => log.id === student.id);
                    if(isPresent) {
                        totalHadir++;
                        rowHTML += `<td class="px-3 py-3 text-center"><i class="fas fa-check text-green-500"></i></td>`;
                    } else {
                        rowHTML += `<td class="px-3 py-3 text-center font-bold text-gray-300 dark:text-gray-600">-</td>`;
                    }
                });
                
                const percent = Math.round((totalHadir / datesForMapel.length) * 100);
                rowHTML += `<td class="px-4 py-3 text-center sticky right-0 bg-white dark:bg-gray-800 z-10 border-l dark:border-gray-700">
                    <span class="font-bold text-purple-600 dark:text-purple-400">${totalHadir}</span> 
                    <span class="text-xs font-normal text-gray-500 block">${percent}%</span>
                </td></tr>`;
                tbodyHTML += rowHTML;
            });
            
            tbody.innerHTML = tbodyHTML;
        }

        function downloadRecapWord() {
            const selectedClass = document.getElementById('recapClassFilter').value;
            const selectedMapel = document.getElementById('recapMapelFilter').value;
            
            if(!selectedClass || !selectedMapel) return showToast("Pilih kelas dan mapel terlebih dahulu", true);
            
            let datesForMapel = [];
            Object.keys(attendanceLog).forEach(key => {
                if(key.substring(11) === selectedMapel) datesForMapel.push(key.substring(0, 10));
            });
            datesForMapel.sort(); 
            
            let students = registeredStudents.filter(s => (s.className || 'Tanpa Kelas') === selectedClass);
            students.sort((a, b) => a.name.localeCompare(b.name));
            
            if(students.length === 0 || datesForMapel.length === 0) return showToast("Tidak ada data untuk direkap", true);

            let htmlContent = `
            <html xmlns:o='urn:schemas-microsoft-com:office:office' xmlns:w='urn:schemas-microsoft-com:office:word' xmlns='http://www.w3.org/TR/REC-html40'>
            <head>
                <meta charset='utf-8'>
                <title>Rekap Kehadiran</title>
                <style>
                    body { font-family: 'Times New Roman', serif; color: #000; }
                    .header-title { text-align: center; font-size: 16pt; font-weight: bold; margin-bottom: 20px; text-transform: uppercase; }
                    .info-text { font-size: 12pt; margin-bottom: 5px; }
                    table { border-collapse: collapse; width: 100%; margin-top: 15px; }
                    th, td { border: 1px solid black; padding: 5px; font-size: 11pt; }
                    th { background-color: #e0e0e0; font-weight: bold; text-align: center; }
                    .text-center { text-align: center; }
                    .check-mark { font-family: Arial, sans-serif; font-weight: bold; color: #000; }
                </style>
            </head>
            <body>
                <div class="header-title">Rekapitulasi Kehadiran Siswa</div>
                <div class="info-text"><strong>Mata Pelajaran :</strong> ${selectedMapel}</div>
                <div class="info-text"><strong>Kelas          :</strong> ${selectedClass}</div>
                <div class="info-text"><strong>Total Pertemuan:</strong> ${datesForMapel.length} Kali</div>
                
                <table>
                    <thead>
                        <tr>
                            <th width="5%">No</th>
                            <th width="30%">Nama Siswa</th>`;
                            
            datesForMapel.forEach((date, i) => {
                const dParts = date.split('-');
                const shortDate = dParts.length === 3 ? `${dParts[2]}/${dParts[1]}` : date;
                htmlContent += `<th>P-${i+1}<br>${shortDate}</th>`;
            });
            
            htmlContent += `<th>Total<br>Hadir</th>
                        </tr>
            </thead>
            <tbody>`;
                    
            students.forEach((student, index) => {
                let totalHadir = 0;
                htmlContent += `<tr><td class="text-center">${index + 1}</td><td>${student.name}</td>`;
                
                datesForMapel.forEach(date => {
                    const logKey = `${date}_${selectedMapel}`;
                    const isPresent = attendanceLog[logKey] && attendanceLog[logKey].some(log => log.id === student.id);
                    if(isPresent) {
                        totalHadir++;
                        htmlContent += `<td class="text-center check-mark">&#10003;</td>`;
                    } else {
                        htmlContent += `<td class="text-center">-</td>`;
                    }
                });
                
                htmlContent += `<td class="text-center"><strong>${totalHadir}</strong></td></tr>`;
            });
            
            htmlContent += `
                    </tbody>
                </table>
                <br><br>
                <table style="border: none; width: 100%;">
                    <tr>
                        <td style="border: none; width: 60%;"></td>
                        <td style="border: none; width: 40%; text-align: center;">
                            Mengetahui,<br>Guru Mata Pelajaran
                            <br><br><br><br><br>
                            _________________________
                        </td>
                    </tr>
                </table>
            </body>
            </html>`;

            const blob = new Blob(['\ufeff', htmlContent], { type: 'application/msword' });
            const fileName = `Rekap_${selectedClass.replace(/[^a-zA-Z0-9]/g, '_')}_${selectedMapel.replace(/[^a-zA-Z0-9]/g, '_')}.doc`;
            const url = URL.createObjectURL(blob);
            const a = document.createElement("a");
            a.href = url;
            a.download = fileName;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        }

        function openEditModal(id) {
            const student = registeredStudents.find(s => s.id === id);
            if (student) {
                document.getElementById('editStudentId').value = student.id;
                document.getElementById('editStudentName').value = student.name;
                document.getElementById('editStudentClass').value = student.className || '';
                document.getElementById('editModal').classList.replace('hidden', 'flex');
            }
        }

        function closeEditModal() {
            document.getElementById('editModal').classList.replace('flex', 'hidden');
        }

        function saveEditStudent() {
            const id = document.getElementById('editStudentId').value;
            const newName = document.getElementById('editStudentName').value.trim();
            const newClass = document.getElementById('editStudentClass').value.trim();

            if (!newName || !newClass) {
                showToast("Nama dan Kelas tidak boleh kosong!", true);
                return;
            }

            const studentIndex = registeredStudents.findIndex(s => s.id === id);
            if (studentIndex !== -1) {
                // Update nama dan kelas tanpa menyentuh data descriptor wajah (array)
                registeredStudents[studentIndex].name = newName;
                registeredStudents[studentIndex].className = newClass;
                
                saveDataToStorage();
                updateUIList();
                closeEditModal();
                showToast("Data siswa berhasil diperbarui!");
            }
        }

        function openDeleteModal(id) {
            document.getElementById('deleteStudentId').value = id;
            document.getElementById('deleteModal').classList.replace('hidden', 'flex');
        }

        function closeDeleteModal() {
            document.getElementById('deleteModal').classList.replace('flex', 'hidden');
        }

        function confirmDeleteStudent() {
            const id = document.getElementById('deleteStudentId').value;
            registeredStudents = registeredStudents.filter(s => s.id !== id);
            saveDataToStorage();
            buildFaceMatcher(); 
            updateUIList();
            closeDeleteModal();
            showToast("Data siswa berhasil dihapus.");
        }

        function showToast(message, isError = false) {
            toastMsg.textContent = message;
            toast.className = `toast ${isError ? 'error' : ''} show`;
            toastIcon.className = isError ? 'fas fa-times-circle mr-2' : 'fas fa-check-circle mr-2';
            setTimeout(() => { toast.classList.remove('show'); }, 3000);
        }

        function toggleFullScreen() {
            const doc = window.document;
            const docEl = doc.documentElement;
            const btn = document.getElementById('fullscreenBtn');

            const requestFullScreen = docEl.requestFullscreen || docEl.mozRequestFullScreen || docEl.webkitRequestFullScreen || docEl.msRequestFullscreen;
            const cancelFullScreen = doc.exitFullscreen || doc.mozCancelFullScreen || doc.webkitExitFullscreen || doc.msExitFullscreen;

            if(!doc.fullscreenElement && !doc.mozFullScreenElement && !doc.webkitFullscreenElement && !doc.msFullscreenElement) {
                if (requestFullScreen) requestFullScreen.call(docEl);
            } else {
                if (cancelFullScreen) cancelFullScreen.call(doc);
            }
        }

        document.addEventListener('fullscreenchange', updateFullscreenButton);
        document.addEventListener('webkitfullscreenchange', updateFullscreenButton);
        document.addEventListener('mozfullscreenchange', updateFullscreenButton);
        document.addEventListener('MSFullscreenChange', updateFullscreenButton);

        function updateFullscreenButton() {
            const btn = document.getElementById('fullscreenBtn');
            if(!document.fullscreenElement && !document.mozFullScreenElement && !document.webkitFullscreenElement && !document.msFullscreenElement) {
                btn.innerHTML = '<i class="fas fa-expand mr-2"></i> Fullscreen';
            } else {
                btn.innerHTML = '<i class="fas fa-compress mr-2"></i> Keluar Fullscreen';
            }
        }

        window.addEventListener('load', init);

    </script>
</body>
</html>
