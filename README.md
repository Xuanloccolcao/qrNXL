<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Quét QR & Tạo Barcode</title>
  <script src="https://unpkg.com/html5-qrcode"></script>
  <script src="https://cdn.jsdelivr.net/npm/jsbarcode@3.11.5/dist/JsBarcode.all.min.js"></script>
  <style>
    body { font-family: Arial, sans-serif; text-align: center; margin: 20px; }
    #reader { width: 300px; margin: auto; }
    #barcode { margin-top: 20px; }
    button { padding: 10px 20px; margin: 10px; font-size: 16px; cursor: pointer; }
  </style>
</head>
<body>
  <h2>📷 Quét QR & ➡️ Chuyển sang Barcode</h2>

  <div id="reader"></div>
  <p id="result">Kết quả: (chưa có)</p>

  <button onclick="startScan()">Mở quét QR Code</button>
  <button onclick="makeBarcode()">Chuyển thành Barcode</button>

  <svg id="barcode"></svg>

  <script>
    let qrScanner;
    let qrResult = "";

    function startScan() {
      const qrRegion = document.getElementById("reader");
      qrScanner = new Html5Qrcode("reader");
      qrScanner.start(
        { facingMode: "environment" }, // camera sau
        { fps: 10, qrbox: 250 },
        qrCodeMessage => {
          qrResult = qrCodeMessage;
          document.getElementById("result").innerText = "Kết quả: " + qrCodeMessage;
          qrScanner.stop(); // dừng quét sau khi nhận
        },
        errorMessage => {
          // có thể bỏ qua lỗi
        }
      ).catch(err => {
        alert("Không mở được camera: " + err);
      });
    }

    function makeBarcode() {
      if (qrResult === "") {
        alert("Chưa có dữ liệu QR để tạo barcode!");
        return;
      }
      JsBarcode("#barcode", qrResult, { format: "CODE128", displayValue: true });
    }
  </script>
</body>
</html>
