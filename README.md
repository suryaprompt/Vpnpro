<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>VPN Global & Uji Kecepatan Game</title>
    <!-- Ketergantungan Eksternal: Tailwind CSS untuk styling dan Font Awesome untuk ikon -->
    <!-- Pastikan Anda memiliki koneksi internet agar ini dapat dimuat -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #0d1117;
            color: #c9d1d9;
        }
        .server-list-bg { background-color: #161b22; }
        .main-panel-bg { background-color: #0d1117; }
        .server-item {
            border-left: 4px solid transparent;
            transition: all 0.2s ease-in-out;
        }
        .server-item:hover {
            background-color: #1f2a38;
            border-left-color: #58a6ff;
        }
        .server-item.selected {
            background-color: #28374c;
            border-left-color: #3b82f6;
        }
        .btn-primary {
            background-color: #238636;
            transition: background-color 0.2s;
        }
        .btn-primary:hover { background-color: #2ea043; }
        .btn-primary:disabled {
            background-color: #30363d;
            cursor: not-allowed;
            opacity: 0.7;
        }
        .btn-disconnect { background-color: #da3633; }
        .btn-disconnect:hover { background-color: #f85149; }
        .result-card, .user-card {
            background: rgba(31, 41, 55, 0.5);
            backdrop-filter: blur(10px);
            border: 1px solid #30363d;
        }
        .status-dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
        }
        .status-dot.connected {
            background-color: #238636;
            animation: pulse-connected 2s infinite;
        }
        @keyframes pulse-connected {
            0%, 100% { box-shadow: 0 0 0 0 rgba(35, 134, 54, 0.7); }
            70% { box-shadow: 0 0 0 10px rgba(35, 134, 54, 0); }
        }
        .status-dot.disconnected {
            background-color: #da3633;
        }
        #recommendationBox { border: 1px solid #30363d; }
        .modal-backdrop {
            position: fixed;
            top: 0; left: 0; right: 0; bottom: 0;
            background-color: rgba(0,0,0,0.7);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 50;
            transition: opacity 0.3s ease;
        }
        .modal-content {
            background-color: #161b22;
            border: 1px solid #30363d;
            transition: transform 0.3s ease;
        }
        .pricing-card {
            border: 2px solid #30363d;
            transition: all 0.2s ease-in-out;
        }
        .pricing-card:hover {
            border-color: #58a6ff;
            transform: translateY(-5px);
        }
        /* Styling untuk Notifikasi Toast */
        #toast {
            position: fixed;
            bottom: 20px;
            left: 50%;
            transform: translate(-50%, 100px);
            padding: 1rem 1.5rem;
            border-radius: 0.5rem;
            color: white;
            font-weight: 600;
            z-index: 100;
            transition: transform 0.5s ease-in-out;
            opacity: 0;
        }
        #toast.show {
            transform: translate(-50%, 0);
            opacity: 1;
        }
        .toast-success { background-color: #238636; }
    </style>
</head>
<body class="min-h-screen">
    <div class="flex flex-col md:flex-row h-screen">
        <!-- Server List Panel (Left) -->
        <aside class="w-full md:w-1/3 lg:w-1/4 server-list-bg border-r border-gray-800 flex flex-col">
            <header class="p-4 border-b border-gray-800 flex justify-between items-center">
                <h2 class="text-xl font-bold text-white">Pilih Server VPN</h2>
                <span id="premiumBadgeHeader" class="hidden text-xs bg-yellow-500 text-black px-2 py-1 rounded-full font-bold">PREMIUM</span>
            </header>
            <div id="serverList" class="overflow-y-auto flex-grow p-2">
                <!-- Server list will be populated by JS -->
            </div>
        </aside>

        <!-- Main Panel (Right) -->
        <main class="w-full md:w-2/3 lg:w-3/4 main-panel-bg p-6 md:p-8 flex flex-col overflow-y-auto">
            <!-- Connection Status & Premium Button -->
            <section class="mb-6">
                <div class="flex items-center justify-between p-4 rounded-lg bg-gray-800 border border-gray-700">
                    <div class="flex items-center gap-4">
                         <div id="statusDot" class="status-dot disconnected"></div>
                         <div>
                            <h3 id="connectionStatus" class="text-lg font-semibold text-white">Terputus</h3>
                            <p id="connectionInfo" class="text-sm text-gray-400">Pilih server dan klik 'Hubungkan'</p>
                         </div>
                    </div>
                     <button id="premiumButton" class="bg-yellow-500 hover:bg-yellow-600 text-black font-bold py-2 px-4 rounded-md text-sm whitespace-nowrap">
                        <i class="fas fa-crown mr-2"></i>Go Premium
                    </button>
                    <button id="connectButton" class="px-6 py-2 font-bold text-white rounded-md btn-primary ml-2" disabled>Hubungkan</button>
                </div>
            </section>

            <!-- Speed Test Section -->
            <section class="mb-8">
                 <h2 class="text-2xl font-bold text-white mb-4">Uji Kecepatan Jaringan</h2>
                 <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 md:gap-6 mb-6">
                    <div class="result-card p-4 rounded-lg text-center">
                        <h3 class="text-sm font-semibold text-gray-400">PING</h3>
                        <p id="pingResult" class="text-4xl font-extrabold text-white my-2">-</p>
                        <p class="text-xs text-gray-500">ms</p>
                    </div>
                    <div class="result-card p-4 rounded-lg text-center">
                        <h3 class="text-sm font-semibold text-gray-400">JITTER</h3>
                        <p id="jitterResult" class="text-4xl font-extrabold text-white my-2">-</p>
                        <p class="text-xs text-gray-500">ms</p>
                    </div>
                    <div class="result-card p-4 rounded-lg text-center">
                        <h3 class="text-sm font-semibold text-gray-400">DOWNLOAD</h3>
                        <p id="downloadResult" class="text-4xl font-extrabold text-white my-2">-</p>
                        <p class="text-xs text-gray-500">Mbps</p>
                    </div>
                    <div class="result-card p-4 rounded-lg text-center">
                        <h3 class="text-sm font-semibold text-gray-400">UPLOAD</h3>
                        <p id="uploadResult" class="text-4xl font-extrabold text-white my-2">-</p>
                        <p class="text-xs text-gray-500">Mbps</p>
                    </div>
                 </div>
                 <div class="text-center">
                    <button id="testButton" class="px-8 py-3 font-bold text-white rounded-md btn-primary w-full md:w-auto" disabled>Mulai Tes</button>
                    <p id="testStatus" class="mt-4 h-6 text-gray-400"></p>
                 </div>
                 <div id="recommendationBox" class="mt-6 p-6 rounded-lg bg-gray-900 hidden">
                     <h3 id="recommendationTitle" class="text-xl font-bold mb-2"></h3>
                     <p id="recommendationText" class="text-gray-300"></p>
                 </div>
            </section>

            <!-- Premium Users Online Section -->
            <section id="premiumUsersSection" class="hidden">
                 <h2 class="text-2xl font-bold text-white mb-4 flex items-center"><i class="fas fa-users text-yellow-400 mr-3"></i>Pengguna Premium Online</h2>
                 <div id="premiumUsersList" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                     <!-- Premium user cards will be populated by JS -->
                 </div>
            </section>
        </main>
    </div>

    <!-- Pricing Modal -->
    <div id="pricingModal" class="modal-backdrop hidden opacity-0">
        <div class="modal-content w-full max-w-3xl rounded-lg p-6 md:p-8 relative transform scale-95">
            <button id="closePricingModal" class="absolute top-4 right-4 text-gray-400 hover:text-white text-3xl">&times;</button>
            <h2 class="text-2xl md:text-3xl font-bold text-center text-white mb-6">Pilih Paket Premium Anda</h2>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                <div class="pricing-card p-6 rounded-lg text-center cursor-pointer">
                    <h3 class="text-xl font-bold text-white">1 Bulan</h3>
                    <p class="text-3xl font-extrabold text-white my-4">Rp 75.000</p>
                    <button class="w-full py-2 bg-blue-600 hover:bg-blue-700 text-white font-bold rounded-md choose-plan-btn">Pilih Paket</button>
                </div>
                <div class="pricing-card p-6 rounded-lg text-center cursor-pointer relative border-blue-500">
                    <span class="absolute -top-3 left-1/2 -translate-x-1/2 bg-blue-500 text-white text-xs font-bold px-3 py-1 rounded-full">Paling Hemat</span>
                    <h3 class="text-xl font-bold text-white">1 Tahun</h3>
                    <p class="text-3xl font-extrabold text-white my-4">Rp 600.000</p>
                     <p class="text-sm text-gray-400 mb-4">(Rp 50.000/bulan)</p>
                    <button class="w-full py-2 bg-blue-600 hover:bg-blue-700 text-white font-bold rounded-md choose-plan-btn">Pilih Paket</button>
                </div>
                <div class="pricing-card p-6 rounded-lg text-center cursor-pointer">
                    <h3 class="text-xl font-bold text-white">6 Bulan</h3>
                    <p class="text-3xl font-extrabold text-white my-4">Rp 360.000</p>
                    <button class="w-full py-2 bg-blue-600 hover:bg-blue-700 text-white font-bold rounded-md choose-plan-btn">Pilih Paket</button>
                </div>
            </div>
            <p class="text-center text-gray-500 text-xs mt-6">Ini adalah simulasi. Tidak ada biaya nyata yang akan dikenakan.</p>
        </div>
    </div>
    
    <!-- Toast Notification Element -->
    <div id="toast"></div>

<script>
document.addEventListener('DOMContentLoaded', () => {
    // --- DATABASE (Simulasi) ---
    const servers = [
        { country: "Singapura", city: "Singapore", ip: "172.104.88.1", isGamingOptimized: true, basePing: 20, baseDownload: 250, baseUpload: 150 },
        { country: "Singapura", city: "Singapore #2", ip: "45.76.183.1", isGamingOptimized: false, basePing: 45, baseDownload: 150, baseUpload: 80 },
        { country: "Jepang", city: "Tokyo", ip: "139.162.65.1", isGamingOptimized: true, basePing: 60, baseDownload: 300, baseUpload: 200 },
        { country: "Jepang", city: "Osaka", ip: "172.104.99.2", isGamingOptimized: false, basePing: 75, baseDownload: 200, baseUpload: 100 },
        { country: "Amerika Serikat", city: "Los Angeles", ip: "192.158.1.38", isGamingOptimized: true, basePing: 150, baseDownload: 400, baseUpload: 250 },
        { country: "Jerman", city: "Frankfurt", ip: "172.104.166.1", isGamingOptimized: true, basePing: 190, baseDownload: 380, baseUpload: 220 },
        { country: "Indonesia", city: "Jakarta", ip: "103.152.4.1", isGamingOptimized: true, basePing: 10, baseDownload: 100, baseUpload: 50 },
    ];
    const fakePremiumUsers = [
        { username: 'GamerPro_ID', country: 'Indonesia', flag: '🇮🇩', server: 'Jakarta' },
        { username: 'Slayer_SG', country: 'Singapura', flag: '🇸🇬', server: 'Singapore' },
        { username: 'TokyoFlash', country: 'Jepang', flag: '🇯🇵', server: 'Tokyo' },
        { username: 'BerlinBeast', country: 'Jerman', flag: '🇩🇪', server: 'Frankfurt' },
        { username: 'LA_Vibes', country: 'Amerika', flag: '🇺🇸', server: 'Los Angeles' },
        { username: 'LionCityRoar', country: 'Singapura', flag: '🇸🇬', server: 'Singapore #2' }
    ];

    // --- ELEMENT SELECTORS ---
    const serverListEl = document.getElementById('serverList');
    const connectButton = document.getElementById('connectButton');
    const testButton = document.getElementById('testButton');
    const connectionStatusEl = document.getElementById('connectionStatus');
    const connectionInfoEl = document.getElementById('connectionInfo');
    const statusDot = document.getElementById('statusDot');
    const testStatusEl = document.getElementById('testStatus');
    const pingResult = document.getElementById('pingResult');
    const jitterResult = document.getElementById('jitterResult');
    const downloadResult = document.getElementById('downloadResult');
    const uploadResult = document.getElementById('uploadResult');
    const recommendationBox = document.getElementById('recommendationBox');
    const recommendationTitle = document.getElementById('recommendationTitle');
    const recommendationText = document.getElementById('recommendationText');
    const premiumButton = document.getElementById('premiumButton');
    const pricingModal = document.getElementById('pricingModal');
    const modalContent = pricingModal.querySelector('.modal-content');
    const closePricingModal = document.getElementById('closePricingModal');
    const premiumBadgeHeader = document.getElementById('premiumBadgeHeader');
    const premiumUsersSection = document.getElementById('premiumUsersSection');
    const premiumUsersList = document.getElementById('premiumUsersList');
    const toastEl = document.getElementById('toast');

    // --- STATE MANAGEMENT ---
    let selectedServer = null;
    let isConnected = false;
    let isTesting = false;
    let isPremium = false;

    // --- RENDER FUNCTIONS ---
    function renderServers() {
        const serversToRender = isPremium ? servers.map(s => ({...s, isGamingOptimized: true})) : servers;
        const groupedServers = serversToRender.reduce((acc, server) => {
            if (!acc[server.country]) acc[server.country] = [];
            acc[server.country].push(server);
            return acc;
        }, {});
        let html = '';
        for (const country in groupedServers) {
            html += `<div class="mb-2">
                <h3 class="font-semibold text-gray-400 px-2 py-1 text-sm">${country}</h3>`;
            groupedServers[country].forEach(server => {
                html += `
                    <div class="server-item p-3 cursor-pointer rounded-md flex justify-between items-center" data-ip="${server.ip}">
                        <div>
                            <p class="font-semibold text-white">${server.city}</p>
                            <p class="text-xs text-gray-500">${server.ip}</p>
                        </div>
                        ${server.isGamingOptimized ? '<span class="text-xs bg-green-800 text-green-300 px-2 py-1 rounded-full font-bold">Gaming</span>' : ''}
                    </div>
                `;
            });
            html += `</div>`;
        }
        serverListEl.innerHTML = html;
        addServerEventListeners();
    }

    function displayPremiumUsers() {
        let html = '';
        fakePremiumUsers.forEach(user => {
            html += `
            <div class="user-card rounded-lg p-4 flex items-center">
                <span class="text-3xl mr-4">${user.flag}</span>
                <div>
                    <p class="font-bold text-white">${user.username}</p>
                    <p class="text-sm text-gray-400">
                        <i class="fas fa-server mr-1"></i> Terhubung ke ${user.server}
                    </p>
                </div>
            </div>
            `;
        });
        premiumUsersList.innerHTML = html;
        premiumUsersSection.classList.remove('hidden');
    }

    // --- EVENT LISTENERS ---
    function addServerEventListeners() {
        document.querySelectorAll('.server-item').forEach(item => {
            item.addEventListener('click', () => {
                if (isConnected) return;
                document.querySelectorAll('.server-item').forEach(el => el.classList.remove('selected'));
                item.classList.add('selected');
                selectedServer = servers.find(s => s.ip === item.dataset.ip);
                connectButton.disabled = false;
            });
        });
    }

    connectButton.addEventListener('click', () => {
        if (!selectedServer) return;
        isConnected ? handleDisconnect() : handleConnect();
    });

    testButton.addEventListener('click', runSpeedTest);

    premiumButton.addEventListener('click', () => {
        pricingModal.classList.remove('hidden');
        setTimeout(() => {
            pricingModal.classList.remove('opacity-0');
            modalContent.classList.remove('scale-95');
        }, 10);
    });

    const closePricing = () => {
        pricingModal.classList.add('opacity-0');
        modalContent.classList.add('scale-95');
        setTimeout(() => {
            pricingModal.classList.add('hidden');
        }, 300);
    };

    closePricingModal.addEventListener('click', closePricing);
    
    document.querySelectorAll('.choose-plan-btn').forEach(button => {
        button.addEventListener('click', () => {
            closePricing();
            setTimeout(() => {
                showToast('Pembayaran Berhasil! Akun Anda kini Premium.');
                activatePremium();
            }, 300);
        });
    });

    // --- LOGIC FUNCTIONS ---
    function showToast(message, type = 'success') {
        toastEl.textContent = message;
        toastEl.className = 'toast-success'; // Reset classes
        toastEl.classList.add('show');
        setTimeout(() => {
            toastEl.classList.remove('show');
        }, 3000);
    }
    
    function activatePremium() {
        isPremium = true;
        premiumButton.innerHTML = `<i class="fas fa-check-circle mr-2"></i>Akun Premium`;
        premiumButton.classList.remove('bg-yellow-500', 'hover:bg-yellow-600');
        premiumButton.classList.add('bg-green-600');
        premiumButton.disabled = true;
        premiumBadgeHeader.classList.remove('hidden');
        renderServers();
        displayPremiumUsers();
    }
    
    function handleConnect() {
        isConnected = true;
        serverListEl.style.pointerEvents = 'none';
        serverListEl.style.opacity = '0.5';
        connectButton.innerHTML = `<i class="fas fa-spinner fa-spin mr-2"></i>Menghubungkan...`;
        connectButton.disabled = true;

        setTimeout(() => {
            connectionStatusEl.textContent = 'Terhubung';
            connectionInfoEl.textContent = `${selectedServer.city}, ${selectedServer.country} (${selectedServer.ip})`;
            statusDot.classList.remove('disconnected');
            statusDot.classList.add('connected');
            connectButton.innerHTML = 'Putuskan';
            connectButton.classList.remove('btn-primary');
            connectButton.classList.add('btn-disconnect');
            connectButton.disabled = false;
            testButton.disabled = false;
        }, 1500);
    }
    
    function handleDisconnect() {
        isConnected = false;
        isTesting = false;
        connectionStatusEl.textContent = 'Terputus';
        connectionInfoEl.textContent = "Pilih server dan klik 'Hubungkan'";
        statusDot.classList.remove('connected');
        statusDot.classList.add('disconnected');
        connectButton.innerHTML = 'Hubungkan';
        connectButton.classList.remove('btn-disconnect');
        connectButton.classList.add('btn-primary');
        connectButton.disabled = true;
        testButton.disabled = true;
        serverListEl.style.pointerEvents = 'auto';
        serverListEl.style.opacity = '1';
        selectedServer = null;
        document.querySelectorAll('.server-item').forEach(el => el.classList.remove('selected'));
        resetTestResults();
    }

    function resetTestResults() {
        pingResult.textContent = '-';
        jitterResult.textContent = '-';
        downloadResult.textContent = '-';
        uploadResult.textContent = '-';
        testStatusEl.textContent = '';
        recommendationBox.classList.add('hidden');
    }

    const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));

    function animateValue(element, end, duration) {
        let start = 0;
        let startTimestamp = null;
        const step = (timestamp) => {
            if (!startTimestamp) startTimestamp = timestamp;
            const progress = Math.min((timestamp - startTimestamp) / duration, 1);
            const currentVal = progress * end;
            element.textContent = element.id.includes('Result') ? currentVal.toFixed(element.id.includes('download') || element.id.includes('upload') ? 2 : 0) : currentVal.toFixed(0);
            if (progress < 1) window.requestAnimationFrame(step);
        };
        window.requestAnimationFrame(step);
    }

    async function runSpeedTest() {
        if (!isConnected || isTesting) return;
        isTesting = true;
        testButton.disabled = true;
        testButton.innerHTML = `<i class="fas fa-spinner fa-spin mr-2"></i>Menguji...`;
        resetTestResults();

        const effectiveGamingOptimized = isPremium || selectedServer.isGamingOptimized;
        const { basePing, baseDownload, baseUpload } = selectedServer;

        testStatusEl.textContent = 'Mengecek Ping & Jitter...';
        await sleep(1500);
        const ping = basePing + Math.floor(Math.random() * (effectiveGamingOptimized ? 15 : 40));
        const jitter = Math.floor(ping / 10) + Math.floor(Math.random() * (effectiveGamingOptimized ? 5 : 15));
        animateValue(pingResult, ping, 1000);
        animateValue(jitterResult, jitter, 1000);

        testStatusEl.textContent = 'Mengecek Kecepatan Download...';
        await sleep(2500);
        const download = baseDownload * (0.8 + Math.random() * 0.4);
        animateValue(downloadResult, download, 1500);

        testStatusEl.textContent = 'Mengecek Kecepatan Upload...';
        await sleep(2500);
        const upload = baseUpload * (0.8 + Math.random() * 0.4);
        animateValue(uploadResult, upload, 1500);

        await sleep(1000);
        testStatusEl.textContent = 'Tes Selesai!';
        giveRecommendation(ping, jitter);
        isTesting = false;
        testButton.disabled = false;
        testButton.innerHTML = 'Ulangi Tes';
    }

    function giveRecommendation(ping, jitter) {
        let title, text, colorClass;
        if (ping <= 40 && jitter <= 10) {
            title = "🏆 Performa Sempurna untuk Gaming!";
            text = `Server ini luar biasa! Dengan ping ${ping}ms dan jitter ${jitter}ms, Anda akan mendapatkan pengalaman bermain game kompetitif yang sangat mulus, tanpa lag, dan super responsif.`;
            colorClass = "text-green-400";
        } else if (ping <= 80 && jitter <= 20) {
            title = "👍 Sangat Direkomendasikan untuk Game";
            text = `Koneksi yang sangat baik. Ping ${ping}ms dan jitter ${jitter}ms lebih dari cukup untuk bermain game online dengan lancar. Lag tidak akan menjadi masalah.`;
            colorClass = "text-blue-400";
        } else if (ping <= 150 && jitter <= 35) {
            title = "😐 Cukup Baik, Namun Hati-hati";
            text = `Anda masih bisa bermain, namun dengan ping ${ping}ms, terkadang mungkin terasa ada sedikit delay. Cukup untuk game kasual, tapi mungkin kurang ideal untuk game kompetitif.`;
            colorClass = "text-yellow-400";
        } else {
            title = "😞 Tidak Direkomendasikan untuk Game";
            text = `Dengan ping di atas 150ms (${ping}ms), bermain game online kompetitif akan sangat sulit. Anda akan mengalami lag signifikan yang akan mengganggu permainan. Server ini lebih cocok untuk browsing atau streaming.`;
            colorClass = "text-red-400";
        }
        recommendationTitle.textContent = title;
        recommendationTitle.className = `text-xl font-bold mb-2 ${colorClass}`;
        recommendationText.textContent = text;
        recommendationBox.classList.remove('hidden');
    }
    
    // --- INITIALIZATION ---
    renderServers();
    updateUIForDisconnection();
});
</script>
</body>
</html>

