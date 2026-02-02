<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MY Visitor Reg</title>
    <script src="https://unpkg.com/html5-qrcode"></script>
    <style>
        body { font-family: sans-serif; padding: 20px; background: #f4f4f4; }
        .card { background: white; padding: 20px; border-radius: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        #reader { width: 100%; border-radius: 10px; overflow: hidden; margin-bottom: 20px; }
        input { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #ddd; border-radius: 5px; box-sizing: border-box; }
        button { width: 100%; background: #008cba; color: white; padding: 15px; border: none; border-radius: 5px; font-weight: bold; margin-top: 10px; }
        #photo-result { width: 100%; margin-top: 10px; border-radius: 5px; display: none; }
        .hidden { display: none; }
    </style>
</head>
<body>

<div class="card">
    <h2>Visitor Registration</h2>
    
    <div id="reader"></div>
    
    <label>Full Name (from ID)</label>
    <input type="text" id="visitor-name" placeholder="Waiting for scan...">
    
    <label>IC / Passport No.</label>
    <input type="text" id="visitor-ic" placeholder="Waiting for scan...">

    <hr>
    
    <video id="webcam" autoplay playsinline style="width:100%; display:none;"></video>
    <canvas id="canvas" class="hidden"></canvas>
    <img id="photo-result" src="">
    
    <button id="snap-btn" class="hidden">Take Photo</button>
    <button id="camera-trigger">Open Camera for Photo</button>
    <button id="submit-btn" style="background:#4CAF50; margin-top:30px;">Register Visitor</button>
</div>

<script>
    // 1. BARCODE SCANNING SETUP
    const html5QrCode = new Html5Qrcode("reader");
    const config = { fps: 10, qrbox: { width: 250, height: 150 } };

    // Start scanner (Targeting PDF417 for MyKad)
    html5QrCode.start(
        { facingMode: "environment" }, 
        config,
        (decodedText) => {
            // MyKad data is often encrypted/binary. 
            // This will show whatever raw text is stored.
            document.getElementById('visitor-ic').value = decodedText;
            html5QrCode.stop();
            document.getElementById('reader').innerHTML = "<p style='color:green'>✅ ID Scanned</p>";
        }
    );

    // 2. PHOTO CAPTURE LOGIC
    const video = document.getElementById('webcam');
    const canvas = document.getElementById('canvas');
    const photoImg = document.getElementById('photo-result');
    const cameraBtn = document.getElementById('camera-trigger');
    const snapBtn = document.getElementById('snap-btn');

    cameraBtn.addEventListener('click', async () => {
        const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" } });
        video.srcObject = stream;
        video.style.display = "block";
        snapBtn.classList.remove('hidden');
        cameraBtn.classList.add('hidden');
    });

    snapBtn.addEventListener('click', () => {
        canvas.width = video.videoWidth;
        canvas.height = video.videoHeight;
        canvas.getContext('2d').drawImage(video, 0, 0);
        
        const dataUrl = canvas.toDataURL('image/png');
        photoImg.src = dataUrl;
        photoImg.style.display = "block";
        
        // Stop camera
        video.srcObject.getTracks().forEach(track => track.stop());
        video.style.display = "none";
        snapBtn.classList.add('hidden');
    });
</script>

</body>
</html>
