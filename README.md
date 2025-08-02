<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Detector de Distorsión</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f7f7f7;
      margin: 0;
      padding: 20px;
      text-align: center;
      color: #333;
    }
    h1 {
      margin-top: 10px;
      font-size: 22px;
    }
    #video {
      width: 95%;
      max-width: 400px;
      margin: 15px auto;
      border: 2px solid #555;
      border-radius: 8px;
      display: block;
    }
    #status {
      font-size: 28px;
      font-weight: bold;
      margin-top: 15px;
    }
    .pass { color: green; }
    .fail { color: red; }
    .score {
      font-size: 18px;
      margin-top: 5px;
      color: #555;
    }
    .file-input {
      margin-top: 20px;
      display: block;
    }
    .instructions {
      font-size: 14px;
      margin-top: 10px;
      color: #666;
    }
  </style>
</head>
<body>
  <h1>Detector de Distorsión</h1>
  <p class="instructions">1. Sube la imagen original.<br>2. Apunta la cámara hacia la imagen.<br>3. Verifica si pasa la prueba.</p>

  <input class="file-input" type="file" id="reference" accept="image/*">
  <video id="video" autoplay playsinline></video>
  <div id="status">Esperando referencia...</div>
  <div class="score" id="score">Similitud: --%</div>

  <!-- Librería SSIM -->
  <script src="https://cdn.jsdelivr.net/npm/ssim.js/dist/ssim.min.js"></script>
  <script>
    const video = document.getElementById('video');
    const statusDiv = document.getElementById('status');
    const scoreDiv = document.getElementById('score');
    let refImg = null;

    // Iniciar cámara
    navigator.mediaDevices.getUserMedia({ video: true }).then(stream => {
      video.srcObject = stream;
    }).catch(err => {
      alert("Error al acceder a la cámara: " + err);
    });

    // Cargar imagen de referencia
    document.getElementById('reference').addEventListener('change', e => {
      const file = e.target.files[0];
      if (!file) return;
      const img = new Image();
      img.onload = () => {
        refImg = img;
        statusDiv.textContent = "Referencia cargada. Analizando...";
        statusDiv.className = "";
      };
      img.src = URL.createObjectURL(file);
    });

    // Analizar frames cada 500ms
    setInterval(() => {
      if (!refImg) return;

      const canvasVideo = document.createElement('canvas');
      const canvasRef = document.createElement('canvas');
      const ctxVideo = canvasVideo.getContext('2d');
      const ctxRef = canvasRef.getContext('2d');

      // Ajustar tamaño fijo para comparación
      const size = 150;
      canvasVideo.width = size;
      canvasVideo.height = size;
      canvasRef.width = size;
      canvasRef.height = size;

      ctxVideo.drawImage(video, 0, 0, size, size);
      ctxRef.drawImage(refImg, 0, 0, size, size);

      const imgDataVideo = ctxVideo.getImageData(0, 0, size, size);
      const imgDataRef = ctxRef.getImageData(0, 0, size, size);

      const { mssim } = ssim(imgDataVideo, imgDataRef);
      const similarity = (mssim * 100).toFixed(2);

      scoreDiv.textContent = `Similitud: ${similarity}%`;

      if (mssim >= 0.95) {
        statusDiv.textContent = "PASS ✅";
        statusDiv.className = "pass";
      } else {
        statusDiv.textContent = "FAIL ❌";
        statusDiv.className = "fail";
      }
    }, 500);
  </script>
</body>
</html>
