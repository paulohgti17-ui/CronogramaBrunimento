<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema TTA - Nuvem Pro</title>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>
    
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/flatpickr@4.6.13/dist/flatpickr.min.js"></script>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/flatpickr@4.6.13/dist/flatpickr.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    
    <style>
        :root { --primary: #003366; --tta-red: #e31d1a; --bg: #f0f2f5; --dark-bg: #121212; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); margin: 0; padding: 20px; transition: 0.3s; }
        body.dark { background: var(--dark-bg); color: white; }
        #area-impressao { background: white; padding: 25px; border-radius: 8px; border-top: 8px solid var(--primary); box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
        body.dark #area-impressao { background: #1e1e1e; }
        .header-layout { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
        .logo-symbol { background: var(--primary); color: white; width: 60px; height: 60px; display: flex; align-items: center; justify-content: center; font-weight: 900; font-size: 30px; border-radius: 4px; }
        .logo-main-text { color: var(--primary); font-size: 32px; font-weight: 900; }
        body.dark .logo-main-text { color: white; }
        .toolbar { display: flex; gap: 10px; flex-wrap: wrap; margin-bottom: 20px; }
        .btn { padding: 10px 15px; border: none; border-radius: 4px; cursor: pointer; color: white; font-weight: bold; font-size: 11px; }
        table { width: 100%; border-collapse: collapse; background: white; }
        body.dark table { background: #252525; }
        th { background: var(--primary); color: white; padding: 12px; text-align: left; }
        td { border: 1px solid #dee2e6; padding: 8px; }
        body.dark td { border-color: #444; }
        .status-em-processo { color: #d39e00; font-weight: bold; }
        .status-finalizado { color: #2e7d32; font-weight: bold; }
        .status-aguardando { color: #e31d1a; font-weight: bold; animation: pisca 1.5s infinite; }
        @keyframes pisca { 50% {opacity: 0.5;} }
        .loading-overlay { position: fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.5); display:none; justify-content:center; align-items:center; color:white; z-index:1000; }
    </style>
</head>
<body>

<div id="loading" class="loading-overlay">Sincronizando com a nuvem...</div>

<div id="area-impressao">
    <div class="header-layout">
        <div class="logo-tta"><div class="logo-symbol">T</div><div><span class="logo-main-text">TUPI</span></div></div>
        <div class="title-box"><h1>Cronograma em Tempo Real</h1></div>
    </div>

    <div class="toolbar" data-html2canvas-ignore="true">
        <input type="text" id="filtroMaterial" class="btn" style="color:black; width:200px" placeholder="🔍 Filtrar..." onkeyup="filtrar()">
        <button class="btn" style="background:#28a745" onclick="adicionar()">+ NOVO REGISTRO</button>
        <button class="btn" style="background:#6f42c1" onclick="toggleTema()">🌙 TEMA</button>
        <button class="btn" style="background:#dc3545" onclick="exportarPDF()">📄 PDF</button>
    </div>

    <table id="tabelaPrincipal">
        <thead>
            <tr>
                <th>Vendedor</th><th>ID Venda</th><th>Cliente</th><th>Material</th><th>Comp.</th><th>Envio</th><th>Obs</th><th>Status</th><th></th>
            </tr>
        </thead>
        <tbody id="corpoTabela"></tbody>
    </table>
</div>

<script>
    // === CONFIGURAÇÃO DO FIREBASE ===
    const firebaseConfig = {
        // COLE AQUI AS SUAS CONFIGURAÇÕES QUE COPIOU DO FIREBASE
        apiKey: "SUA_API_KEY",
        authDomain: "SEU_PROJETO.firebaseapp.com",
        databaseURL: "https://SEU_PROJETO.firebaseio.com",
        projectId: "SEU_PROJETO",
        storageBucket: "SEU_PROJETO.appspot.com",
        messagingSenderId: "ID",
        appId: "APP_ID"
    };

    // Inicializar Firebase
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database().ref('cronograma');

    // === LÓGICA DE SINCRONIZAÇÃO ===

    // Escutar mudanças no Banco de Dados (Sincronização em Tempo Real)
    db.on('value', (snapshot) => {
        const dados = snapshot.val();
        renderizarTabela(dados);
    });

    function renderizarTabela(dados) {
        const corpo = document.getElementById("corpoTabela");
        corpo.innerHTML = "";
        
        if (!dados) return;

        Object.keys(dados).forEach(id => {
            const d = dados[id];
            const tr = document.createElement("tr");
            tr.innerHTML = `
                <td contenteditable="true" onblur="editar('${id}', 0, this.innerText)">${d[0] || ''}</td>
                <td contenteditable="true" onblur="editar('${id}', 1, this.innerText)">${d[1] || ''}</td>
                <td contenteditable="true" onblur="editar('${id}', 2, this.innerText)">${d[2] || ''}</td>
                <td contenteditable="true" onblur="editar('${id}', 3, this.innerText)">${d[3] || ''}</td>
                <td contenteditable="true" onblur="editar('${id}', 4, this.innerText)">${d[4] || ''}</td>
                <td contenteditable="true" class="date-field" onblur="editar('${id}', 5, this.innerText)">${d[5] || ''}</td>
                <td contenteditable="true" onblur="editar('${id}', 6, this.innerText)">${d[6] || ''}</td>
                <td>
                    <select class="btn ${getStatusClass(d[7])}" style="color:inherit" onchange="editar('${id}', 7, this.value)">
                        <option value="EM PROCESSO" ${d[7]==='EM PROCESSO'?'selected':''}>EM PROCESSO</option>
                        <option value="AGUARDANDO MATERIAL" ${d[7]==='AGUARDANDO MATERIAL'?'selected':''}>AGUARDANDO MATERIAL</option>
                        <option value="FINALIZADO" ${d[7]==='FINALIZADO'?'selected':''}>FINALIZADO</option>
                    </select>
                </td>
                <td><button onclick="remover('${id}')" style="background:none; border:none; color:red; cursor:pointer">×</button></td>
            `;
            corpo.appendChild(tr);
        });
        flatpickr(".date-field", { dateFormat: "d/m/Y" });
    }

    function adicionar() {
        const novaRef = db.push();
        novaRef.set(["", "", "", "", "", "", "", "EM PROCESSO"]);
    }

    function editar(id, index, valor) {
        db.child(id).child(index).set(valor);
    }

    function remover(id) {
        if(confirm("Excluir registro?")) db.child(id).remove();
    }

    function getStatusClass(s) {
        if (s === "EM PROCESSO") return "status-em-processo";
        if (s === "AGUARDANDO MATERIAL") return "status-aguardando";
        if (s === "FINALIZADO") return "status-finalizado";
        return "";
    }

    function toggleTema() {
        document.body.classList.toggle('dark');
    }

    function filtrar() {
        let val = document.getElementById("filtroMaterial").value.toLowerCase();
        document.querySelectorAll("#corpoTabela tr").forEach(tr => {
            tr.style.display = tr.innerText.toLowerCase().includes(val) ? "" : "none";
        });
    }

    async function exportarPDF() {
        const { jsPDF } = window.jspdf;
        const canvas = await html2canvas(document.querySelector("#area-impressao"));
        const doc = new jsPDF('l', 'mm', 'a4');
        doc.addImage(canvas.toDataURL('image/png'), 'PNG', 10, 10, 280, 0);
        doc.save('cronograma_tta.pdf');
    }
</script>
</body>
</html>
