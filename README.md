<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema TTA - Cronograma Pro</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/flatpickr@4.6.13/dist/flatpickr.min.js"></script>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/flatpickr@4.6.13/dist/flatpickr.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { 
            --primary: #003366; --tta-red: #e31d1a; --bg: #f0f2f5; 
            --dark-bg: #121212; --dark-text: #ffffff; --dark-primary: #0056b3;
        }
        body { font-family: 'Segoe UI', Arial, sans-serif; background: var(--bg); margin: 0; padding: 20px; transition: background 0.3s; }
        body.dark { background: var(--dark-bg); color: var(--dark-text); }
        
        #area-impressao { 
            background: white; padding: 25px; border-radius: 8px; 
            border-top: 8px solid var(--primary); box-shadow: 0 5px 15px rgba(0,0,0,0.1);
            transition: background 0.3s;
        }
        body.dark #area-impressao { background: #1e1e1e; border-top-color: var(--dark-primary); }
        
        .header-layout { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
        .logo-tta { display: flex; align-items: center; gap: 15px; }
        .logo-symbol { background: var(--primary); color: white; width: 60px; height: 60px; display: flex; align-items: center; justify-content: center; font-weight: 900; font-size: 30px; border-radius: 4px; }
        .logo-main-text { color: var(--primary); font-size: 32px; font-weight: 900; line-height: 0.8; letter-spacing: -1px; }
        body.dark .logo-main-text { color: white; }
        .logo-sub-text { color: var(--tta-red); font-size: 14px; font-weight: 700; text-transform: uppercase; margin-top: 5px; display: block;}

        .title-box { text-align: right; }
        .title-box h1 { margin: 0; color: var(--primary); font-size: 20px; text-transform: uppercase; }
        body.dark .title-box h1 { color: white; }
        .separator { height: 3px; background: var(--primary); margin-bottom: 25px; }

        .toolbar { display: flex; justify-content: space-between; margin-bottom: 20px; gap: 10px; flex-wrap: wrap; }
        .search-box { padding: 12px; border: 2px solid #ddd; border-radius: 4px; width: 350px; outline: none; }
        body.dark .search-box { background: #333; color: white; border-color: #444; }
        
        .btn { padding: 10px 15px; border: none; border-radius: 4px; cursor: pointer; color: white; font-weight: bold; font-size: 11px; transition: 0.3s; }
        .btn:hover { filter: brightness(1.2); }
        .btn-add { background: #28a745; }
        .btn-img { background: #6c757d; }
        .btn-pdf { background: #dc3545; }
        .btn-csv { background: #007bff; }
        .btn-backup { background: #ffc107; color: black; }
        .btn-restore { background: #17a2b8; }
        .btn-theme { background: #6f42c1; }
        .btn-sync { background: #20c997; }

        table { width: 100%; border-collapse: collapse; background: white; margin-bottom: 20px; }
        body.dark table { background: #252525; }
        th { background: var(--primary); color: white; padding: 14px; text-align: left; font-size: 13px; }
        
        tr.linha-finalizada { background-color: #f2f2f2 !important; color: #888; text-decoration: line-through; }
        body.dark tr.linha-finalizada { background-color: #2a2a2a !important; color: #666; }
        
        td { border: 1px solid #dee2e6; padding: 8px; font-size: 14px; }
        body.dark td { border-color: #444; }
        
        .select-status { width: 100%; padding: 5px; border: none; background: transparent; font-weight: bold; cursor: pointer; }
        .status-em-processo { color: #d39e00; }
        .status-finalizado { color: #2e7d32; }
        .status-aguardando { color: #e31d1a; animation: pisca 1.5s infinite; }

        @keyframes pisca { 0% {opacity: 1;} 50% {opacity: 0.5;} 100% {opacity: 1;} }
        .del-btn { color: #dc3545; cursor: pointer; font-size: 20px; border: none; background: none; }

        .chart-container { margin-top: 20px; background: white; padding: 20px; border-radius: 8px; display: none; }
        body.dark .chart-container { background: #252525; }
        .chart-container.show { display: block; }
        
        .history { margin-top: 20px; max-height: 150px; overflow-y: auto; background: #eee; padding: 10px; font-size: 12px; }
        body.dark .history { background: #333; }
    </style>
</head>
<body>

<div id="area-impressao">
    <div class="header-layout">
        <div class="logo-tta">
            <div class="logo-symbol">T</div>
            <div>
                <span class="logo-main-text">TUPI</span>
                <span class="logo-sub-text">TUBOS E AÇOS</span>
            </div>
        </div>
        <div class="title-box">
            <h1>Cronograma de Brunimento</h1>
            <p id="data-topo" style="margin:5px 0 0; color:#666; font-size:13px;"></p>
        </div>
    </div>
    <div class="separator"></div>

    <div class="toolbar" data-html2canvas-ignore="true">
        <div class="filter-group">
            <input type="text" class="search-box" id="filtroMaterial" placeholder="🔍 Filtrar por Material..." onkeyup="filtrar()">
            <select class="search-box" style="width: 180px;" id="filtroStatus" onchange="filtrar()">
                <option value="">Todos os Status</option>
                <option value="EM PROCESSO">EM PROCESSO</option>
                <option value="AGUARDANDO MATERIAL">AGUARDANDO MATERIAL</option>
                <option value="FINALIZADO">FINALIZADO</option>
            </select>
        </div>
        <div class="btn-group">
            <button class="btn btn-add" onclick="adicionar()">+ ADICIONAR</button>
            <button class="btn btn-img" onclick="exportarImagem()">📸 PRINT</button>
            <button class="btn btn-pdf" onclick="exportarPDF()">📄 PDF</button>
            <button class="btn btn-csv" onclick="exportarCSV()">📊 CSV</button>
            <button class="btn btn-backup" onclick="backupDados()">💾 BACKUP</button>
            <button class="btn btn-restore" onclick="document.getElementById('restoreFile').click()">🔄 RESTORE</button>
            <button class="btn btn-theme" onclick="toggleTema()">🌙 TEMA</button>
            <button class="btn btn-sync" onclick="toggleGraficos()">📈 GRÁFICOS</button>
            <input type="file" id="restoreFile" style="display:none" onchange="restoreDados(event)">
        </div>
    </div>

    <div style="overflow-x:auto;">
        <table id="tabelaPrincipal">
            <thead>
                <tr>
                    <th>Vendedor</th>
                    <th>ID Venda</th>
                    <th>Cliente</th>
                    <th>Material</th>
                    <th>Comp.</th>
                    <th>Envio</th>
                    <th>Observação</th>
                    <th style="width: 180px;">Status</th>
                    <th class="no-print" data-html2canvas-ignore="true"></th>
                </tr>
            </thead>
            <tbody id="corpoTabela"></tbody>
        </table>
    </div>

    <div class="chart-container" id="graficoBox">
        <canvas id="statusChart" height="100"></canvas>
    </div>

    <div class="history" id="historico" data-html2canvas-ignore="true">
        <strong>Histórico de Atividades:</strong>
        <div id="logHistorico"></div>
    </div>
</div>

<script>
    let myChart = null;

    window.onload = function() {
        const salvos = JSON.parse(localStorage.getItem('dadosTTA')) || [];
        if (salvos.length > 0) {
            salvos.forEach(d => criarLinha(d));
        } else {
            adicionar();
        }
        document.getElementById('data-topo').innerText = new Date().toLocaleDateString('pt-BR', { weekday: 'long', day: 'numeric', month: 'long', year: 'numeric' });
        if (localStorage.getItem('tema') === 'dark') toggleTema();
    };

    function criarLinha(d = ["", "", "", "", "", "", "", "EM PROCESSO"]) {
        const tb = document.getElementById("corpoTabela");
        const tr = document.createElement("tr");
        if (d[7] === "FINALIZADO") tr.classList.add("linha-finalizada");

        tr.innerHTML = `
            <td contenteditable="true" onblur="salvarTudo()">${d[0]}</td>
            <td contenteditable="true" onblur="salvarTudo()">${d[1]}</td>
            <td contenteditable="true" onblur="salvarTudo()">${d[2]}</td>
            <td contenteditable="true" onblur="salvarTudo()">${d[3]}</td>
            <td contenteditable="true" onblur="salvarTudo()">${d[4]}</td>
            <td contenteditable="true" class="date-field" onblur="salvarTudo()">${d[5]}</td>
            <td contenteditable="true" onblur="salvarTudo()">${d[6]}</td>
            <td>
                <select class="select-status ${getStatusClass(d[7])}" onchange="atualizarStatus(this)">
                    <option value="EM PROCESSO" ${d[7]==='EM PROCESSO'?'selected':''}>EM PROCESSO</option>
                    <option value="AGUARDANDO MATERIAL" ${d[7]==='AGUARDANDO MATERIAL'?'selected':''}>AGUARDANDO MATERIAL</option>
                    <option value="FINALIZADO" ${d[7]==='FINALIZADO'?'selected':''}>FINALIZADO</option>
                </select>
            </td>
            <td class="no-print" data-html2canvas-ignore="true">
                <button class="del-btn" onclick="removerLinha(this)">×</button>
            </td>
        `;
        tb.appendChild(tr);
        flatpickr(".date-field", { dateFormat: "d/m/Y" });
    }

    function getStatusClass(status) {
        if (status === "EM PROCESSO") return "status-em-processo";
        if (status === "AGUARDANDO MATERIAL") return "status-aguardando";
        if (status === "FINALIZADO") return "status-finalizado";
        return "";
    }

    function atualizarStatus(select) {
        const tr = select.closest('tr');
        select.className = "select-status " + getStatusClass(select.value);
        if (select.value === "FINALIZADO") tr.classList.add("linha-finalizada");
        else tr.classList.remove("linha-finalizada");
        salvarTudo();
        logAtividade(`Status alterado para ${select.value}`);
    }

    function adicionar() {
        criarLinha();
        salvarTudo();
    }

    function removerLinha(btn) {
        if (confirm("Excluir este registro?")) {
            btn.closest('tr').remove();
            salvarTudo();
        }
    }

    function salvarTudo() {
        const dados = [];
        document.querySelectorAll("#corpoTabela tr").forEach(tr => {
            const cells = tr.querySelectorAll("td");
            dados.push([
                cells[0].innerText, cells[1].innerText, cells[2].innerText, 
                cells[3].innerText, cells[4].innerText, cells[5].innerText, 
                cells[6].innerText, cells[7].querySelector("select").value
            ]);
        });
        localStorage.setItem('dadosTTA', JSON.stringify(dados));
        atualizarGraficos();
    }

    function filtrar() {
        const material = document.getElementById("filtroMaterial").value.toLowerCase();
        const status = document.getElementById("filtroStatus").value;
        document.querySelectorAll("#corpoTabela tr").forEach(tr => {
            const txtMaterial = tr.cells[3].innerText.toLowerCase();
            const txtStatus = tr.querySelector("select").value;
            const matchM = txtMaterial.includes(material);
            const matchS = status === "" || txtStatus === status;
            tr.style.display = (matchM && matchS) ? "" : "none";
        });
    }

    // --- BOTÕES DE AÇÃO ---

    function toggleTema() {
        document.body.classList.toggle('dark');
        const mode = document.body.classList.contains('dark') ? 'dark' : 'light';
        localStorage.setItem('tema', mode);
    }

    function exportarImagem() {
        html2canvas(document.querySelector("#area-impressao")).then(canvas => {
            const link = document.createElement('a');
            link.download = 'cronograma_tta.png';
            link.href = canvas.toDataURL();
            link.click();
        });
    }

    async function exportarPDF() {
        const { jsPDF } = window.jspdf;
        const doc = new jsPDF('l', 'mm', 'a4');
        const element = document.querySelector("#area-impressao");
        
        await html2canvas(element, { scale: 2 }).then(canvas => {
            const imgData = canvas.toDataURL('image/png');
            const imgProps = doc.getImageProperties(imgData);
            const pdfWidth = doc.internal.pageSize.getWidth();
            const pdfHeight = (imgProps.height * pdfWidth) / imgProps.width;
            doc.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight);
            doc.save('cronograma_tta.pdf');
        });
    }

    function exportarCSV() {
        let csv = "Vendedor;ID Venda;Cliente;Material;Comp;Envio;Obs;Status\n";
        JSON.parse(localStorage.getItem('dadosTTA')).forEach(row => {
            csv += row.join(";") + "\n";
        });
        const blob = new Blob(["\ufeff" + csv], { type: 'text/csv;charset=utf-8;' });
        const link = document.createElement("a");
        link.href = URL.createObjectURL(blob);
        link.download = "cronograma.csv";
        link.click();
    }

    function backupDados() {
        const dados = localStorage.getItem('dadosTTA');
        const blob = new Blob([dados], { type: 'application/json' });
        const a = document.createElement('a');
        a.href = URL.createObjectURL(blob);
        a.download = `backup_tta_${new Date().toLocaleDateString()}.json`;
        a.click();
    }

    function restoreDados(event) {
        const reader = new FileReader();
        reader.onload = function(e) {
            localStorage.setItem('dadosTTA', e.target.result);
            location.reload();
        };
        reader.readAsText(event.target.files[0]);
    }

    function toggleGraficos() {
        const box = document.getElementById("graficoBox");
        box.classList.toggle("show");
        if (box.classList.contains("show")) atualizarGraficos();
    }

    function atualizarGraficos() {
        const dados = JSON.parse(localStorage.getItem('dadosTTA')) || [];
        const stats = { "EM PROCESSO": 0, "AGUARDANDO MATERIAL": 0, "FINALIZADO": 0 };
        dados.forEach(d => stats[d[7]]++);

        const ctx = document.getElementById('statusChart').getContext('2d');
        if (myChart) myChart.destroy();
        myChart = new Chart(ctx, {
            type: 'bar',
            data: {
                labels: Object.keys(stats),
                datasets: [{
                    label: 'Quantidade por Status',
                    data: Object.values(stats),
                    backgroundColor: ['#d39e00', '#e31d1a', '#2e7d32']
                }]
            },
            options: { responsive: true, scales: { y: { beginAtZero: true, ticks: { stepSize: 1 } } } }
        });
    }

    function logAtividade(msg) {
        const log = document.getElementById("logHistorico");
        const time = new Date().toLocaleTimeString();
        log.innerHTML = `<div>[${time}] ${msg}</div>` + log.innerHTML;
    }
</script>

</body>
</html>
