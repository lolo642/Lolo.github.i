# Lolo.github.i<!DOCTYPE html>
<html lang="sv">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>LagerSkann – Barcode Scanner</title>
  <script src="https://unpkg.com/@zxing/library@latest"></script>
  <link rel="manifest" href="manifest.json" />
  <meta name="theme-color" content="#e60000" />
  <style>
    :root { --primary: #e60000; --dark: #111; }
    body {
      margin: 0; font-family: Arial, sans-serif;
      background-color: var(--dark); color: white;
    }
    header {
      background-color: var(--primary);
      padding: 1rem; text-align: center; font-size: 1.5rem;
    }
    .container { padding: 1rem; max-width: 900px; margin: auto; }
    video {
      width: 100%; max-height: 300px;
      border: 2px solid var(--primary); border-radius: 8px;
    }
    .product {
      background-color: #2a2a2a; padding: 0.5rem;
      margin-top: 0.5rem; border-radius: 5px;
      display: flex; justify-content: space-between; align-items: center;
    }
    button {
      background-color: var(--primary); border: none; color: white;
      padding: 0.4rem 0.8rem; margin: 0.2rem; border-radius: 5px; cursor: pointer;
    }
    button:hover { background-color: #cc0000; }
    @media (max-width: 600px) { header { font-size: 1.2rem; } }
  </style>
</head>
<body>

  <header>LagerSkann</header>

  <div class="container">
    <h2>📷 Skanna med Kamera</h2>
    <video id="video" muted autoplay playsinline></video>
    <p id="scan-result">🔍 Väntar på skanning...</p>

    <h2>📦 Produkter</h2>
    <div id="product-list"></div>

    <h2>🕒 Logg</h2>
    <div id="log-list"></div>
  </div>

  <script>
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('service-worker.js')
      .then(() => console.log('✅ Service Worker registrerad'))
      .catch(err => console.error('❌ Service Worker misslyckades:', err));
    }

    const codeReader = new ZXing.BrowserMultiFormatReader();
    const videoElement = document.getElementById("video");
    const scanResult = document.getElementById("scan-result");
    const products = [];

    codeReader
      .listVideoInputDevices()
      .then(videoInputDevices => {
        const firstDeviceId = videoInputDevices[0]?.deviceId;
        if (firstDeviceId) {
          codeReader.decodeFromVideoDevice(firstDeviceId, videoElement, (result, err) => {
            if (result) {
              const code = result.text;
              if (!products.find(p => p.id === code)) {
                addProduct(code);
              }
              scanResult.textContent = `✅ Skannad: ${code}`;
            }
          });
        } else {
          scanResult.textContent = "❌ Ingen kamera hittades.";
        }
      })
      .catch(err => {
        console.error(err);
        scanResult.textContent = "❌ Fel vid åtkomst till kamera.";
      });

    function addProduct(code) {
      const time = new Date().toLocaleTimeString();
      const product = {
        id: code,
        name: "Produkt " + code,
        scannedAt: time,
        soldAt: null,
        quantity: 1
      };
      products.push(product);
      updateUI();
    }

    function updateUI() {
      const list = document.getElementById("product-list");
      list.innerHTML = "";
      products.forEach((p, index) => {
        const div = document.createElement("div");
        div.className = "product";
        div.innerHTML = `
          <div>
            <strong>${p.name}</strong><br/>
            ⏱️ Inskannad: ${p.scannedAt}<br/>
            ${p.soldAt ? "💰 Såld: " + p.soldAt : ""}
          </div>
          <div>
            <button onclick="changeQty(${index}, -1)">-</button>
            ${p.quantity}
            <button onclick="changeQty(${index}, 1)">+</button><br/>
            <button onclick="markAsSold(${index})">Sälj</button>
          </div>
        `;
        list.appendChild(div);
      });

      const log = document.getElementById("log-list");
      log.innerHTML = "";
      products
        .filter(p => p.soldAt)
        .forEach(p => {
          const div = document.createElement("div");
          div.className = "product";
          div.innerHTML = `
            <div>
              ${p.name} - Antal: ${p.quantity}<br/>
              Såld: ${p.soldAt}
            </div>
          `;
          log.appendChild(div);
        });
    }

    function changeQty(index, delta) {
      products[index].quantity += delta;
      if (products[index].quantity < 1) products[index].quantity = 1;
      updateUI();
    }

    function markAsSold(index) {
      products[index].soldAt = new Date().toLocaleTimeString();
      updateUI();
    }
  </script>

</body>
</html>
