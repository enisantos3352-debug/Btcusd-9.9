
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Robô BTC - Alerta de Pivô - Nivaldo</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #121212;
            color: #ffffff;
            padding: 10px;
            margin: 0;
        }
        .container {
            max-width: 100%;
            background: #1e1e1e;
            padding: 15px;
            border-radius: 8px;
            box-sizing: border-box;
        }
        h2 {
            color: #00ffcc;
            text-align: center;
            font-size: 15px;
            margin-top: 0;
        }
        .painel-topo {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 8px;
            margin-bottom: 10px;
        }
        .card-topo {
            background: #2d2d2d;
            padding: 8px;
            border-radius: 6px;
            text-align: center;
        }
        .card-topo span {
            display: block;
            font-size: 10px;
            color: #b0b0b0;
            margin-bottom: 2px;
        }
        .bloco-horarios {
            display: flex;
            flex-direction: column;
            gap: 6px;
        }
        .linha-hora {
            font-size: 12px;
            color: #fff;
        }
        .grafico-box {
            background: #2d2d2d;
            padding: 10px;
            border-radius: 6px;
            margin-bottom: 15px;
        }
        canvas {
            width: 100% !important;
            height: 210px !important;
        }
        .form-group {
            background: #2d2d2d;
            padding: 10px;
            border-radius: 6px;
            margin-bottom: 15px;
        }
        label {
            display: block;
            font-size: 11px;
            margin-bottom: 2px;
            color: #b0b0b0;
        }
        input {
            width: 100%;
            padding: 6px;
            box-sizing: border-box;
            background: #3a3a3a;
            border: 1px solid #555;
            color: #fff;
            border-radius: 4px;
            margin-bottom: 6px;
            font-size: 13px;
        }
        button {
            background: #00ffcc;
            color: #121212;
            border: none;
            padding: 8px;
            width: 100%;
            font-weight: bold;
            cursor: pointer;
            border-radius: 4px;
            font-size: 13px;
        }
        .btn-limpar {
            background: #ff4444;
            color: #fff;
            margin-top: 8px;
        }
        table {
            width: 100%;
            margin-top: 5px;
            border-collapse: collapse;
        }
        th, td {
            padding: 6px;
            text-align: center;
            border-bottom: 1px solid #444;
            font-size: 11px;
        }
        th {
            color: #00ffcc;
        }
        .status-alvo {
            background: #252525;
            padding: 10px;
            border-radius: 6px;
            border-left: 4px solid #ffaa00;
            margin-bottom: 15px;
            font-size: 12px;
            transition: all 0.3s ease;
        }
        .alerta-ativo {
            background: #3a1c1c !important;
            border-left: 4px solid #ff3333 !important;
            animation: piscarAlerta 1s infinite alternate;
        }
        @keyframes piscarAlerta {
            from { opacity: 0.8; }
            to { opacity: 1; transform: scale(1.01); }
        }
    </style>
</head>
<body>

<div class="container">
    <h2>Robô BTC - Alerta de Pivô - Nivaldo</h2>

    <!-- HORÁRIOS EMPILHADOS NO TOPO -->
    <div class="painel-topo">
        <div class="card-topo">
            <span>Relógios (Local / Binance)</span>
            <div class="bloco-horarios">
                <div class="linha-hora">Local: <strong id="visorHoraLocal" style="color: #00ffcc;">--:--:--</strong></div>
                <div class="linha-hora">Binance: <strong id="visorHoraBase" style="color: #ffaa00;">--:--:--</strong></div>
            </div>
        </div>
        <div class="card-topo">
            <span>Pivô & Preço Atual</span>
            <div class="bloco-horarios">
                <div class="linha-hora">Pivô: <strong id="visorPivo" style="color: #ff4444;">76638.00</strong></div>
                <div class="linha-hora">Preço: <strong id="visorPrecoAtual" style="color: #00ffcc;">Conectando...</strong></div>
            </div>
        </div>
    </div>
    
    <div class="grafico-box">
        <canvas id="graficoPreco"></canvas>
    </div>

    <!-- CONFIGURAÇÃO E ALERTA INTELIGENTE -->
    <div class="form-group">
        <span style="color: #00ffcc; font-weight: bold; display: block; margin-bottom: 8px; font-size: 13px;">🎯 Configuração do Pivô e Alvo (0.01):</span>
        <label>Valor do Pivô (Base Esquerda):</label>
        <input type="number" step="0.01" id="inputPivoBase" value="76638.00" oninput="atualizarPivoManual()">
        
        <label>Distância do Gatilho de Aproximação (Pontos):</label>
        <input type="number" id="inputPontosAlvo" value="300" oninput="recalcularAlvo()">

        <div class="status-alvo" id="visorStatusAlvo">
            Analisando ponto pivô e mercado...
        </div>
    </div>

    <!-- HISTÓRICO AUTOMÁTICO -->
    <div class="form-group">
        <span style="color: #00ffcc; font-weight: bold; display: block; margin-bottom: 5px; font-size: 12px;">📊 Histórico de Ordens Automáticas:</span>
        <div style="max-height: 140px; overflow-y: auto;">
            <table id="tabelaHistorico">
                <thead>
                    <tr>
                        <th>Hora</th>
                        <th>Preço Compra</th>
                        <th>Alvo Venda</th>
                        <th>Lucro Est.</th>
                    </tr>
                </thead>
                <tbody></tbody>
            </table>
        </div>
        <button class="btn-limpar" onclick="limparTudo()">Apagar Histórico</button>
    </div>
