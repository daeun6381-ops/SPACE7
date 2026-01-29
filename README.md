<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Our Space | 우리만의 비밀 공간</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Pretendard:wght@400;600;700&display=swap');
        @import url('https://fonts.googleapis.com/css2?family=Gaegu:wght@400;700&display=swap');
        
        body { 
            font-family: 'Pretendard', sans-serif; 
            transition: background-color 0.5s ease; 
        }

        .memo-font { 
            font-family: 'Gaegu', cursive; 
            font-weight: 700; 
        }

        .fade-in { animation: fadeIn 0.4s ease-out forwards; }
        @keyframes fadeIn { 
            from { opacity: 0; transform: translateY(10px); } 
            to { opacity: 1; transform: translateY(0); } 
        }

        .scale-up { animation: scaleUp 0.3s cubic-bezier(0.16, 1, 0.3, 1) forwards; }
        @keyframes scaleUp {
            from { opacity: 0; transform: translate(-50%, -45%) scale(0.9); }
            to { opacity: 1; transform: translate(-50%, -50%) scale(1); }
        }

        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }

        .btn-active:active { transform: scale(0.95); }
        
        .chat-memo {
            position: relative;
            max-width: 85%;
            padding: 14px 18px;
            border-radius: 22px;
            font-size: 1.25rem;
            line-height: 1.4;
            box-shadow: 2px 4px 10px rgba(0,0,0,0.08);
            margin-bottom: 10px;
            word-break: break-all;
        }
        
        .chat-memo.mine {
            background: #fff9c4;
            align-self: flex-end;
            border-bottom-right-radius: 4px;
            transform: rotate(1deg);
            color: #4a4200;
        }
        
        .chat-memo.others {
            background: #e3f2fd;
            align-self: flex-start;
            border-bottom-left-radius: 4px;
            transform: rotate(-1deg);
            color: #0d3b66;
        }

        .photo-badge-top {
            background: linear-gradient(135deg, #ff7eb3 0%, #ff758c 100%);
            box-shadow: 0 4px 12px rgba(255, 117, 140, 0.4);
            color: white;
            padding: 4px 12px;
            border-radius: 12px;
            font-size: 13px;
            font-weight: 800;
        }

        .canvas-container {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 90%;
            max-width: 500px;
            height: 80vh;
            max-height: 650px;
            background: white;
            border-radius: 2.5rem;
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
            z-index: 160;
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        .dark-mode { background-color: #1a1a1a; color: #f3f4f6; }
        .dark-mode .bg-white { background-color: #262626; border-color: #333; }
        .dark-mode .canvas-container { background-color: #262626; }
        .dark-mode .chat-memo.mine { background-color: #5c5623 !important; color: #fff; }
        .dark-mode .chat-memo.others { background-color: #233e5c !important; color: #fff; }

        .sync-height {
            min-height: 560px;
        }
    </style>
</head>
<body class="bg-gray-50 text-gray-800 overflow-x-hidden min-h-screen">

    <!-- [1] 접속 보안 화면 -->
    <div id="auth-screen" class="fixed inset-0 z-[100] flex flex-col items-center justify-center bg-white p-6 transition-opacity duration-500">
        <div class="w-full max-w-sm text-center space-y-10">
            <div class="flex justify-center">
                <div id="lock-icon-bg" class="p-6 bg-pink-50 rounded-full text-pink-500 transition-colors duration-500">
                    <i data-lucide="lock" class="w-14 h-14"></i>
                </div>
            </div>
            <div class="space-y-2">
                <h1 class="text-3xl font-bold tracking-tight">Our Secret Space</h1>
                <p class="text-gray-500 text-base">비밀 코드를 입력하세요</p>
            </div>
            <input type="password" id="secret-code" maxlength="4" placeholder="••••" 
                   class="w-full text-center text-5xl tracking-[1.5rem] p-4 border-b-2 border-gray-200 focus:outline-none focus:border-pink-400 transition-all uppercase placeholder:tracking-normal">
            <p id="auth-error" class="text-red-400 text-base opacity-0 transition-opacity">코드가 올바르지 않습니다.</p>
        </div>
    </div>

    <!-- [3] 캔버스 그리기 오버레이 -->
    <div id="drawing-backdrop" class="fixed inset-0 z-[150] hidden bg-black/40 backdrop-blur-sm transition-all">
        <div id="canvas-popup" class="canvas-container scale-up">
            <div class="p-5 flex justify-between items-center border-b border-gray-100">
                <button onclick="closeDrawingCanvas()" class="p-2 text-gray-400 hover:text-gray-600 transition-colors"><i data-lucide="x" class="w-7 h-7"></i></button>
                <h3 class="font-bold text-lg">그림 메모</h3>
                <button onclick="sendDrawing()" id="drawing-send-btn" class="px-5 py-2 bg-pink-500 text-white rounded-full font-bold text-sm shadow-md shadow-pink-100">보내기</button>
            </div>
            <div class="flex-1 bg-[#F9FAFB] relative overflow-hidden">
                <canvas id="paint-canvas" class="w-full h-full cursor-crosshair touch-none"></canvas>
            </div>
            <div class="p-5 grid grid-cols-4 gap-3 bg-white border-t border-gray-100">
                <div class="col-span-4 flex items-center gap-4 mb-2">
                    <span class="text-xs font-bold text-gray-400 uppercase">두께</span>
                    <input type="range" id="pen-width" min="1" max="25" value="6" class="flex-1 accent-pink-500">
                </div>
                <button onclick="clearCanvas()" class="p-3 bg-gray-50 rounded-2xl flex items-center justify-center border border-gray-100 hover:bg-gray-100 transition-colors">
                    <i data-lucide="rotate-ccw" class="w-5 h-5 text-gray-500"></i>
                </button>
                <div class="relative p-1 bg-gray-50 rounded-2xl border border-gray-100 flex items-center justify-center">
                    <input type="color" id="pen-color" value="#FF6491" class="w-full h-8 cursor-pointer border-none bg-transparent">
                </div>
                <button onclick="toggleEraser()" id="eraser-btn" class="p-3 bg-gray-50 rounded-2xl flex items-center justify-center border border-gray-100 transition-colors">
                    <i data-lucide="eraser" class="w-5 h-5 text-gray-500"></i>
                </button>
                <button id="brush-indicator" class="p-3 bg-pink-50 rounded-2xl flex items-center justify-center border border-pink-100">
                   <i data-lucide="pencil" class="w-5 h-5 text-pink-500"></i>
                </button>
            </div>
        </div>
    </div>

    <!-- [2] 메인 애플리케이션 -->
    <div id="main-app" class="hidden opacity-0 transition-opacity duration-700">
        <header class="sticky top-0 z-40 bg-white/80 backdrop-blur-lg border-b border-gray-100 transition-colors duration-500">
            <div class="max-w-4xl mx-auto px-6 h-20 flex justify-between items-center">
                <div class="flex flex-col">
                    <div class="flex items-center gap-2">
                        <span id="header-dday" class="text-2xl font-black tracking-tighter text-pink-500 transition-colors duration-500">D+0</span>
                        <div id="online-indicator" class="w-2 h-2 rounded-full bg-green-400 animate-pulse hidden"></div>
                    </div>
                    <span class="text-[11px] text-gray-500 font-bold uppercase tracking-widest">Since 2025.05.28</span>
                </div>
                <div class="flex items-center gap-2">
                    <button onclick="toggleThemePanel()" class="p-2 rounded-full hover:bg-gray-100 btn-active">
                        <i data-lucide="palette" class="w-6 h-6 text-gray-500"></i>
                    </button>
                </div>
            </div>
            <div id="theme-panel" class="hidden max-w-4xl mx-auto px-6 pb-4 flex gap-3 overflow-x-auto no-scrollbar fade-in">
                <button onclick="changeTheme('pink')" class="flex-shrink-0 px-5 py-2 rounded-full bg-pink-100 text-pink-600 text-sm font-bold">Pink</button>
                <button onclick="changeTheme('blue')" class="flex-shrink-0 px-5 py-2 rounded-full bg-blue-100 text-blue-600 text-sm font-bold">Blue</button>
                <button onclick="changeTheme('green')" class="flex-shrink-0 px-5 py-2 rounded-full bg-emerald-100 text-emerald-600 text-sm font-bold">Green</button>
                <button onclick="changeTheme('dark')" class="flex-shrink-0 px-5 py-2 rounded-full bg-gray-800 text-white text-sm font-bold">Dark</button>
            </div>
        </header>

        <main class="max-w-5xl mx-auto p-6 pb-32 space-y-12">
            <div id="tab-home" class="tab-content space-y-12">
                <div class="relative">
                    <h4 class="font-bold text-xl mb-4 flex items-center justify-between">
                        <div class="flex items-center gap-2">
                            <i data-lucide="sparkles" class="w-6 h-6 text-yellow-400"></i>
                            STORY
                        </div>
                    </h4>
                    <div id="main-photo-slider" class="flex gap-4 overflow-x-auto no-scrollbar py-2 snap-x snap-mandatory min-h-[280px] scroll-smooth">
                        <div class="min-w-full h-72 bg-gray-100 rounded-[2.5rem] flex items-center justify-center text-gray-500 text-base">
                            사진을 불러오는 중입니다...
                        </div>
                    </div>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 items-stretch">
                    <div class="bg-white rounded-[2.5rem] p-8 shadow-xl shadow-gray-200/50 border border-gray-50 flex flex-col sync-height">
                        <div class="flex items-center gap-3 mb-6">
                            <div id="upload-icon-box" class="p-2.5 bg-pink-100 rounded-xl text-pink-500">
                                <i data-lucide="camera" class="w-6 h-6"></i>
                            </div>
                            <h4 class="font-bold text-lg">사진 업로드</h4>
                        </div>
                        
                        <div class="flex-1 flex flex-col space-y-4">
                            <div id="image-preview-container" class="hidden w-full h-48 bg-gray-50 rounded-2xl overflow-hidden relative border-2 border-dashed border-gray-200">
                                <img id="image-preview" src="" class="w-full h-full object-cover">
                                <button onclick="clearImageSelection()" class="absolute top-3 right-3 bg-black/50 text-white p-1.5 rounded-full hover:bg-black/70 transition-colors">
                                    <i data-lucide="x" class="w-5 h-5"></i>
                                </button>
                            </div>
                            
                            <textarea id="post-caption" placeholder="이날 어떤 추억이 있었나요?" class="flex-1 w-full p-5 bg-gray-50 rounded-2xl resize-none focus:outline-none focus:ring-2 ring-pink-100 transition-all text-base min-h-[120px]"></textarea>
                            
                            <div class="grid grid-cols-2 gap-3">
                                <input type="date" id="post-date" class="w-full p-4 bg-gray-50 rounded-xl text-sm focus:outline-none border-none">
                                <label for="post-file" class="flex items-center justify-center w-full p-4 bg-gray-100 rounded-xl text-xs font-bold cursor-pointer hover:bg-gray-200 transition-colors">
                                    <span id="file-name-label" class="truncate px-2">사진 선택하기</span>
                                    <input type="file" id="post-file" accept="image/*" class="hidden" onchange="handleFileSelect(event)">
                                </label>
                            </div>
                            
                            <button onclick="uploadPost()" id="upload-btn" class="w-full bg-pink-500 text-white py-5 rounded-2xl font-bold text-lg btn-active shadow-lg shadow-pink-100">기억 저장하기</button>
                        </div>
                    </div>

                    <div class="bg-white rounded-[2.5rem] p-8 shadow-xl shadow-gray-200/50 border border-gray-50 flex flex-col sync-height">
                        <div class="flex items-center gap-3 mb-6 px-2">
                            <i data-lucide="message-circle" class="w-6 h-6 text-pink-500"></i>
                            <h4 class="font-bold text-lg">메모 남기기</h4>
                        </div>
                        
                        <div id="memo-board" class="flex-1 flex flex-col gap-4 p-6 bg-orange-50/30 rounded-2xl overflow-y-auto no-scrollbar border-2 border-dashed border-orange-100 mb-5 min-h-[250px]">
                        </div>
                        
                        <div class="flex items-center gap-2.5">
                            <button onclick="openDrawingCanvas()" class="p-5 bg-white border border-gray-100 rounded-2xl shadow-sm text-pink-500 btn-active flex-shrink-0 hover:bg-gray-50">
                                <i data-lucide="brush" class="w-6 h-6"></i>
                            </button>
                            <div class="relative flex-1 shadow-sm">
                                <input type="text" id="memo-input" placeholder="메시지를 남겨주세요..." onkeypress="if(event.keyCode==13) sendMemo()" class="w-full p-5 bg-white rounded-2xl border border-gray-100 text-base focus:outline-none pr-14 focus:ring-2 ring-pink-50 transition-all">
                                <button onclick="sendMemo()" class="absolute right-3 top-1/2 -translate-y-1/2 text-pink-500 p-2 hover:scale-110 transition-transform">
                                    <i data-lucide="send" class="w-6 h-6"></i>
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <div id="tab-gallery" class="tab-content hidden space-y-8 fade-in">
                <div class="px-2">
                    <h4 class="font-bold text-3xl tracking-tight">앨범</h4>
                    <p class="text-gray-500 text-base font-medium">우리의 소중한 기록들</p>
                </div>
                <div id="gallery-grid" class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
                </div>
            </div>
        </main>

        <nav class="fixed bottom-8 left-1/2 -translate-x-1/2 w-[calc(100%-3rem)] max-w-md h-22 bg-white/90 backdrop-blur-xl border border-white/20 rounded-[2.5rem] shadow-2xl flex items-center justify-around px-4 z-50">
            <button onclick="switchTab('home')" id="nav-home" class="flex flex-col items-center gap-1.5 text-pink-500 transition-all duration-300 scale-110">
                <i data-lucide="heart" class="w-7 h-7"></i>
                <span class="text-[12px] font-bold">홈</span>
            </button>
            <button onclick="switchTab('gallery')" id="nav-gallery" class="flex flex-col items-center gap-1.5 text-gray-400 transition-all duration-300">
                <i data-lucide="layout-grid" class="w-7 h-7"></i>
                <span class="text-[12px] font-bold">앨범</span>
            </button>
        </nav>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.0.2/firebase-app.js";
        import { getFirestore, collection, addDoc, onSnapshot, query, orderBy, doc, setDoc, deleteDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.0.2/firebase-firestore.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.0.2/firebase-auth.js";

        const firebaseConfig = {
            apiKey: "AIzaSyDDPLecqluEvnmfwUqtaUFHuUReqTkybkc",
            authDomain: "space1-b521a.firebaseapp.com",
            projectId: "space1-b521a",
            storageBucket: "space1-b521a.firebasestorage.app",
            messagingSenderId: "235223237323",
            appId: "1:235223237323:web:d5cbf1999654707e65514e",
            measurementId: "G-5K1ETJG4T6"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'our-secret-space-v5';
        const START_DATE = new Date('2025-05-28');

        let isAuthInProgress = false;
        let isAppInitialized = false;
        let selectedImageData = null;

        // --- 캔버스 드로잉 로직 ---
        const canvas = document.getElementById('paint-canvas');
        const ctx = canvas.getContext('2d');
        let painting = false;
        let isEraser = false;

        function getPos(e) {
            const rect = canvas.getBoundingClientRect();
            const clientX = e.touches ? e.touches[0].clientX : e.clientX;
            const clientY = e.touches ? e.touches[0].clientY : e.clientY;
            return { x: clientX - rect.left, y: clientY - rect.top };
        }

        function startPosition(e) {
            painting = true;
            const pos = getPos(e);
            ctx.beginPath();
            ctx.moveTo(pos.x, pos.y);
            if(e.cancelable) e.preventDefault();
        }

        function finishedPosition() {
            painting = false;
            ctx.beginPath();
        }

        function draw(e) {
            if (!painting) return;
            const pos = getPos(e);
            ctx.lineWidth = document.getElementById('pen-width').value;
            ctx.lineCap = 'round';
            ctx.strokeStyle = isEraser ? '#F9FAFB' : document.getElementById('pen-color').value;
            ctx.lineTo(pos.x, pos.y);
            ctx.stroke();
            ctx.beginPath();
            ctx.moveTo(pos.x, pos.y);
            if(e.cancelable) e.preventDefault();
        }

        window.openDrawingCanvas = () => {
            document.getElementById('drawing-backdrop').classList.remove('hidden');
            const parent = canvas.parentElement;
            canvas.width = parent.clientWidth;
            canvas.height = parent.clientHeight;
            ctx.fillStyle = '#F9FAFB';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            canvas.onmousedown = startPosition;
            canvas.onmouseup = finishedPosition;
            canvas.onmousemove = draw;
            canvas.ontouchstart = startPosition;
            canvas.ontouchend = finishedPosition;
            canvas.ontouchmove = draw;
        };

        window.closeDrawingCanvas = () => document.getElementById('drawing-backdrop').classList.add('hidden');
        window.clearCanvas = () => { ctx.fillStyle = '#F9FAFB'; ctx.fillRect(0, 0, canvas.width, canvas.height); };
        window.toggleEraser = () => {
            isEraser = !isEraser;
            document.getElementById('eraser-btn').classList.toggle('bg-pink-100', isEraser);
            document.getElementById('brush-indicator').classList.toggle('bg-pink-50', !isEraser);
        };

        window.sendDrawing = async () => {
            if (!auth.currentUser) return;
            const btn = document.getElementById('drawing-send-btn');
            btn.disabled = true;
            btn.innerText = '전송중...';
            const dataUrl = canvas.toDataURL('image/png', 0.5);
            try {
                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'memos'), {
                    type: 'image', imgUrl: dataUrl, createdAt: serverTimestamp(), userId: auth.currentUser.uid
                });
                closeDrawingCanvas();
            } finally {
                btn.disabled = false;
                btn.innerText = '보내기';
            }
        };

        // --- 인증 로직 수정: mismatch 에러 방지 ---
        async function authenticate() {
            if (isAuthInProgress) return;
            isAuthInProgress = true;
            try {
                // 토큰 불일치 에러 방지를 위해 try-catch로 감싸고 실패 시 익명 로그인 시도
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) { 
                    try {
                        await signInWithCustomToken(auth, __initial_auth_token); 
                    } catch (tokenError) {
                        console.warn("Custom token failed, falling back to anonymous:", tokenError);
                        await signInAnonymously(auth);
                    }
                } else { 
                    await signInAnonymously(auth); 
                }
            } catch (err) {
                console.error("Auth failed:", err);
            } finally { 
                isAuthInProgress = false; 
            }
        }

        onAuthStateChanged(auth, (user) => {
            if (user) {
                document.getElementById('online-indicator')?.classList.remove('hidden');
                if (!document.getElementById('main-app').classList.contains('hidden')) initApp();
            }
        });

        document.getElementById('secret-code').addEventListener('input', async (e) => {
            if (e.target.value.toUpperCase() === 'EVOL') {
                if (!auth.currentUser) await authenticate();
                document.getElementById('auth-screen').classList.add('opacity-0');
                setTimeout(() => {
                    document.getElementById('auth-screen').style.display = 'none';
                    document.getElementById('main-app').classList.remove('hidden');
                    setTimeout(() => {
                        document.getElementById('main-app').classList.add('opacity-100');
                        initApp();
                    }, 50);
                }, 500);
            }
        });

        function initApp() {
            if (isAppInitialized || !auth.currentUser) return;
            isAppInitialized = true;
            lucide.createIcons();
            updateDDay();
            listenToTheme();
            listenToPosts();
            listenToMemos();
            document.getElementById('post-date').valueAsDate = new Date();
        }

        function updateDDay() {
            const diff = Math.floor((new Date() - START_DATE) / (1000 * 60 * 60 * 24));
            document.getElementById('header-dday').innerText = `D+${diff}`;
        }

        window.changeTheme = async (theme) => {
            if (!auth.currentUser) return;
            await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'settings', 'currentTheme'), { name: theme });
        };

        function applyTheme(theme) {
            document.body.classList.toggle('dark-mode', theme === 'dark');
            const colorMap = { pink: 'text-pink-500', blue: 'text-blue-500', green: 'text-emerald-500', dark: 'text-white' };
            document.getElementById('header-dday').className = `text-2xl font-black tracking-tighter ${colorMap[theme] || 'text-pink-500'}`;
        }

        function listenToTheme() {
            onSnapshot(doc(db, 'artifacts', appId, 'public', 'data', 'settings', 'currentTheme'), (s) => {
                if (s.exists()) applyTheme(s.data().name);
            });
        }

        window.handleFileSelect = async (e) => {
            const file = e.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = (ev) => {
                selectedImageData = ev.target.result;
                document.getElementById('image-preview').src = selectedImageData;
                document.getElementById('image-preview-container').classList.remove('hidden');
                document.getElementById('file-name-label').innerText = file.name;
            };
            reader.readAsDataURL(file);
        };

        window.clearImageSelection = () => {
            selectedImageData = null;
            document.getElementById('image-preview-container').classList.add('hidden');
            document.getElementById('file-name-label').innerText = "사진 선택하기";
            document.getElementById('post-file').value = '';
        };

        window.uploadPost = async () => {
            const caption = document.getElementById('post-caption').value;
            const dateVal = document.getElementById('post-date').value;
            if (!selectedImageData && !caption) return;
            if (!auth.currentUser) return;
            
            await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'posts'), {
                caption, imgUrl: selectedImageData || '', dateVal,
                createdAt: serverTimestamp(), daysDiff: Math.floor((new Date(dateVal) - START_DATE) / (1000*60*60*24)) + 1,
                userId: auth.currentUser.uid
            });
            document.getElementById('post-caption').value = '';
            clearImageSelection();
        };

        function listenToPosts() {
            const q = query(collection(db, 'artifacts', appId, 'public', 'data', 'posts'), orderBy('dateVal', 'asc'));
            onSnapshot(q, (snapshot) => {
                const slider = document.getElementById('main-photo-slider');
                slider.innerHTML = '';
                snapshot.forEach(snap => {
                    const p = snap.data();
                    const slide = document.createElement('div');
                    slide.className = 'min-w-[85%] lg:min-w-[45%] h-72 relative rounded-[2.5rem] overflow-hidden shadow-xl snap-center flex-shrink-0 fade-in cursor-pointer';
                    slide.ondblclick = () => deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'posts', snap.id));
                    slide.innerHTML = `
                        <img src="${p.imgUrl}" class="w-full h-full object-cover">
                        <div class="absolute inset-0 bg-gradient-to-t from-black/60 to-transparent"></div>
                        <div class="absolute top-4 left-4 photo-badge-top">Day ${p.daysDiff}</div>
                        <div class="absolute bottom-4 left-4 text-white font-bold px-2">${p.caption || ''}</div>
                    `;
                    slider.appendChild(slide);
                });
            });
        }

        window.sendMemo = async () => {
            const input = document.getElementById('memo-input');
            if (!input.value.trim() || !auth.currentUser) return;
            await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'memos'), {
                type: 'text', text: input.value, createdAt: serverTimestamp(), userId: auth.currentUser.uid
            });
            input.value = '';
        };

        function listenToMemos() {
            if(!auth.currentUser) return;
            const q = query(collection(db, 'artifacts', appId, 'public', 'data', 'memos'), orderBy('createdAt', 'asc'));
            onSnapshot(q, (snapshot) => {
                const board = document.getElementById('memo-board');
                board.innerHTML = '';
                snapshot.forEach(docSnap => {
                    const m = docSnap.data();
                    const isMine = m.userId === auth.currentUser.uid;
                    const note = document.createElement('div');
                    note.className = `chat-memo ${isMine ? 'mine' : 'others'} memo-font fade-in`;
                    if (m.type === 'image') {
                        note.innerHTML = `<img src="${m.imgUrl}" class="rounded-xl w-full max-w-[220px] shadow-inner">`;
                    } else {
                        note.innerText = m.text;
                    }
                    board.appendChild(note);
                });
                board.scrollTo({ top: board.scrollHeight, behavior: 'smooth' });
            });
        }

        window.switchTab = (tab) => {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            document.getElementById(`tab-${tab}`).classList.remove('hidden');
            const navHome = document.getElementById('nav-home');
            const navGallery = document.getElementById('nav-gallery');
            if (tab === 'home') {
                navHome.classList.add('text-pink-500', 'scale-110'); navHome.classList.remove('text-gray-400');
                navGallery.classList.add('text-gray-400'); navGallery.classList.remove('text-pink-500', 'scale-110');
            } else {
                navGallery.classList.add('text-pink-500', 'scale-110'); navGallery.classList.remove('text-gray-400');
                navHome.classList.add('text-gray-400'); navHome.classList.remove('text-pink-500', 'scale-110');
            }
            lucide.createIcons();
        };

        window.toggleThemePanel = () => document.getElementById('theme-panel').classList.toggle('hidden');
    </script>
</body>
</html>
