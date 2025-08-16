<!doctype html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Quét QR → Tạo Barcode</title>

  <!-- html5-qrcode để quét QR bằng camera -->
  <script src="https://unpkg.com/html5-qrcode@2.3.8/minified/html5-qrcode.min.js"></script>
  <!-- JsBarcode để tạo barcode (SVG) -->
  <script src="https://cdn.jsdelivr.net/npm/jsbarcode@3.11.6/dist/JsBarcode.all.min.js"></script>

  <style>
    body { font-family: Arial, Helvetica, sans-serif; padding: 18px; color: #111; background:#fafafa; }
    h1 { font-size: 20px; margin-bottom: 6px; }
    .card { background: #fff; border-radius: 8px; padding: 12px; box-shadow: 0 1px 6px rgba(0,0,0,0.08); max-width:560px; margin:auto; }
    .center { text-align:center; }
    #qr-reader { margin: 8px auto; width: 100%; max-width:360px; }
    #qr-result { font-weight:600; color:#0b66ff; word-break:break-all; }
    button { padding: 10px 14px; margin:6px 6px 6px 0; border-radius:6px; border: none; background:#0b66ff; color:white; cursor:pointer; }
    button.secondary { background:#36a; opacity:0.95; }
    button.danger { background:#d33; }
    #barcode { display:block; margin: 12px auto; max-width:100%; }
    .small { font-size:13px; color:#666; margin-top:6px; }
  </style>
</head>
<body>
  <div class="card">
    <div class="center">
      <h1>📷 Quét QR → 🔖 Tạo Barcode</h1>
      <div class="small">Bấm "Mở camera" để bật camera, quét QR rồi bấm "Tạo Barcode" để hiển thị mã vạch.</div>
    </div>

    <div class="center" style="margin-top:10px;">
      <button id="btnStart">📷 Mở camera & Quét QR</button>
      <button id="btnStop" class="secondary" disabled>⏹ Dừng camera</button>
      <button id="btnGen" class="secondary" disabled>🔖 Tạo Barcode</button>
      <button id="btnDownload" class="secondary" disabled>Tải Barcode (PNG)</button>
    </div>

    <div id="qr-reader" style="display:none;"></div>

    <p style="margin-top:10px;"><strong>Kết quả QR:</strong> <span id="qr-result">— Chưa quét —</span></p>

    <div id="barcodeWrap" class="center">
      <!-- JsBarcode sẽ vẽ vào SVG này -->
      <svg id="barcode" jsbarcode-format="CODE128"></svg>
    </div>

    <p class="small center">Lưu ý: Trang cần <strong>HTTPS</strong> trên điện thoại để truy cập camera. Dùng GitHub Pages (https) hoặc localhost.</p>
  </div>

<script>
  // Biến toàn cục
  let html5QrCode = null;
  let qrData = "";
  const qrResultEl = document.getElementById("qr-result");
  const btnStart = document.getElementById("btnStart");
  const btnStop  = document.getElementById("btnStop");
  const btnGen   = document.getElementById("btnGen");
  const btnDownload = document.getElementById("btnDownload");
  const readerDiv = document.getElementById("qr-reader");
  const barcodeSvg = document.getElementById("barcode");

  // Hàm khi quét thành công
  function onScanSuccess(decodedText, decodedResult) {
    qrData = decodedText;
    qrResultEl.textContent = qrData;
    // Tự dừng camera sau khi quét 1 kết quả (để khỏi chạy tiếp)
    stopScanner();
    btnGen.disabled = false;
    btnDownload.disabled = true; // chưa có barcode tạo
  }

  function onScanError(errorMessage) {
    // có thể hiển thị log nếu muốn (hiện để trống)
    // console.log("scan error", errorMessage);
  }

  // Bật scanner
  async function startScanner() {
    // Hiển thị khu vực quét
    readerDiv.style.display = "block";
    btnStart.disabled = true;
    btnStop.disabled = false;

    if (!html5QrCode) {
      html5QrCode = new Html5Qrcode("qr-reader", /* verbose= */ false);
    }
    // Chọn camera môi trường (sau) nếu có
    const config = { fps: 10, qrbox: { width: 250, height: 250 } };

    try {
      // start sẽ yêu cầu quyền camera nếu chưa có
      await html5QrCode.start(
        { facingMode: "environment" },
        config,
        onScanSuccess,
        onScanError
      );
    } catch (err) {
      // lỗi mở camera (quyền, không có camera, HTTPS bị chặn...)
      alert("Không mở được camera: " + err);
      readerDiv.style.display = "none";
      btnStart.disabled = false;
      btnStop.disabled = true;
    }
  }

  // Dừng scanner
  async function stopScanner() {
    if (html5QrCode && html5QrCode.getState && html5QrCode.getState() !== Html5QrcodeScannerState.NOT_STARTED) {
      try {
        await html5QrCode.stop();
      } catch(e) {
        // ignore
      }
    }
    readerDiv.style.display = "none";
    btnStart.disabled = false;
    btnStop.disabled = true;
  }

  // Tạo barcode (JsBarcode vẽ sang SVG)
  function createBarcode() {
    if (!qrData) {
      alert("Chưa có dữ liệu QR để tạo Barcode!");
      return;
    }
    // Gán bằng JsBarcode
    try {
      JsBarcode("#barcode", qrData, {
        format: "CODE128",
        displayValue: true,
        fontSize: 16,
        margin: 8,
        width: 2,
        height: 80
      });
      btnDownload.disabled = false;
    } catch (e) {
      alert("Tạo barcode lỗi: " + e);
    }
  }

  // Hàm chuyển SVG sang PNG và tải về
  function downloadSvgAsPng(svgEl, filename = "barcode.png") {
    const serializer = new XMLSerializer();
    const svgStr = serializer.serializeToString(svgEl);

    // Thêm namespace nếu thiếu
    const svgBlob = new Blob([svgStr], {type: "image/svg+xml;charset=utf-8"});
    const url = URL.createObjectURL(svgBlob);

    const img = new Image();
    img.onload = function() {
      const canvas = document.createElement("canvas");
      // đặt kích thước canvas = kích thước SVG * devicePixelRatio để nét
      const rect = svgEl.getBoundingClientRect();
      const dpr = window.devicePixelRatio || 1;
      canvas.width = Math.max(300, rect.width * dpr);
      canvas.height = Math.max(100, rect.height * dpr);
      const ctx = canvas.getContext("2d");
      // trắng nền
      ctx.fillStyle = "#ffffff";
      ctx.fillRect(0,0,canvas.width,canvas.height);
      // vẽ ảnh SVG lên canvas (scale theo dpr)
      ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
      // tạo link tải
      const png = canvas.toDataURL("image/png");
      const link = document.createElement("a");
      link.href = png;
      link.download = filename;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      URL.revokeObjectURL(url);
    };
    img.onerror = function(e) {
      alert("Không thể chuyển SVG sang PNG: " + e);
      URL.revokeObjectURL(url);
    };
    img.src = url;
  }

  // Event listeners cho các nút
  btnStart.addEventListener("click", function(){
    startScanner();
  });
  btnStop.addEventListener("click", function(){
    stopScanner();
  });
  btnGen.addEventListener("click", function(){
    createBarcode();
  });
  btnDownload.addEventListener("click", function(){
    downloadSvgAsPng(barcodeSvg, "barcode.png");
  });

  // Tắt camera khi rời trang (phòng trường hợp)
  window.addEventListener("beforeunload", function(){
    if (html5QrCode) {
      try { html5QrCode.stop(); } catch(e) {}
    }
  });
</script>
</body>
</html>