</div>

<script>
let meuGrafico;
let precoAoVivoGlobal = 0;
let pivoGlobal = 76638.00;
let precoAlvoCompraGlobal = 0;
let ultimaCompraRegistradaPreço = 0;
let ultimoAlertaSonoroTempo = 0;

window.onload = function() {
    criarGraficoInicial();
    carregarDados();
    buscarDadosBinance();
    
    setInterval(buscarDadosBinance, 3000);
    setInterval(atualizarRelogiosEEixo, 1000);
};

function emitirBipeAlerta() {
    let agoraTempo = Date.now();
    if (agoraTempo - ultimoAlertaSonoroTempo < 15000) return; // Evita disparar o som sem parar (a cada 15 seg)
    ultimoAlertaSonoroTempo = agoraTempo;

    try {
        let audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        let osc = audioCtx.createOscillator();
        let gain = audioCtx.createGain();
        osc.type = 'sine';
        osc.frequency.setValueAtTime(880, audioCtx.currentTime); // Tom agudo de alerta
        gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.start();
        osc.stop(audioCtx.currentTime + 0.3);
    } catch (e) {
        // Navegador bloqueia áudio sem interação prévia se necessário
    }
}

function obterHoraLocal(date) {
    return date.toLocaleTimeString('pt-BR', { timeZone: 'America/Fortaleza', hour: '2-digit', minute: '2-digit', second: '2-digit' });
}

function obterHoraBaseUTC(date) {
    let h = String(date.getUTCHours()).padStart(2, '0');
    let m = String(date.getUTCMinutes()).padStart(2, '0');
    let s = String(date.getUTCSeconds()).padStart(2, '0');
    return `${h}:${m}:${s}`;
}

function gerarLabelsEixo11Min() {
    let agora = new Date();
    let labels = [];
    for (let i = -4; i <= 0; i++) {
        let tempoBloco = new Date(agora.getTime() + (i * 11 * 60 * 1000));
        let horaLoc = tempoBloco.toLocaleTimeString('pt-BR', { timeZone: 'America/Fortaleza', hour: '2-digit', minute: '2-digit' });
        let hUtc = String(tempoBloco.getUTCHours()).padStart(2, '0');
        let mUtc = String(tempoBloco.getUTCMinutes()).padStart(2, '0');
        let horaBin = `${hUtc}:${mUtc}`;
        labels.push([horaLoc, horaBin]);
    }
    labels.push(["Ao Vivo", "Binance"]);
    return labels;
}

function atualizarRelogiosEEixo() {
    let agora = new Date();
    document.getElementById('visorHoraLocal').innerText = obterHoraLocal(agora);
    document.getElementById('visorHoraBase').innerText = obterHoraBaseUTC(agora);

    if (meuGrafico) {
        meuGrafico.data.labels = gerarLabelsEixo11Min();
        meuGrafico.update('none');
    }
}

