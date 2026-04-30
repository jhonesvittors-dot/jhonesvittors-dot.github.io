<!DOCTYPE html>
<html lang="pt-pt">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Backrooms 3D - CatNap Edition</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        body { margin: 0; overflow: hidden; background: #000; color: #eee; font-family: 'Courier New', Courier, monospace; }
        canvas { display: block; }
        #ui-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 50; }
        #crosshair { position: absolute; top: 50%; left: 50%; width: 4px; height: 4px; background: rgba(255,255,255,0.5); border-radius: 50%; transform: translate(-50%, -50%); }
        #vhs-effect { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: radial-gradient(circle, transparent 20%, black 150%); pointer-events: none; opacity: 0.4; z-index: 10; }
        
        .menu-panel { 
            pointer-events: auto; 
            background: rgba(10, 10, 5, 0.95); 
            border: 1px solid #443300; 
            padding: 2rem; 
            border-radius: 8px; 
            text-align: center;
        }
        
        #glitch-overlay {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: url('https://www.transparenttextures.com/patterns/stardust.png');
            opacity: 0; pointer-events: none; mix-blend-mode: overlay; z-index: 5;
        }

        #warning-msg {
            position: absolute; bottom: 20%; left: 50%; transform: translateX(-50%);
            background: rgba(200, 0, 0, 0.9); padding: 12px 24px; border: 2px solid red;
            color: white; font-weight: bold; text-transform: uppercase; opacity: 0; 
            transition: opacity 0.2s; pointer-events: none; z-index: 100; font-size: 16px;
        }

        input[type=range] { appearance: none; background: #222; height: 4px; }
        input[type=range]::-webkit-slider-thumb { appearance: none; width: 12px; height: 12px; background: #ca8a04; border-radius: 50%; cursor: pointer; }
        
        #battery-container { width: 100px; height: 10px; background: #333; border: 1px solid #555; position: relative; overflow: hidden; margin-top: 4px; }
        #battery-bar { height: 100%; width: 100%; background: #22c55e; transition: width 0.2s, background-color 0.5s; }

        #role-badge { position: absolute; top: 20px; right: 20px; padding: 10px 20px; border-radius: 4px; font-weight: 900; text-transform: uppercase; letter-spacing: 2px; display: none; z-index: 60; }
        .is-survivor { background: #166534; color: white; border: 1px solid #22c55e; }
        .is-monster { background: #7f1d1d; color: white; border: 1px solid #ef4444; animation: pulse 1s infinite; }
        
        @keyframes pulse { 0% { opacity: 1; } 50% { opacity: 0.7; } 100% { opacity: 1; } }
    </style>
</head>
<body>

    <div id="vhs-effect"></div>
    <div id="glitch-overlay"></div>
    <div id="warning-msg">CATNAP ESTÁ PRÓXIMO...</div>
    <div id="role-badge"></div>
    
    <div id="ui-overlay" class="flex items-center justify-center">
        <div id="main-menu" class="menu-panel max-w-sm w-full mx-4 shadow-2xl">
            <h1 id="menu-title" class="text-3xl font-black text-yellow-600 mb-6 italic tracking-tighter">THE BACKROOMS 3D</h1>
            
            <div class="space-y-5 text-left">
                <div id="name-section">
                    <label class="block text-[10px] text-yellow-600/70 mb-1 uppercase font-bold">Nickname:</label>
                    <input type="text" id="nickname-input" maxlength="12" placeholder="Digite seu nome..." 
                        class="w-full bg-black/40 border border-yellow-900/30 p-2 text-yellow-500 focus:outline-none focus:border-yellow-600 text-sm uppercase">
                </div>

                <div>
                    <label class="block text-[10px] text-yellow-600/70 mb-1 uppercase font-bold">Qualidade Gráfica:</label>
                    <select id="quality-select" class="w-full bg-black/40 border border-yellow-900/30 p-2 text-yellow-500 focus:outline-none text-sm cursor-pointer">
                        <option value="low">DESEMPENHO (LO-FI)</option>
                        <option value="medium" selected>EQUILIBRADO</option>
                        <option value="high">QUALIDADE (ULTRA)</option>
                    </select>
                </div>

                <div>
                    <div class="flex justify-between items-center mb-1">
                        <label class="text-[10px] text-yellow-600/70 uppercase font-bold">Volume Global:</label>
                        <span id="volume-val" class="text-[10px] text-yellow-600">50%</span>
                    </div>
                    <input type="range" id="volume-slider" min="0" max="100" value="50" class="w-full">
                </div>
            </div>

            <div class="mt-8 space-y-2">
                <button id="start-btn" class="w-full py-3 bg-yellow-800 hover:bg-yellow-700 text-white font-bold rounded transition-all uppercase tracking-widest text-sm">Iniciar Partida</button>
                <button id="resume-btn" class="hidden w-full py-3 bg-neutral-800 hover:bg-neutral-700 text-white font-bold rounded transition-all uppercase tracking-widest text-sm">Continuar</button>
            </div>
            
            <div id="status" class="mt-4 text-[9px] opacity-40 font-mono tracking-tight text-center italic text-yellow-600">Mínimo 3 jogadores para o Modo Monstro</div>
        </div>

        <div id="hud" class="hidden absolute top-4 left-4 flex flex-col gap-2 pointer-events-none">
            <div class="bg-black/70 backdrop-blur-md px-4 py-3 rounded-lg border border-yellow-900/30">
                <div id="level-display" class="text-yellow-500 font-bold text-sm uppercase">NÍVEL 0</div>
                <div id="player-count" class="text-[10px] opacity-60 text-white">Jogadores: 1</div>
                
                <div class="mt-2">
                    <div class="flex justify-between items-center text-[9px] uppercase font-bold text-white/70">
                        <span>Lanterna [F]</span>
                        <span id="battery-text">100%</span>
                    </div>
                    <div id="battery-container">
                        <div id="battery-bar"></div>
                    </div>
                </div>
                <div id="monster-status" class="text-[9px] text-red-500 mt-2 font-bold uppercase hidden">MODO MONSTRO ATIVO</div>
            </div>
        </div>

        <div id="crosshair" class="hidden"></div>
    </div>

    <div id="status-indicator" style="position: absolute; bottom: 20px; left: 20px; font-size: 10px; color: #ca8a04; font-weight: bold; text-shadow: 1px 1px 2px black; opacity: 0.7; z-index: 40; pointer-events: none;"></div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getFirestore, doc, setDoc, onSnapshot, collection, query, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";

        const firebaseConfig = JSON.parse(__firebase_config);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'backrooms-3d-catnap-infected';
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);

        let scene, camera, renderer, clock, listener;
        let isLocked = false, gameStarted = false;
        let velocity = new THREE.Vector3(), pitch = 0, yaw = 0;
        let playerLight, headBobTimer = 0; 
        
        let flashlightOn = false;
        let flashlightBattery = 100;
        let flashlightSpotlight = null;
        const batteryDrainRate = 4.5; 
        const batteryRecoverRate = 3.0; 
        
        const cellSize = 4;
        const playerRadius = 0.35; 
        let playerPos = new THREE.Vector3(cellSize * 1.5, 1.6, cellSize * 1.5); 
        let localPlayerModel = null;
        let cameraMode = 0; 
        
        let maze = [];
        let disposableObjects = []; 
        const mazeSize = 25;
        
        let userId = null, nickname = "ANÔNIMO", characterId = -1;
        let otherPlayers = {}, monster, isHost = false;
        let currentLevel = 0, isMonsterFrozen = false, isMonsterChasing = false;
        let monsterPlayerId = null; 
        let allOnlinePlayerIds = [];

        let lastSyncTime = 0, nextStepTime = 0;
        let horrorSoundActive = false, monsterHorrorNode = null;
        
        let lastSentPos = new THREE.Vector3();
        let lastSentYaw = 0;

        const CHARACTER_TYPES = [
            { name: "Recrutado", color: 0xe5e5e5, suitColor: 0x222222 },
            { name: "Explorador", color: 0xffd700, suitColor: 0x8b4513 },
            { name: "Cientista", color: 0x00ffff, suitColor: 0x2f4f4f },
            { name: "Segurança", color: 0x1e90ff, suitColor: 0x000080 },
            { name: "Sobrevivente", color: 0x32cd32, suitColor: 0x556b2f }
        ];

        const LEVEL_THEMES = [
            { name: "O LOBBY", color: "#e2c463", fog: 0x1a1a08, wallProb: 0.14, mSpeed: 0.22, floor: "#8b7d4b" },
            { name: "TUBAGENS", color: "#666666", fog: 0x101010, wallProb: 0.18, mSpeed: 0.25, floor: "#222222" },
            { name: "VAZIO", color: "#222222", fog: 0x050505, wallProb: 0.22, mSpeed: 0.30, floor: "#111111" }
        ];

        async function safeRequestPointerLock() {
            try {
                await new Promise(resolve => setTimeout(resolve, 100));
                if (document.body.requestPointerLock) {
                    await document.body.requestPointerLock();
                }
            } catch (err) {
                console.warn("Pointer lock recusado:", err.message);
            }
        }

        function applyQualitySettings() {
            if (!renderer) return;
            const quality = document.getElementById('quality-select').value;
            switch(quality) {
                case 'low': renderer.setPixelRatio(0.6); renderer.shadowMap.enabled = false; break;
                case 'medium': renderer.setPixelRatio(1.0); renderer.shadowMap.enabled = false; break;
                case 'high': renderer.setPixelRatio(window.devicePixelRatio || 2); renderer.shadowMap.enabled = true; renderer.shadowMap.type = THREE.PCFSoftShadowMap; break;
            }
            if (gameStarted) loadLevel(currentLevel, false);
        }

        function createFloorTexture(colorStr, quality) {
            const size = (quality === 'high') ? 512 : 256;
            const canvas = document.createElement('canvas');
            canvas.width = size; canvas.height = size;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = colorStr; ctx.fillRect(0, 0, size, size);
            for (let i = 0; i < (quality === 'high' ? 80000 : 30000); i++) {
                const opacity = Math.random() * 0.15;
                ctx.fillStyle = Math.random() > 0.5 ? `rgba(0,0,0,${opacity})` : `rgba(255,255,255,${opacity})`;
                ctx.fillRect(Math.random() * size, Math.random() * size, 1, 1);
            }
            return new THREE.CanvasTexture(canvas);
        }

        function createWallTexture(colorStr, quality) {
            const size = (quality === 'high') ? 512 : 256;
            const canvas = document.createElement('canvas');
            canvas.width = size; canvas.height = size;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = colorStr; ctx.fillRect(0, 0, size, size);
            for (let i = 0; i < (quality === 'high' ? 40000 : 15000); i++) {
                ctx.fillStyle = `rgba(0,0,0,${Math.random() * 0.05})`;
                ctx.fillRect(Math.random() * size, Math.random() * size, 1, 2);
            }
            const step = 32;
            ctx.strokeStyle = 'rgba(0,0,0,0.03)';
            for(let x=0; x<size; x+=step) {
                for(let y=0; y<size; y+=step) {
                    ctx.beginPath(); ctx.arc(x + step/2, y + step/2, 2, 0, Math.PI*2); ctx.stroke();
                }
            }
            return new THREE.CanvasTexture(canvas);
        }

        function setupAudio() {
            if (listener) return;
            listener = new THREE.AudioListener();
            camera.add(listener);
            const ctx = listener.context;
            const osc = ctx.createOscillator();
            const gain = ctx.createGain();
            osc.type = 'sawtooth'; osc.frequency.value = 55; gain.gain.value = 0.015; 
            osc.connect(gain); gain.connect(listener.getInput()); osc.start();
            listener.setMasterVolume(document.getElementById('volume-slider').value / 100);
        }

        function playFootstep() {
            if (!listener) return;
            const ctx = listener.context;
            const osc = ctx.createOscillator();
            const gain = ctx.createGain();
            osc.frequency.setValueAtTime(110, ctx.currentTime);
            osc.frequency.exponentialRampToValueAtTime(30, ctx.currentTime + 0.1);
            gain.gain.setValueAtTime(0.06, ctx.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.1);
            osc.connect(gain); gain.connect(listener.getInput()); osc.start(); osc.stop(ctx.currentTime + 0.1);
        }

        function updateHorrorAudio(distance) {
            if (!listener) return;
            const ctx = listener.context;
            const maxAudioRange = 12.0; 

            if (distance < maxAudioRange && !horrorSoundActive) {
                horrorSoundActive = true;
                const oscBass = ctx.createOscillator();
                const lfo = ctx.createOscillator();
                const lfoGain = ctx.createGain();
                const mainGain = ctx.createGain();
                const filter = ctx.createBiquadFilter();
                oscBass.type = 'square';
                oscBass.frequency.value = 32; 
                lfo.type = 'sine';
                lfo.frequency.value = 6.5; 
                lfoGain.gain.value = 18; 
                filter.type = 'lowpass';
                filter.frequency.value = 85;
                filter.Q.value = 12;
                lfo.connect(lfoGain);
                lfoGain.connect(oscBass.frequency);
                oscBass.connect(filter);
                filter.connect(mainGain);
                mainGain.connect(listener.getInput());
                mainGain.gain.setValueAtTime(0, ctx.currentTime);
                mainGain.gain.linearRampToValueAtTime(0.4, ctx.currentTime + 1.0);
                oscBass.start();
                lfo.start();
                monsterHorrorNode = { oscBass, lfo, mainGain, filter };
            } 
            
            if (horrorSoundActive && monsterHorrorNode) {
                const intensity = Math.max(0, 1 - (distance / maxAudioRange));
                monsterHorrorNode.filter.frequency.setTargetAtTime(70 + (intensity * 600), ctx.currentTime, 0.1);
                monsterHorrorNode.mainGain.gain.setTargetAtTime(intensity * 0.55, ctx.currentTime, 0.1);
                monsterHorrorNode.lfo.frequency.setTargetAtTime(6.5 + (intensity * 4), ctx.currentTime, 0.2);
                if (distance > maxAudioRange + 2) {
                    monsterHorrorNode.mainGain.gain.linearRampToValueAtTime(0, ctx.currentTime + 1.0);
                    monsterHorrorNode.oscBass.stop(ctx.currentTime + 1.1);
                    monsterHorrorNode.lfo.stop(ctx.currentTime + 1.1);
                    monsterHorrorNode = null; horrorSoundActive = false;
                }
            }
        }

        async function initFirebase() {
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) { await signInWithCustomToken(auth, __initial_auth_token); } 
                else { await signInAnonymously(auth); }
                onAuthStateChanged(auth, (u) => { if (u) { userId = u.uid; startSync(); } });
            } catch (err) { console.error(err); }
        }

        function startSync() {
            const playersCol = collection(db, 'artifacts', appId, 'public', 'data', 'players');
            onSnapshot(query(playersCol), (snapshot) => {
                const ids = [];
                snapshot.forEach(docSnap => {
                    const data = docSnap.data();
                    const id = docSnap.id;
                    ids.push(id);
                    if (id !== userId && data.level === currentLevel) {
                        updateOtherPlayer(id, data);
                    }
                });
                allOnlinePlayerIds = ids.sort();
                isHost = (allOnlinePlayerIds[0] === userId);
                document.getElementById('player-count').innerText = `Jogadores: ${allOnlinePlayerIds.length}`;
                
                if (isHost && allOnlinePlayerIds.length >= 3 && !monsterPlayerId) {
                    const randomIdx = Math.floor(Math.random() * allOnlinePlayerIds.length);
                    const pickedId = allOnlinePlayerIds[randomIdx];
                    setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'monsters', `level_${currentLevel}`), { 
                        monsterPlayerId: pickedId 
                    }, { merge: true });
                }
            }, (err) => console.warn("Firestore snapshot error:", err));

            const monsterDoc = doc(db, 'artifacts', appId, 'public', 'data', 'monsters', `level_${currentLevel}`);
            onSnapshot(monsterDoc, (docSnap) => {
                if (docSnap.exists()) {
                    const data = docSnap.data();
                    monsterPlayerId = data.monsterPlayerId || null;
                    
                    if (monster) { 
                        if (!monsterPlayerId) {
                            monster.position.x = data.x || monster.position.x; 
                            monster.position.z = data.z || monster.position.z; 
                        }
                        isMonsterFrozen = data.frozen || false; 
                        isMonsterChasing = data.chasing || false;
                    }
                    updateRoleUI();
                }
            }, (err) => console.warn("Monster sync error:", err));
        }

        function updateRoleUI() {
            const badge = document.getElementById('role-badge');
            const statusText = document.getElementById('monster-status');
            
            if (monsterPlayerId) {
                badge.style.display = 'block';
                statusText.classList.remove('hidden');
                if (userId === monsterPlayerId) {
                    badge.innerText = "És o CatNap!";
                    badge.className = "is-monster";
                    if (cameraMode === 0) cameraMode = 1; 
                } else {
                    badge.innerText = "Sobrevivente";
                    badge.className = "is-survivor";
                }
            } else {
                badge.style.display = 'none';
                statusText.classList.add('hidden');
            }
        }

        function setupLocalPlayerModel() {
            if (characterId === -1) characterId = Math.floor(Math.random() * CHARACTER_TYPES.length); 
            if (localPlayerModel) scene.remove(localPlayerModel);
            localPlayerModel = createPlayerModel(characterId);
            scene.add(localPlayerModel);
            localPlayerModel.visible = (cameraMode !== 0);
        }

        async function updateMyPos() {
            if (!userId || characterId === -1 || !gameStarted) return;
            const dist = playerPos.distanceTo(lastSentPos);
            const rotDist = Math.abs(yaw - lastSentYaw);
            if (dist < 0.1 && rotDist < 0.1) return;

            lastSentPos.copy(playerPos);
            lastSentYaw = yaw;

            const myDoc = doc(db, 'artifacts', appId, 'public', 'data', 'players', userId);
            try {
                await setDoc(myDoc, { 
                    nickname, 
                    charId: characterId, 
                    x: playerPos.x, 
                    z: playerPos.z, 
                    ry: yaw, 
                    level: currentLevel, 
                    updatedAt: serverTimestamp() 
                }, { merge: true });
            } catch (e) { console.error("Update pos error:", e); }
        }

        function initThree() {
            scene = new THREE.Scene();
            camera = new THREE.PerspectiveCamera(70, window.innerWidth / window.innerHeight, 0.1, 150);
            camera.rotation.order = 'YXZ';
            renderer = new THREE.WebGLRenderer({ antialias: false, powerPreference: "high-performance" });
            renderer.setSize(window.innerWidth, window.innerHeight);
            document.body.appendChild(renderer.domElement);
            applyQualitySettings();
            scene.add(new THREE.AmbientLight(0xfff5cc, 0.4));
            playerLight = new THREE.PointLight(0xffffee, 0.3, 8); 
            scene.add(playerLight);

            flashlightSpotlight = new THREE.SpotLight(0xffffff, 0, 15, Math.PI/6, 0.3, 1);
            flashlightSpotlight.castShadow = true;
            scene.add(flashlightSpotlight);
            scene.add(flashlightSpotlight.target);

            loadLevel(0);
            clock = new THREE.Clock();
            animate();
        }

        function loadLevel(index, resetLevelData = true) {
            disposableObjects.forEach(obj => {
                scene.remove(obj);
                obj.traverse(o => { if (o.geometry) o.geometry.dispose(); if (o.material) { if (Array.isArray(o.material)) o.material.forEach(m => m.dispose()); else o.material.dispose(); if (o.material.map) o.material.map.dispose(); } });
            });
            disposableObjects = [];
            if (resetLevelData) {
                currentLevel = index; generateMazeData(); 
                playerPos.set(cellSize * 1.5, 1.6, cellSize * 1.5); 
                camera.position.copy(playerPos); velocity.set(0,0,0);
            }
            const theme = LEVEL_THEMES[currentLevel % LEVEL_THEMES.length];
            const q = document.getElementById('quality-select').value;
            scene.background = new THREE.Color(theme.fog);
            scene.fog = new THREE.FogExp2(theme.fog, (q === 'low' ? 0.28 : q === 'medium' ? 0.16 : 0.08));
            document.getElementById('level-display').innerText = `NÍVEL ${currentLevel}: ${theme.name}`;
            buildMazeScene(theme, q);
            if(monster) scene.remove(monster);
            monster = createCatNap(); 
            if (resetLevelData) monster.position.set(cellSize * 10, 0, cellSize * 10); 
            scene.add(monster);
            if(localPlayerModel) scene.add(localPlayerModel);
        }

        function generateMazeData() {
            maze = []; const theme = LEVEL_THEMES[currentLevel % LEVEL_THEMES.length];
            const exitX = mazeSize - 2; const exitY = mazeSize - 2;
            const mSX = 10; const mSY = 10;
            for(let y=0; y<mazeSize; y++) {
                maze[y] = [];
                for(let x=0; x<mazeSize; x++) {
                    const isPS = (x >= 1 && x <= 2 && y >= 1 && y <= 2);
                    const isEZ = (x >= exitX - 1 && x <= exitX && y >= exitY - 1 && y <= exitY);
                    const isEP = (x === exitX && y >= exitY - 3) || (y === exitY && x >= exitX - 3);
                    const isMS = (x >= mSX - 1 && x <= mSX + 1 && y >= mSY - 1 && y <= mSY + 1);
                    if(x === 0 || y === 0 || x === mazeSize-1 || y === mazeSize-1) { maze[y][x] = 1; } 
                    else if (isPS || isEZ || isEP || isMS) { maze[y][x] = 0; } 
                    else if (Math.random() < theme.wallProb) { maze[y][x] = 1; } 
                    else { maze[y][x] = 0; }
                }
            }
            maze[exitY][exitX] = 2; 
        }

        function buildMazeScene(theme, quality) {
            const wallTex = createWallTexture(theme.color, quality);
            const floorTex = createFloorTexture(theme.floor, quality);
            let wallsPositions = [];
            for(let y=0; y<mazeSize; y++) { for(let x=0; x<mazeSize; x++) { if(maze[y][x] === 1) wallsPositions.push({x, y}); } }
            const wallGeo = new THREE.BoxGeometry(cellSize, 3, cellSize);
            const wallMat = new THREE.MeshStandardMaterial({ map: wallTex, roughness: 1.0 });
            const instWalls = new THREE.InstancedMesh(wallGeo, wallMat, wallsPositions.length);
            const dummy = new THREE.Object3D();
            wallsPositions.forEach((p, i) => { dummy.position.set(p.x * cellSize, 1.5, p.y * cellSize); dummy.updateMatrix(); instWalls.setMatrixAt(i, dummy.matrix); });
            scene.add(instWalls); disposableObjects.push(instWalls);
            const floor = new THREE.Mesh(new THREE.PlaneGeometry(mazeSize * cellSize, mazeSize * cellSize), new THREE.MeshStandardMaterial({ map: floorTex, roughness: 1.0 }));
            floor.rotation.x = -Math.PI / 2; floor.position.set((mazeSize*cellSize)/2 - cellSize/2, 0, (mazeSize*cellSize)/2 - cellSize/2);
            floorTex.repeat.set(mazeSize, mazeSize);
            scene.add(floor); disposableObjects.push(floor);
            const ceil = new THREE.Mesh(new THREE.PlaneGeometry(mazeSize * cellSize, mazeSize * cellSize), new THREE.MeshStandardMaterial({ color: 0xeeeeee, roughness: 0.8 }));
            ceil.rotation.x = Math.PI / 2; ceil.position.set((mazeSize*cellSize)/2 - cellSize/2, 3, (mazeSize*cellSize)/2 - cellSize/2);
            scene.add(ceil); disposableObjects.push(ceil);
            const exitX = (mazeSize - 2) * cellSize; const exitZ = (mazeSize - 2) * cellSize;
            const portal = new THREE.Mesh(new THREE.PlaneGeometry(1.5, 2.5), new THREE.MeshBasicMaterial({ color: 0xffffff, transparent: true, opacity: 0.9, side: THREE.DoubleSide }));
            portal.position.set(exitX, 1.25, exitZ); scene.add(portal); disposableObjects.push(portal);
            const portalGlow = new THREE.PointLight(0x00ff00, 5, 8); portalGlow.position.set(exitX, 1.25, exitZ);
            scene.add(portalGlow); disposableObjects.push(portalGlow);
        }

        function createCatNap() {
            const group = new THREE.Group();
            const purpleMat = new THREE.MeshStandardMaterial({ color: 0x4B3A8C, roughness: 0.9 }); 
            const darkMat = new THREE.MeshStandardMaterial({ color: 0x050505 });
            const glowWhiteMat = new THREE.MeshStandardMaterial({ color: 0xffffff, emissive: 0xffffff, emissiveIntensity: 6.0 });
            const goldMat = new THREE.MeshStandardMaterial({ color: 0xFFD700, roughness: 0.1, metalness: 1.0 });

            const torso = new THREE.Mesh(new THREE.CylinderGeometry(0.14, 0.18, 0.75, 16), purpleMat);
            torso.position.y = 1.25; group.add(torso);
            const neck = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.12, 0.12, 12), purpleMat);
            neck.position.y = 0.435; torso.add(neck);
            const head = new THREE.Group();
            head.position.y = 0.1; neck.add(head);
            const headShape = new THREE.Mesh(new THREE.SphereGeometry(0.24, 24, 24), purpleMat);
            head.add(headShape);
            const earGeo = new THREE.ConeGeometry(0.1, 0.28, 4);
            const earL = new THREE.Mesh(earGeo, purpleMat); earL.position.set(-0.16, 0.25, 0); earL.rotation.z = 0.35; head.add(earL);
            const earR = new THREE.Mesh(earGeo, purpleMat); earR.position.set(0.16, 0.25, 0); earR.rotation.z = -0.35; head.add(earR);
            const eyeGeo = new THREE.SphereGeometry(0.065, 16, 16);
            const eyeL = new THREE.Mesh(eyeGeo, glowWhiteMat); eyeL.position.set(-0.11, 0.12, 0.2); head.add(eyeL);
            const eyeR = new THREE.Mesh(eyeGeo, glowWhiteMat); eyeR.position.set(0.11, 0.12, 0.2); head.add(eyeR);
            const mouthVoid = new THREE.Mesh(new THREE.SphereGeometry(0.19, 16, 16), darkMat);
            mouthVoid.scale.set(0.9, 0.65, 0.45); mouthVoid.position.set(0, -0.08, 0.15); head.add(mouthVoid);
            const collar = new THREE.Mesh(new THREE.TorusGeometry(0.1, 0.02, 8, 16), darkMat);
            collar.rotation.x = Math.PI/2; collar.position.y = -0.05; neck.add(collar);
            const moon = new THREE.Mesh(new THREE.SphereGeometry(0.06, 12, 12), goldMat);
            moon.scale.set(1, 1, 0.2); moon.position.set(0, -0.05, 0.12); collar.add(moon);
            const tailRoot = new THREE.Group();
            tailRoot.position.set(0, 0.1, -0.14); torso.add(tailRoot);
            const tail = new THREE.Mesh(new THREE.CylinderGeometry(0.035, 0.015, 0.85, 12), purpleMat);
            tail.position.y = 0.35; tail.rotation.x = -Math.PI / 1.6; tailRoot.add(tail);
            const createLimb = (side, isLeg) => {
                const root = new THREE.Group(); root.position.set(side * (isLeg ? 0.12 : 0.18), isLeg ? 0.9 : 1.6, 0);
                const sJoint = new THREE.Mesh(new THREE.SphereGeometry(0.07, 8, 8), purpleMat); root.add(sJoint);
                const upper = new THREE.Mesh(new THREE.CylinderGeometry(0.06, 0.05, 0.45), purpleMat); upper.position.y = -0.225; root.add(upper);
                const midJoint = new THREE.Group(); midJoint.position.y = -0.45; root.add(midJoint);
                const midSphere = new THREE.Mesh(new THREE.SphereGeometry(0.06, 8, 8), purpleMat); midJoint.add(midSphere);
                const lower = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.045, 0.45), purpleMat); lower.position.y = -0.225; midJoint.add(lower);
                const paw = new THREE.Mesh(new THREE.BoxGeometry(0.14, 0.08, 0.16), darkMat); paw.position.y = -0.45; midJoint.add(paw);
                return { root, mid: midJoint };
            };
            const armR = createLimb(1, false); group.add(armR.root);
            const armL = createLimb(-1, false); group.add(armL.root);
            const legR = createLimb(1, true); group.add(legR.root);
            const legL = createLimb(-1, true); group.add(legL.root);
            group.armParts = { R: armR, L: armL }; group.legParts = { R: legR, L: legL };
            group.faceParts = { eyeL, eyeR, mouth: mouthVoid, tail: tailRoot };
            group.traverse(o => { if (o.isMesh) { o.castShadow = true; o.receiveShadow = true; } });
            return group;
        }

        function updateAngelPose(monster, isFrozen, delta) {
            if (!monster || !monster.armParts) return;
            const speed = delta * 8.0;
            const armRotationX = -Math.PI / 2;

            const lerpRot = (obj, target) => {
                obj.rotation.x = THREE.MathUtils.lerp(obj.rotation.x, target.x, speed);
                obj.rotation.y = THREE.MathUtils.lerp(obj.rotation.y, target.y, speed);
                obj.rotation.z = THREE.MathUtils.lerp(obj.rotation.z, target.z, speed);
            };

            lerpRot(monster.armParts.R.root, { x: armRotationX, y: 0, z: 0.15 });
            lerpRot(monster.armParts.L.root, { x: armRotationX, y: 0, z: -0.15 });

            const t = performance.now() * 0.005;
            monster.faceParts.tail.rotation.z = Math.sin(t) * 0.45;
            monster.faceParts.tail.rotation.x = -Math.PI/1.6 + Math.cos(t * 0.4) * 0.15;

            if(!isFrozen) {
                const walkT = t * 2.8;
                monster.legParts.R.root.rotation.x = Math.sin(walkT) * 0.55;
                monster.legParts.R.mid.rotation.x = Math.abs(Math.cos(walkT)) * 0.45; 
                monster.legParts.L.root.rotation.x = Math.sin(walkT + Math.PI) * 0.55;
                monster.legParts.L.mid.rotation.x = Math.abs(Math.cos(walkT + Math.PI)) * 0.45;
            } else {
                lerpRot(monster.legParts.R.root, { x: 0, y: 0, z: 0 });
                lerpRot(monster.legParts.L.root, { x: 0, y: 0, z: 0 });
            }

            const { eyeL, eyeR, mouth } = monster.faceParts;
            if (isFrozen) {
                eyeL.scale.set(0.65, 0.25, 1); eyeR.scale.set(0.65, 0.25, 1);
                mouth.scale.set(1.0, 0.45, 1);
            } else {
                eyeL.scale.lerp(new THREE.Vector3(1.6, 1.6, 1), speed);
                eyeR.scale.lerp(new THREE.Vector3(1.6, 1.6, 1), speed);
                mouth.scale.lerp(new THREE.Vector3(1.1, 2.1, 1.3), speed);
            }
        }

        function createPlayerModel(typeIdx) {
            const type = CHARACTER_TYPES[typeIdx % CHARACTER_TYPES.length]; 
            const group = new THREE.Group();
            const suitMat = new THREE.MeshStandardMaterial({ color: type.suitColor });
            const skinMat = new THREE.MeshStandardMaterial({ color: 0xffdbac });
            const darkMat = new THREE.MeshStandardMaterial({ color: 0x111111 });
            const detailMat = new THREE.MeshStandardMaterial({ color: type.color });
            const torso = new THREE.Mesh(new THREE.BoxGeometry(0.45, 0.9, 0.3), suitMat);
            torso.position.y = 1.05; group.add(torso);
            const backpack = new THREE.Mesh(new THREE.BoxGeometry(0.35, 0.5, 0.12), detailMat);
            backpack.position.set(0, 0, -0.21); torso.add(backpack);
            const head = new THREE.Mesh(new THREE.BoxGeometry(0.4, 0.4, 0.4), skinMat);
            head.position.y = 1.75; group.add(head); group.head = head;
            const eyeGeo = new THREE.BoxGeometry(0.08, 0.08, 0.04);
            const eyeMat = new THREE.MeshStandardMaterial({ color: 0x000000 });
            const eyeL = new THREE.Mesh(eyeGeo, eyeMat); eyeL.position.set(-0.1, 0.05, 0.2); head.add(eyeL);
            const eyeR = new THREE.Mesh(eyeGeo, eyeMat); eyeR.position.set(0.1, 0.05, 0.2); head.add(eyeR);
            const mouth = new THREE.Mesh(new THREE.BoxGeometry(0.15, 0.04, 0.02), new THREE.MeshStandardMaterial({ color: 0x330000 }));
            mouth.position.set(0, -0.1, 0.2); head.add(mouth);
            const limbGeo = new THREE.BoxGeometry(0.15, 0.6, 0.15);
            const createL = (side) => {
                const g = new THREE.Group(); g.position.set(side * 0.15, 0.6, 0);
                const l = new THREE.Mesh(limbGeo, suitMat); l.position.y = -0.3; g.add(l);
                const s = new THREE.Mesh(new THREE.BoxGeometry(0.17, 0.15, 0.25), darkMat); s.position.set(0, -0.6, 0.05); g.add(s);
                return g;
            };
            group.legL = createL(-1); group.add(group.legL); group.legR = createL(1); group.add(group.legR);
            const createA = (side) => {
                const g = new THREE.Group(); g.position.set(side * 0.3, 1.5, 0);
                const a = new THREE.Mesh(limbGeo, suitMat); a.position.y = -0.3; g.add(a);
                const h = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.12, 0.12), skinMat); h.position.y = -0.6; g.add(h);
                return g;
            };
            group.armL = createA(-1); group.add(group.armL); group.armR = createA(1); group.add(group.armR);

            const flashlightBody = new THREE.Mesh(new THREE.CylinderGeometry(0.03, 0.03, 0.15), darkMat);
            flashlightBody.rotation.x = Math.PI / 2;
            flashlightBody.position.y = -0.7; 
            flashlightBody.position.z = 0.05;
            group.armR.add(flashlightBody);
            const lens = new THREE.Mesh(new THREE.CylinderGeometry(0.03, 0.03, 0.01), new THREE.MeshBasicMaterial({color: 0xaaaaaa}));
            lens.rotation.x = Math.PI / 2; lens.position.y = 0.075; flashlightBody.add(lens);

            group.traverse(o => { if (o.isMesh) { o.castShadow = true; o.receiveShadow = true; } });
            return group;
        }

        function updateOtherPlayer(id, data) {
            if (!otherPlayers[id]) { 
                const g = createPlayerModel(data.charId || 0); 
                const l = createTextLabel(data.nickname || "???"); 
                l.position.y = 2.2; g.add(l); scene.add(g); otherPlayers[id] = { mesh: g }; 
            }
            const m = otherPlayers[id].mesh;
            const isTargetMonster = (id === monsterPlayerId);
            m.visible = !isTargetMonster;

            if (!isTargetMonster) {
                m.position.lerp(new THREE.Vector3(data.x, 0, data.z), 0.2); 
                m.rotation.y = THREE.MathUtils.lerp(m.rotation.y, data.ry + Math.PI, 0.2); 
                animateModelLimbs(m, true, false); 
            } else if (monster) {
                monster.position.lerp(new THREE.Vector3(data.x, 0, data.z), 0.2);
                // CORREÇÃO: Removemos o offset de PI para alinhar a face do modelo monstro (+Z) com o alvo
                monster.rotation.y = THREE.MathUtils.lerp(monster.rotation.y, data.ry, 0.2);
            }
        }

        function createTextLabel(text) {
            const canvas = document.createElement('canvas'); const ctx = canvas.getContext('2d'); canvas.width = 128; canvas.height = 32;
            ctx.fillStyle = 'rgba(0,0,0,0.5)'; ctx.fillRect(0, 0, 128, 32); ctx.font = '14px monospace'; ctx.fillStyle = '#ca8a04'; ctx.textAlign = 'center'; ctx.fillText(text.toUpperCase(), 64, 22);
            const texture = new THREE.CanvasTexture(canvas); const sprite = new THREE.Sprite(new THREE.SpriteMaterial({ map: texture })); sprite.scale.set(1.2, 0.3, 1); return sprite;
        }

        function animateModelLimbs(m, isM, isR) {
            if (!m) return;
            const t = performance.now() * 0.006 * (isR ? 1.5 : 1.0);
            if (isM) {
                const s = Math.sin(t) * 0.6;
                if (m.legL) m.legL.rotation.x = s; if (m.legR) m.legR.rotation.x = -s;
                if (m.armL) m.armL.rotation.x = -s * 0.8; if (m.armR) m.armR.rotation.x = s * 0.8;
            } else {
                const b = Math.sin(t * 0.5) * 0.03;
                if (m.legL) m.legL.rotation.x = 0; if (m.legR) m.legR.rotation.x = 0;
                if (m.armL) m.armL.rotation.x = b; if (m.armR) m.armR.rotation.x = b;
            }
        }

        function checkCollision(pos) {
            const pts = [{ x: pos.x - playerRadius, z: pos.z - playerRadius }, { x: pos.x + playerRadius, z: pos.z - playerRadius }, { x: pos.x - playerRadius, z: pos.z + playerRadius }, { x: pos.x + playerRadius, z: pos.z + playerRadius }, { x: pos.x, z: pos.z }];
            for (let pt of pts) {
                const gx = Math.round(pt.x / cellSize); const gz = Math.round(pt.z / cellSize);
                if (gx < 0 || gx >= mazeSize || gz < 0 || gz >= mazeSize) return true; if (maze[gz] && maze[gz][gx] === 1) return true;
            }
            return false;
        }

        function checkMonsterLOS() {
            if (!monster) return false;
            const start = monster.position.clone(); const end = playerPos.clone();
            const direction = new THREE.Vector3().subVectors(end, start); const dist = direction.length(); direction.normalize();
            const precision = 0.5; const steps = Math.floor(dist / precision);
            for (let i = 1; i < steps; i++) {
                const checkPos = start.clone().addScaledVector(direction, i * precision);
                const gx = Math.round(checkPos.x / cellSize); const gz = Math.round(checkPos.z / cellSize);
                if (maze[gz] && maze[gz][gx] === 1) return false;
            }
            return true;
        }

        async function infectPlayer(targetId) {
            if (!isLocked || !userId) return;
            const monsterDoc = doc(db, 'artifacts', appId, 'public', 'data', 'monsters', `level_${currentLevel}`);
            try {
                await setDoc(monsterDoc, { monsterPlayerId: targetId }, { merge: true });
            } catch (e) { console.error("Erro ao infetar:", e); }
        }

        function animate() {
            requestAnimationFrame(animate); 
            const delta = Math.min(clock.getDelta(), 0.1);
            if(isLocked && gameStarted) {
                if (flashlightOn) {
                    flashlightBattery = Math.max(0, flashlightBattery - batteryDrainRate * delta);
                    if (flashlightBattery === 0) flashlightOn = false;
                } else {
                    flashlightBattery = Math.min(100, flashlightBattery + batteryRecoverRate * delta);
                }
                
                document.getElementById('battery-text').innerText = `${Math.floor(flashlightBattery)}%`;
                const bBar = document.getElementById('battery-bar');
                bBar.style.width = `${flashlightBattery}%`;
                
                flashlightSpotlight.intensity = flashlightOn ? 1.8 : 0;
                flashlightSpotlight.position.copy(camera.position);
                const targetPos = new THREE.Vector3();
                camera.getWorldDirection(targetPos);
                flashlightSpotlight.target.position.copy(camera.position).add(targetPos);

                velocity.x -= velocity.x * 10.0 * delta; velocity.z -= velocity.z * 10.0 * delta;
                const cDir = new THREE.Vector3(-Math.sin(yaw), 0, -Math.cos(yaw)).normalize();
                const rDir = new THREE.Vector3(Math.cos(yaw), 0, -Math.sin(yaw)).normalize();
                const keys = window.activeKeys || {}; 
                const isR = keys['shiftleft'] || keys['shiftright'];
                
                const isMeMonster = (userId === monsterPlayerId);
                const baseSpeed = isMeMonster ? 75.0 : 55.0;
                const spd = baseSpeed * delta * (isR ? 1.6 : 1.0);

                if (keys['keyw'] || keys['arrowup']) velocity.addScaledVector(cDir, spd); 
                if (keys['keys'] || keys['arrowdown']) velocity.addScaledVector(cDir, -spd);
                if (keys['keyd'] || keys['arrowright']) velocity.addScaledVector(rDir, spd); 
                if (keys['keya'] || keys['arrowleft']) velocity.addScaledVector(rDir, -spd);
                
                const nX = playerPos.clone(); nX.x += velocity.x * delta; if (!checkCollision(nX)) playerPos.x = nX.x; else velocity.x = 0;
                const nZ = playerPos.clone(); nZ.z += velocity.z * delta; if (!checkCollision(nZ)) playerPos.z = nZ.z; else velocity.z = 0;
                
                const eX = (mazeSize - 2) * cellSize; const eZ = (mazeSize - 2) * cellSize;
                if (!isMeMonster && Math.sqrt(Math.pow(playerPos.x - eX, 2) + Math.pow(playerPos.z - eZ, 2)) < 1.0) loadLevel(currentLevel + 1);
                
                const isM = velocity.length() > 0.05; 
                if (isM) headBobTimer += delta * (isR ? 15 : 10); else headBobTimer = THREE.MathUtils.lerp(headBobTimer, 0, 0.1);
                const bY = Math.sin(headBobTimer) * 0.04;
                
                if (localPlayerModel) { 
                    localPlayerModel.position.set(playerPos.x, 0, playerPos.z); 
                    localPlayerModel.rotation.y = yaw + Math.PI; 
                    localPlayerModel.visible = !isMeMonster && (cameraMode !== 0); 
                    animateModelLimbs(localPlayerModel, isM, isR);
                }

                if (cameraMode === 0) { 
                    camera.position.set(playerPos.x, playerPos.y + bY, playerPos.z); 
                    camera.rotation.set(pitch, yaw, 0, 'YXZ'); 
                } else { 
                    const d = (cameraMode === 2) ? -3.5 : 3.5; 
                    camera.position.copy(playerPos).add(new THREE.Vector3(Math.sin(yaw)*d, 0.5, Math.cos(yaw)*d)); 
                    camera.lookAt(playerPos.x, playerPos.y, playerPos.z); 
                }
                
                playerLight.position.lerp(camera.position, 0.2); 
                if (isM && performance.now() > nextStepTime) { playFootstep(); nextStepTime = performance.now() + (isR ? 300 : 450); }
                
                let d2M = 100;
                if (monster) {
                    if (isMeMonster) {
                        monster.position.set(playerPos.x, 0, playerPos.z);
                        // CORREÇÃO: O monstro local agora usa o yaw direto (frente do modelo +Z alinhada com a vista)
                        monster.rotation.y = yaw;
                        
                        for (let id in otherPlayers) {
                            const other = otherPlayers[id].mesh;
                            if (other.visible && playerPos.distanceTo(other.position) < 1.5) {
                                infectPlayer(id);
                                break;
                            }
                        }
                    } else if (!monsterPlayerId) {
                        const tP = new THREE.Vector3().subVectors(playerPos, monster.position).normalize();
                        // CORREÇÃO: IA agora usa atan2 puro para o modelo encarar o jogador
                        monster.rotation.y = Math.atan2(tP.x, tP.z); 
                    }
                    const dx = playerPos.x - monster.position.x; const dz = playerPos.z - monster.position.z; 
                    d2M = Math.sqrt(dx*dx + dz*dz); 
                }

                updateHorrorAudio(d2M);
                if (monster) updateAngelPose(monster, isMonsterFrozen && !monsterPlayerId, delta);

                if (isHost && !monsterPlayerId && monster) {
                    updateMonsterAI();
                }

                const toM = new THREE.Vector3().subVectors(monster ? monster.position : new THREE.Vector3(), playerPos).normalize();
                const camV = new THREE.Vector3(); camera.getWorldDirection(camV);
                isMonsterFrozen = (camV.dot(toM) > 0.7 && d2M < 15);
                
                const mLOS = checkMonsterLOS();
                document.getElementById('status-indicator').innerText = isMeMonster ? "CAÇA OS SOBREVIVENTES!" : (isMonsterFrozen ? "CATNAP ESTÁ A OBSERVAR..." : (mLOS ? "CATNAP ESTÁ EM MOVIMENTO!" : "À PROCURA DE TI..."));
                document.getElementById('glitch-overlay').style.opacity = Math.max(0, 1 - (d2M / 14)) * 0.6;
                
                const now = performance.now();
                if (now - lastSyncTime > 1000) { 
                    updateMyPos(); 
                    if (isHost && monster && !monsterPlayerId) {
                        try {
                            setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'monsters', `level_${currentLevel}`), { 
                                x: monster.position.x, z: monster.position.z, frozen: isMonsterFrozen, chasing: isMonsterChasing 
                            }, { merge: true });
                        } catch(e) {}
                    }
                    lastSyncTime = now; 
                }
                
                if (!isMeMonster && d2M < 1.3) { 
                    loadLevel(Math.max(0, currentLevel - 1)); 
                    document.getElementById('warning-msg').innerText = "REGRESSANDO AO NÍVEL ANTERIOR...";
                    document.getElementById('warning-msg').style.opacity = '1'; 
                    setTimeout(() => document.getElementById('warning-msg').style.opacity = '0', 2500); 
                }
            }
            renderer.render(scene, camera);
        }

        function updateMonsterAI() {
            if(!monster || isMonsterFrozen) return;
            const hL = checkMonsterLOS(); isMonsterChasing = hL; if (!hL) return; 
            const mxG = Math.round(monster.position.x / cellSize), myG = Math.round(monster.position.z / cellSize);
            const pxG = Math.round(playerPos.x / cellSize), pyG = Math.round(playerPos.z / cellSize);
            if (mxG === pxG && myG === pyG) {
                const dir = new THREE.Vector3(playerPos.x - monster.position.x, 0, playerPos.z - monster.position.z).normalize();
                monster.position.x += dir.x * 0.4; monster.position.z += dir.z * 0.4; return;
            }
            const q = [[mxG, myG, []]], v = new Set([`${mxG},${myG}`]); let path = null;
            while(q.length > 0 && q.length < 400) {
                const [cx, cy, p] = q.shift(); if(cx === pxG && cy === pyG) { path = p; break; }
                [[0,1],[0,-1],[1,0],[-1,0]].forEach(([nx, ny]) => {
                    const x = cx + nx, y = cy + ny; if(x >= 0 && x < mazeSize && y >= 0 && y < mazeSize && maze[y] && maze[y][x] !== 1 && !v.has(`${x},${y}`)) { v.add(`${x},${y}`); q.push([x, y, [...p, {x, y}]]); }
                });
            }
            if(path && path.length > 0) { 
                const n = path[0]; const dir = new THREE.Vector3(n.x * cellSize - monster.position.x, 0, n.y * cellSize - monster.position.z).normalize(); 
                monster.position.x += dir.x * LEVEL_THEMES[currentLevel % LEVEL_THEMES.length].mSpeed; monster.position.z += dir.z * LEVEL_THEMES[currentLevel % LEVEL_THEMES.length].mSpeed; 
            }
        }

        window.activeKeys = {};
        window.addEventListener('keydown', (e) => { 
            window.activeKeys[e.code.toLowerCase()] = true; 
            if (e.key === 'F5') { 
                e.preventDefault(); 
                cameraMode = (cameraMode + 1) % 3; 
            }
            if (e.code.toLowerCase() === 'keyf' && gameStarted && userId !== monsterPlayerId) {
                if (flashlightBattery > 5) {
                    flashlightOn = !flashlightOn;
                }
            }
        });
        window.addEventListener('keyup', (e) => window.activeKeys[e.code.toLowerCase()] = false);
        document.getElementById('start-btn').addEventListener('click', async () => { nickname = document.getElementById('nickname-input').value.trim() || "EXPLORADOR"; gameStarted = true; setupAudio(); setupLocalPlayerModel(); await safeRequestPointerLock(); });
        document.getElementById('resume-btn').addEventListener('click', async () => { await safeRequestPointerLock(); });
        document.getElementById('quality-select').addEventListener('change', applyQualitySettings);
        document.getElementById('volume-slider').addEventListener('input', (e) => { document.getElementById('volume-val').innerText = e.target.value + "%"; if (listener) listener.setMasterVolume(e.target.value / 100); });
        document.addEventListener('pointerlockchange', () => { isLocked = (document.pointerLockElement === document.body); document.getElementById('main-menu').style.display = isLocked ? 'none' : 'block'; document.getElementById('hud').style.display = isLocked ? 'block' : 'none'; document.getElementById('crosshair').style.display = isLocked ? 'block' : 'none'; if(gameStarted) document.getElementById('resume-btn').classList.remove('hidden'); });
        document.addEventListener('mousemove', (e) => { if(!isLocked) return; yaw -= e.movementX * 0.002; pitch = Math.max(-Math.PI/2.3, Math.min(Math.PI/2.3, pitch - e.movementY * 0.002)); });
        window.addEventListener('resize', () => { camera.aspect = window.innerWidth / window.innerHeight; camera.updateProjectionMatrix(); renderer.setSize(window.innerWidth, window.innerHeight); });
        initFirebase(); initThree();
    </script>
</body>
</html>
