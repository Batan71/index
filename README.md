<!DOCTYPE html>
<html lang="sr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Protokol Loboda - Biofizički Kalkulator Masenog Fluksa</title>
    <script src="https://cloudflare.com"></script>
    <style>
        :root { --primary: #00ffcc; --bg-dark: #0a0f1d; --card-bg: #141d34; --text: #ffffff; --text-muted: #8fa0dd; --accent: #ff3366; }
        body { font-family: 'Segoe UI', sans-serif; background-color: var(--bg-dark); color: var(--text); margin: 0; padding: 20px; display: flex; justify-content: center; align-items: center; min-height: 100vh; }
        .container { max-width: 800px; width: 100%; background: var(--card-bg); padding: 30px; border-radius: 16px; border: 1px solid rgba(0, 255, 204, 0.1); }
        h1 { color: var(--primary); text-align: center; font-size: 24px; text-transform: uppercase; }
        .subtitle { text-align: center; color: var(--text-muted); font-size: 14px; margin-bottom: 30px; }
        .grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 20px; margin-bottom: 25px; }
        .form-group { display: flex; flex-direction: column; }
        label { font-size: 13px; color: var(--text-muted); margin-bottom: 8px; }
        input, select { background: rgba(10, 15, 29, 0.7); border: 1px solid rgba(143, 160, 221, 0.3); padding: 12px; border-radius: 8px; color: var(--text); }
        button { width: 100%; background: linear-gradient(135deg, #00ffcc 0%, #00b3ff 100%); color: #0a0f1d; border: none; padding: 15px; font-size: 16px; font-weight: bold; border-radius: 8px; cursor: pointer; text-transform: uppercase; }
        .results { margin-top: 35px; border-top: 1px solid rgba(143, 160, 221, 0.2); padding-top: 25px; display: none; }
        .phase-title { color: var(--primary); font-size: 16px; font-weight: bold; margin-top: 25px; border-left: 3px solid var(--primary); padding-left: 10px; }
        .data-box { background: rgba(10, 15, 29, 0.5); padding: 15px; border-radius: 8px; margin-bottom: 15px; }
        .data-row { display: flex; justify-content: space-between; margin-bottom: 8px; font-size: 14px; }
        .chronology { max-height: 200px; overflow-y: auto; font-family: monospace; background: rgba(0, 0, 0, 0.2); padding: 10px; border-radius: 6px; }
        .embedded-qr-box { margin-top: 30px; padding: 20px; background: rgba(0, 255, 204, 0.03); border: 1px dashed var(--primary); border-radius: 12px; text-align: center; }
        .qr-container { display: inline-block; padding: 10px; background: white; border-radius: 8px; margin-top: 15px; }
    </style>
</head>
<body>
<div class="container">
    <h1>Protokol Loboda</h1>
    <div class="subtitle">Simulacija zatvorenog, idealnog biološkog sistema (Klon A)</div>
    <div class="grid">
        <div class="form-group"><label for="starost">Starost subjekta (godina)</label><input type="number" id="starost" value="55"></div>
        <div class="form-group"><label for="visina">Telesna visina (cm)</label><input type="number" id="visina" value="176"></div>
        <div class="form-group"><label for="masa">Telesna masa (kg)</label><input type="number" id="masa" value="86"></div>
        <div class="form-group"><label for="pol">Pol subjekta</label><select id="pol"><option value="Muski">Muški</option><option value="Zenski">Ženski</option></select></div>
        <div class="form-group"><label for="toksicni_dug">Ekološki dug</label><select id="toksicni_dug"><option value="baza">Baza (Čist teren)</option><option value="srednje">Srednji šum</option><option value="maksimalno">Maksimalni šum</option></select></div>
    </div>
    <button onclick="izvrsiProracun()">Pokreni proračun masenog fluksa</button>
    <div id="results" class="results">
        <div class="phase-title">Faza 1: Matrica i centralna jednačina</div>
        <div class="data-box">
            <div class="data-row"><span class="label-text">Faktor Površine Tela (SF):</span><span class="value-text" id="res-sf">0.0000 m²</span></div>
            <div class="data-row"><span class="label-text">Nominalni broj kapi (N):</span><span class="value-text" id="res-n" style="color:#00ffcc;">0 kapi</span></div>
        </div>
        <div class="phase-title">Faza 2: Redistribucija masenog fluksa (15/9)</div>
        <div class="data-box">
            <div class="data-row"><span class="label-text">Dnevni jurišni volumen:</span><span class="value-text" id="res-dnevni">0 kapi</span></div>
            <div class="data-row"><span class="label-text">Noćni lipidni anker:</span><span class="value-text" id="res-nocni">0 kapi</span></div>
        </div>
        <div class="phase-title">Faza 3: Operativne recepture</div>
        <div class="data-box"><strong>OPCIJA A: Vodeni štit</strong><div id="res-hronologija" class="chronology" style="margin-top:10px;"></div></div>
        <div class="phase-title">Faza 4: Aktuarska prognoza (Ve)</div>
        <div class="data-box">
            <div class="data-row"><span class="label-text">Predviđanje BEZ halogene zaštite:</span><span class="value-text" id="res-ve-bez" style="color:#ff3366;">0.0 godina</span></div>
            <div class="data-row" style="font-size:18px; font-weight:bold;"><span class="label-text">Konačni ekstrapolirani vek (Ve):</span><span class="value-text" id="res-ve-sa" style="color:#00ffcc;">0.0 godina</span></div>
        </div>
    </div>
    <div class="embedded-qr-box">
        <div style="font-weight: bold; color: white; font-size: 15px;">tinyurl.com/protokol-loboda</div>
        <div class="qr-container" id="qrcode"></div>
    </div>
</div>
<script>
    window.onload = function() { new QRCode(document.getElementById("qrcode"), { text: "https://github.io", width: 140, height: 140 }); };
    function izvrsiProracun() {
        const Vstart = parseFloat(document.getElementById('starost').value);
        const H = parseFloat(document.getElementById('visina').value);
        const M = parseFloat(document.getElementById('masa').value);
        const pol = document.getElementById('pol').value;
        const dug = document.getElementById('toksicni_dug').value;
        const SF = 0.007184 * Math.pow(M, 0.425) * Math.pow(H, 0.725);
        let FM = 1.0; let Fto = 0.0;
        if (dug === 'srednje') { FM = 1.3; Fto = -0.10; } else if (dug === 'maksimalno') { FM = 1.6; Fto = -0.25; }
        const N = Math.round(12 * SF * FM);
        const dnevniVolumen = Math.round(N * 0.833);
        const nocniAnker = N - dnevniVolumen;
        let hronologijaHtml = ""; let preostaleKapi = dnevniVolumen;
        for (let sat = 8; sat <= 22; sat++) {
            let kapiUSatu = Math.ceil(preostaleKapi / (22 - sat + 1)); preostaleKapi -= kapiUSatu;
            hronologijaHtml += `${sat < 10 ? '0'+sat : sat}:00č — doza 50 ml (${kapiUSatu} kapi)<br>`;
        }
        const Vn = pol === 'Muski' ? 82.9 : 89.0;
        const Ve = Vstart + ((Vn - Vstart) * 6.6667 * (1 + Fto));
        document.getElementById('res-sf').innerText = `${SF.toFixed(4)} m²`;
        document.getElementById('res-n').innerText = `${N} kapi`;
        document.getElementById('res-dnevni').innerText = `${dnevniVolumen} kapi`;
        document.getElementById('res-nocni').innerText = `${nocniAnker} kapi`;
        document.getElementById('res-hronologija').innerHTML = hronologijaHtml;
        document.getElementById('res-ve-bez').innerText = `${Vn.toFixed(1)} godina`;
        document.getElementById('res-ve-sa').innerText = `${Ve.toFixed(1)} godina`;
        document.getElementById('results').style.display = 'block';
    }
</script>
</body>
</html>
