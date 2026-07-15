<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Photo Marki Stamp</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
  <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700&family=Open+Sans:wght@400;700&family=Roboto:wght@400;700&display=swap" rel="stylesheet">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  
  <style>
    body { background-color: #f3f4f6; font-family: 'Open Sans', sans-serif; padding-bottom: 50px; }
    .navbar { background-color: #3f2b56; }
    .main-container { max-width: 700px; margin: 30px auto; background: white; padding: 25px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); }
    .preview-box { width: 100%; min-height: 250px; border: 2px dashed #ccc; border-radius: 8px; display: flex; align-items: center; justify-content: center; background: #fafafa; overflow: hidden; position: relative; margin-bottom: 20px; }
    #canvasPreview { max-width: 100%; height: auto; }
    .section-title { font-size: 0.9rem; font-weight: 700; color: #555; text-transform: uppercase; letter-spacing: 0.5px; border-bottom: 1px solid #eee; padding-bottom: 5px; margin-top: 20px; margin-bottom: 15px; }
    .autocomplete-items { position: absolute; border: 1px solid #d4d4d4; z-index: 99; top: 100%; left: 0; right: 0; max-height: 200px; overflow-y: auto; background-color: #fff; box-shadow: 0 4px 5px rgba(0,0,0,0.1); border-radius: 0 0 8px 8px; }
    .autocomplete-items div { padding: 10px; cursor: pointer; border-bottom: 1px solid #e1e1e1; font-size: 0.85rem; }
    .autocomplete-items div:hover { background-color: #e9e9e9; }
  </style>
</head>
<body>

  <nav class="navbar navbar-dark shadow-sm">
    <div class="container justify-content-center">
      <span class="navbar-brand mb-0 h1"><i class="fa-solid fa-stamp me-2"></i> PHOTO MARKI STAMP</span>
    </div>
  </nav>

  <div class="container">
    <div class="main-container">
      <div class="preview-box">
        <div class="text-center text-muted" id="placeholderText"><i class="fa-regular fa-image fa-3x mb-2"></i><p>Unggah Foto</p></div>
        <canvas id="canvasPreview" class="d-none"></canvas>
      </div>

      <button class="btn btn-outline-primary w-100 py-2 fw-bold mb-4" onclick="document.getElementById('inputFoto').click()">
        <i class="fa-solid fa-camera me-2"></i> AMBIL / UPLOAD FOTO
      </button>
      <input type="file" id="inputFoto" accept="image/*" class="d-none">

      <form id="formTimestamp">
        <div class="section-title">1. Subject (Nama Tempat/Proyek)</div>
        <div class="mb-3"><input type="text" id="inputSubject" class="form-control" placeholder="Contoh: Gedung A / Toko Subur" autocomplete="off"></div>

        <div class="section-title">2. Edit Waktu</div>
        <div class="row g-3">
          <div class="col-6"><input type="date" id="inputTanggal" class="form-control"></div>
          <div class="col-6"><input type="time" id="inputJam" class="form-control"></div>
        </div>

        <div class="section-title">3. Lokasi Presisi</div>
        <div class="mb-3 position-relative">
          <div class="input-group">
            <input type="text" id="inputAlamat" class="form-control" placeholder="Cari alamat..." autocomplete="off">
            <button class="btn btn-secondary" type="button" id="btnGPS"><i class="fa-solid fa-location-crosshairs"></i></button>
          </div>
          <div id="autocompleteList" class="autocomplete-items"></div>
        </div>

        <div class="section-title">4. Style Stamp</div>
        <div class="row g-3">
            <div class="col-6"><label class="small text-muted">Posisi</label><select id="selectPosisi" class="form-select"><option value="bottom-right">Kanan Bawah</option><option value="bottom-left">Kiri Bawah</option></select></div>
            <div class="col-6"><label class="small text-muted">Ukuran Teks</label><input type="range" id="sliderUkuran" class="form-range" min="15" max="60" value="25"></div>
        </div>

        <button type="button" id="btnDownload" class="btn btn-primary w-100 py-3 mt-4 fw-bold" disabled><i class="fa-solid fa-download me-2"></i> SIMPAN FOTO</button>
      </form>
    </div>
  </div>

  <script>
    document.getElementById('inputTanggal').valueAsDate = new Date();
    document.getElementById('inputJam').value = new Date().toLocaleTimeString('id-ID', {hour: '2-digit', minute:'2-digit'});

    const inputFoto = document.getElementById('inputFoto'), canvas = document.getElementById('canvasPreview'), ctx = canvas.getContext('2d'), inputAlamat = document.getElementById('inputAlamat'), inputSubject = document.getElementById('inputSubject');
    
    ['inputTanggal', 'inputJam', 'inputAlamat', 'inputSubject', 'selectPosisi', 'sliderUkuran'].forEach(id => document.getElementById(id).addEventListener('input', drawWatermark));

    inputFoto.addEventListener('change', function(e) {
      const reader = new FileReader();
      reader.onload = function(event) {
        imgObj = new Image();
        imgObj.onload = function() { canvas.width = imgObj.width; canvas.height = imgObj.height; document.getElementById('placeholderText').classList.add('d-none'); canvas.classList.remove('d-none'); document.getElementById('btnDownload').disabled = false; drawWatermark(); };
        imgObj.src = event.target.result;
      };
      reader.readAsDataURL(e.target.files[0]);
    });

    function wrapText(context, text, maxWidth) {
      const words = text.split(' '), lines = [];
      let currentLine = words[0] || '';
      for (let i = 1; i < words.length; i++) {
        if (context.measureText(currentLine + " " + words[i]).width < maxWidth) currentLine += " " + words[i];
        else { lines.push(currentLine); currentLine = words[i]; }
      }
      lines.push(currentLine); return lines;
    }

    function drawWatermark() {
      if (typeof imgObj === 'undefined' || !imgObj) return;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(imgObj, 0, 0, canvas.width, canvas.height);

      const sub = inputSubject.value.toUpperCase();
      // Format Tanggal Indonesia
      const tglObj = new Date(document.getElementById('inputTanggal').value);
      const tgl = tglObj.toLocaleDateString('id-ID', {day: '2-digit', month: 'short', year: 'numeric'}).toUpperCase();
      const jam = document.getElementById('inputJam').value;
      const alm = inputAlamat.value;
      
      const fontSize = Math.round((canvas.width / 1000) * document.getElementById('sliderUkuran').value); 
      const margin = canvas.width * 0.03, padding = fontSize * 0.6, lineSpacing = fontSize * 1.3, maxW = canvas.width * 0.45;

      ctx.font = `bold ${fontSize}px 'Montserrat'`;
      const lines = [];
      if(sub) lines.push(sub);
      lines.push(`${tgl} - ${jam} WIB`);
      if(alm) wrapText(ctx, alm, maxW).forEach(l => lines.push(l));

      let lebarMax = 0; lines.forEach(l => lebarMax = Math.max(lebarMax, ctx.measureText(l).width));
      const boxW = lebarMax + (padding * 2), boxH = (lines.length * lineSpacing) + (padding * 1.2) - (lineSpacing - fontSize);
      const pos = document.getElementById('selectPosisi').value;
      const x = pos.includes('right') ? canvas.width - boxW - margin : margin, y = pos.includes('bottom') ? canvas.height - boxH - margin : margin;

      ctx.fillStyle = 'rgba(0,0,0,0.7)';
      ctx.fillRect(x, y, boxW, boxH);
      ctx.fillStyle = '#ffffff';
      ctx.textBaseline = 'top';
      lines.forEach((line, i) => ctx.fillText(line, x + padding, y + padding + (i * lineSpacing)));
    }

    inputAlamat.addEventListener('input', function() {
      if (this.value.length < 3) return;
      fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(this.value)}&limit=3&countrycodes=id&accept-language=id`)
        .then(res => res.json())
        .then(data => {
            const list = document.getElementById('autocompleteList'); list.innerHTML = '';
            data.forEach(item => {
                const div = document.createElement('div'); div.innerText = item.display_name;
                div.onclick = () => { inputAlamat.value = item.display_name; list.innerHTML = ''; drawWatermark(); };
                list.appendChild(div);
            });
        });
    });

    document.getElementById('btnGPS').addEventListener('click', () => {
      navigator.geolocation.getCurrentPosition(pos => {
        fetch(`https://nominatim.openstreetmap.org/reverse?format=json&lat=${pos.coords.latitude}&lon=${pos.coords.longitude}&accept-language=id`)
          .then(res => res.json())
          .then(data => { inputAlamat.value = data.display_name; drawWatermark(); });
      });
    });

    document.getElementById('btnDownload').addEventListener('click', () => {
      const link = document.createElement('a');
      link.download = 'Stamp_' + Date.now() + '.png';
      link.href = canvas.toDataURL('image/png');
      link.click();
    });
  </script>
</body>
</html>