function criarGraficoInicial() {
    const ctx = document.getElementById('graficoPreco').getContext('2d');
    meuGrafico = new Chart(ctx, {
        type: 'line',
        data: {
            labels: gerarLabelsEixo11Min(),
            datasets: [{
                label: 'Eixo Pivô 76638',
                data: [0, 0, 0, 0, 0, 0],
                borderColor: '#00ffcc',
                backgroundColor: 'rgba(0, 255, 204, 0.1)',
                borderWidth: 2,
                fill: true,
                tension: 0.2
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: { display: false } },
            scales: {
                x: { 
                    ticks: { color: '#00ffcc', font: { size: 10, weight: 'bold' } }, 
                    grid: { color: '#333' } 
                },
                y: { 
                    ticks: { color: '#b0b0b0', font: { size: 9 } }, 
                    grid: { color: '#333' } 
                }
            }
        }
    });
}

function buscarDadosBinance() {
    fetch('https://api.binance.com/api/v3/ticker/price?symbol=BTCUSDT')
        .then(response => response.json())
        .then(data => {
            let precoReal = parseFloat(data.price);
            precoAoVivoGlobal = precoReal;

            document.getElementById('visorPrecoAtual').innerText = precoReal.toFixed(2);

            if (meuGrafico) {
                let p1 = pivoGlobal;
                let p2 = pivoGlobal + (precoReal - pivoGlobal) * 0.25;
                let p3 = pivoGlobal + (precoReal - pivoGlobal) * 0.50;
                let p4 = pivoGlobal + (precoReal - pivoGlobal) * 0.75;
                let p5 = precoReal;

                meuGrafico.data.datasets[0].data = [p1, p2, p3, p4, p5, precoReal];
                meuGrafico.update('none');
            }

            verificarDisparoAutomatico();
        })
        .catch(error => {
            console.error('Erro de Conexão:', error);
            document.getElementById('visorPrecoAtual').innerText = "Erro de Conexão";
        });
}

function atualizarPivoManual() {
    pivoGlobal = parseFloat(document.getElementById('inputPivoBase').value) || 76638.00;
    document.getElementById('visorPivo').innerText = pivoGlobal.toFixed(2);
    recalcularAlvo();
}

function recalcularAlvo() {
    verificarDisparoAutomatico();
}

function verificarDisparoAutomatico() {
    if (precoAoVivoGlobal === 0) return;

    let pontosDesejados = parseFloat(document.getElementById('inputPontosAlvo').value) || 300;
    precoAlvoCompraGlobal = pivoGlobal + pontosDesejados;
    let precoAlvoVenda = precoAlvoCompraGlobal + pontosDesejados;
    let lucroEstimado = pontosDesejados * 0.01;

    let boxStatus = document.getElementById('visorStatusAlvo');
    let diferencaPreco = precoAoVivoGlobal - precoAlvoCompraGlobal;
    let margemDeAproximacao = 200;

    if (Math.abs(diferencaPreco) <= margemDeAproximacao) {
        boxStatus.className = "status-alvo alerta-ativo";
        boxStatus.innerHTML = `
            🚨 <strong>ATENÇÃO NIVALDO: HORA DE ABRIR OPERAÇÃO!</strong><br>
            • Opa, chegamos perto do preço de gatilho: <strong style="color: #ff3333;">${precoAlvoCompraGlobal.toFixed(2)}</strong>!<br>
            • Preço Atual na Binance: <strong style="color: #ffaa00;">${precoAoVivoGlobal.toFixed(2)}</strong><br>
            • Prepare a primeira ordem!
        `;
        emitirBipeAlerta();
    } else {
        boxStatus.className = "status-alvo";
        boxStatus.innerHTML = `
            ⚡ <strong>Status: Monitorando Pivô (${pivoGlobal.toFixed(2)})</strong><br>
            • Preço Atual: <strong style="color: #ffaa00;">${precoAoVivoGlobal.toFixed(2)}</strong><br>
            • Gatilho de Alerta / Compra: <strong style="color: #00ffcc;">${precoAlvoCompraGlobal.toFixed(2)}</strong><br>
            • Lucro Estimado (0.01): <strong style="color: #00ffcc;">+$${lucroEstimado.toFixed(2)} USD</strong>
        `;
    }

    if (precoAoVivoGlobal <= precoAlvoCompraGlobal && Math.abs(precoAoVivoGlobal - ultimaCompraRegistradaPreço) > 50) {
        salvarOrdemAutomatica(precoAoVivoGlobal, precoAlvoVenda, lucroEstimado);
        ultimaCompraRegistradaPreço = precoAoVivoGlobal;
    }
}

function salvarOrdemAutomatica(precoCompra, precoVenda, lucro) {
    let agora = new Date();
    let novaOp = {
        hora: obterHoraLocal(agora),
        compra: precoCompra.toFixed(2),
        venda: precoVenda.toFixed(2),
        lucro: lucro.toFixed(2)
    };

    let historico = JSON.parse(localStorage.getItem('historicoPivoNivaldo')) || [];
    historico.unshift(novaOp);
    localStorage.setItem('historicoPivoNivaldo', JSON.stringify(historico));

    atualizarTabela();
}

function atualizarTabela() {
    let tbody = document.getElementById('tabelaHistorico').getElementsByTagName('tbody')[0];
    tbody.innerHTML = "";

    let historico = JSON.parse(localStorage.getItem('historicoPivoNivaldo')) || [];

    if (historico.length === 0) {
        tbody.innerHTML = "<tr><td colspan='4' style='color: #888;'>Nenhuma ordem disparada ainda.</td></tr>";
        return;
    }

    historico.forEach(op => {
        let linha = tbody.insertRow();
        linha.insertCell(0).innerText = op.hora;
        linha.insertCell(1).innerText = `$${op.compra}`;
        linha.insertCell(2).innerText = `$${op.venda}`;
        linha.insertCell(3).innerHTML = `<span style='color: #00ffcc;'>+$${op.lucro}</span>`;
    });
}

function carregarDados() {
    atualizarTabela();
}

function limparTudo() {
    if (confirm("Deseja apagar o histórico de ordens?")) {
        localStorage.removeItem('historicoPivoNivaldo');
        ultimaCompraRegistradaPreço = 0;
        atualizarTabela();
    }
}
</script>

</body>
</html>
