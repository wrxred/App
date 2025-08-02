<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Detección de Distorsión</title>
  <meta name="theme-color" content="#4CAF50">
  <style>
    body { text-align: center; font-family: Arial; background: #f4f4f4; }
    h1 { margin-top: 20px; }
    #video { width: 90%; max-width: 400px; margin-top: 20px; border: 3px solid #333; border-radius: 10px; }
    #status { font-size: 40px; font-weight: bold; margin-top: 20px; }
    .pass { color: green; }
    .fail { color: red; }
    input { margin-top: 15px; }
  </style>
</head>
<body>
  <h1>Prueba de Distorsión</h1>
  <p>Selecciona la imagen original y apunta la cámara hacia ella.</p>
  <input type="file" id="reference" accept="image/*"><br>
  <video id="video" autoplay playsinline></video>
  <div id="status">Esperando referencia...</div>

  <!-- Librería SSIM -->
  <script src="https://cdn.jsdelivr.net/npm/ssim.js/dist/ssim.min.js"></script>
  <script>
    const video = document.getElementById('video');
    const statusDiv = document.getElementById('status');
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

      // Ajustar tamaño para comparación
      canvasVideo.width = 200;
      canvasVideo.height = 200;
      canvasRef.width = 200;
      canvasRef.height = 200;

      ctxVideo.drawImage(video, 0, 0, 200, 200);
      ctxRef.drawImage(refImg, 0, 0, 200, 200);

      const imgDataVideo = ctxVideo.getImageData(0, 0, 200, 200);
      const imgDataRef = ctxRef.getImageData(0, 0, 200, 200);

      const { mssim } = ssim(imgDataVideo, imgDataRef);
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
