# qrNXL
NGÔ XUÂN LỘC
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>QR to Barcode</title>
  <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
  <script src="https://cdn.jsdelivr.net/npm/jsbarcode@3.11.6/dist/JsBarcode.all.min.js"></script>
</head>
<body style="font-family: Arial; text-align: center; margin: 20px;">

  <h2>Quét QR → Tạo Barcode</h2>

  <!-- Nút bấm để quét QR -->
  <button onclick="startScan()">📷 Quét QR</button>
  <div id="qr-reader" style="width:300px; margin:auto; display:none;"></div>

  <p>Kết quả QR: <span id="qr-result" style="font-weight:bold; color:blue;"></span></p>

  <!-- Nút tạo barcode -->
  <button onclick="makeBarcode()">Tạo Barcode</button>
  <br><br>
  <svg id="barcode"></svg>

  <script>
    let qrData = ""; // Biến lưu dữ liệu QR
    let html5QrCode;

    // Hàm bật quét QR khi nhấn nút
    function startScan() {
      document.getElementById("qr-reader").style.display = "block"; // hiện khung quét
      if (!html5QrCode) {
        html5QrCode = new Html5Qrcode("qr-reader");
      }
      html5QrCode.start(
        { facingMode: "environment" }, // camera sau
        { fps: 10, qrbox: 250 },
        onScanSuccess
      ).catch(err => {
        alert("Không mở được camera: " + err);
      });
    }

    // Khi quét thành công
    function onScanSuccess(decodedText) {
      qrData = decodedText;
      document.getElementById("qr-result").innerText = qrData;
      html5QrCode.stop(); // dừng camera sau khi quét thành công
    }

    // Hàm tạo barcode
    function makeBarcode() {
      if (qrData) {
        JsBarcode("#barcode", qrData, {
          format: "CODE128",
          displayValue: true,
          fontSize: 18
        });
      } else {
        alert("Chưa quét được dữ liệu QR!");
      }
    }
  </script>
</body>
</html>
