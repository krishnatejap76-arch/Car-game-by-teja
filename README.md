<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Live Camera Capture</title>
    <style>
        body { font-family: sans-serif; text-align: center; padding: 20px; }
        video { width: 100%; max-width: 500px; border-radius: 10px; background: #000; }
        canvas { display: none; } /* Hidden, used for processing the image */
        .controls { margin-top: 15px; }
        button { padding: 10px 20px; font-size: 16px; cursor: pointer; border-radius: 5px; border: none; background: #007bff; color: white; }
        #photo-preview { margin-top: 20px; border: 2px solid #ddd; width: 200px; }
    </style>
</head>
<body>

    <h2>Live Camera Access</h2>
    
    <video id="video" autoplay playsinline></video>
    
    <div class="controls">
        <button id="start-camera">Start Camera</button>
        <button id="take-photo">Capture Photo</button>
    </div>

    <canvas id="canvas" width="640" height="480"></canvas>

    <h3>Captured Image:</h3>
    <img id="photo-preview" alt="The screen capture will appear here.">

    <script>
        const video = document.getElementById('video');
        const canvas = document.getElementById('canvas');
        const photoPreview = document.getElementById('photo-preview');
        const startBtn = document.getElementById('start-camera');
        const captureBtn = document.getElementById('take-photo');

        // 1. Request Camera Access
        startBtn.addEventListener('click', async () => {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { facingMode: "user" }, // Use "environment" for back camera
                    audio: false 
                });
                video.srcObject = stream;
            } catch (err) {
                console.error("Error accessing camera: ", err);
                alert("Could not access camera. Please ensure you are using HTTPS.");
            }
        });

        // 2. Capture the Picture
        captureBtn.addEventListener('click', () => {
            const context = canvas.getContext('2d');
            // Draw the current video frame onto the canvas
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            
            // Convert canvas to a Base64 Image URL
            const dataUrl = canvas.toDataURL('image/png');
            photoPreview.src = dataUrl;
        });
    </script>
</body>
</html>
