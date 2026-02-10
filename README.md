# moje-kalkulacka
co 
<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Profi Kalkulačka Složeného Úročení</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --primary: #2563eb;
            --secondary: #10b981;
            --dark: #1f2937;
            --light: #f3f4f6;
            --accent: #f59e0b;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f2f5;
            color: var(--dark);
            margin: 0;
            padding: 20px;
        }

        .container {
            max-width: 1100px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
        }

        header {
            text-align: center;
            margin-bottom: 30px;
        }

        h1 { color: var(--primary); margin-bottom: 5px; }
        p.subtitle { color: #6b7280; font-size: 0.9em; }

        .main-grid {
            display: grid;
            grid-template-columns: 1fr 2fr;
            gap: 30px;
        }

        @media (max-width: 850px) {
            .main-grid { grid-template-columns: 1fr; }
        }

        .input-section {
            background: var(--light);
            padding: 20px;
            border-radius: 10px;
        }

        .input-group {
            margin-bottom: 15px;
        }

        label {
            display: block;
            font-weight: 600;
            margin-bottom: 5px;
            font-size: 0.85em;
        }

        input, select {
            width: 100%;
            padding: 10px;
            border: 1px solid #d1d5db;
            border-radius: 6px;
            box-sizing: border-box;
            font-size: 1em;
        }

        .step-up-group {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            background: #e5e7eb;
            padding: 10px;
            border-radius: 6px;
            margin-top: 10px;
        }

        button {
            width: 100%;
            background: var(--primary);
            color: white;
            border: none;
            padding: 15px;
            border-radius: 8px;
            font-size: 1.1em;
            font-weight: bold;
            cursor: pointer;
            transition: background 0.3s;
            margin-top: 10px;
        }

        button:hover { background: #1d4ed8; }

        .results-cards {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
            gap: 15px;
            margin-bottom: 25px;
        }

        .card {
            padding: 15px;
            border-radius: 10px;
            text-align: center;
            border-bottom: 4px solid var(--primary);
            background: #fff;
            box-shadow: 0 2px 5px rgba(0,0,0,0.05);
        }

        .card.highlight { border-color: var(--secondary); }
        .card.inflation { border-color: var(--accent); }

        .card h3 { margin: 0; font-size: 0.8em; text-transform: uppercase; color: #6b7280; }
        .card .value { font-size: 1.25em; font-weight: bold; margin-top: 5px; }

        .chart-container {
            position: relative;
            height: 400px;
            width: 100%;
        }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>Investiční Kalkulačka 2026</h1>
        <p class="subtitle">Plánujte svou budoucnost s přihlédnutím k inflaci a dynamickým vkladům.</p>
    </header>

    <div class="main-grid">
        <div class="input-section">
            <div class="input-group">
                <label>Počáteční investice (Kč)</label>
                <input type="number" id="initialDeposit" value="100000" step="1000">
            </div>

            <div class="input-group">
                <label>Měsíční příspěvek (Kč)</label>
                <input type="number" id="monthlyContribution" value="5000" step="100">
            </div>

            <div class="input-group">
                <label>Doba investování (roky)</label>
                <input type="number" id="years" value="20" min="1" max="50">
            </div>

            <div class="input-group">
                <label>Očekávaný roční výnos (%)</label>
                <input type="number" id="annualReturn" value="7" step="0.1">
            </div>

            <div class="input-group">
                <label>Průměrná roční inflace (%)</label>
                <input type="number" id="inflation" value="3" step="0.1">
            </div>

            <div class="input-group">
                <label>Frekvence připisování úroků</label>
                <select id="compounding">
                    <option value="12">Měsíčně</option>
                    <option value="4">Kvartálně</option>
                    <option value="1" selected>Ročně</option>
                </select>
            </div>

            <hr>
            <label>Zvyšování vkladů (Step-up)</label>
            <div class="step-up-group">
                <div>
                    <label style="font-size: 0.7em;">Každých (let)</label>
                    <input type="number" id="stepYears" value="5" min="1">
                </div>
                <div>
                    <label style="font-size: 0.7em;">Zvýšit o (Kč)</label>
                    <input type="number" id="stepAmount" value="1000" step="100">
                </div>
            </div>

            <button onclick="calculate()">Vypočítat a vykreslit</button>
        </div>

        <div class="display-section">
            <div class="results-cards">
                <div class="card">
                    <h3>Celkem vloženo</h3>
                    <div class="value" id="resInvested">0 Kč</div>
                </div>
                <div class="card highlight">
                    <h3>Nominální hodnota</h3>
                    <div class="value" id="resNominal">0 Kč</div>
                </div>
                <div class="card inflation">
                    <h3>Reálná hodnota</h3>
                    <div class="value" id="resReal">0 Kč</div>
                </div>
            </div>

            <div class="chart-container">
                <canvas id="investmentChart"></canvas>
            </div>
        </div>
    </div>
</div>

<script>
    let myChart;

    function formatCZK(val) {
        return new Intl.NumberFormat('cs-CZ', { style: 'currency', currency: 'CZK', maximumFractionDigits: 0 }).format(val);
    }

    function calculate() {
        // Načtení hodnot
        let initial = parseFloat(document.getElementById('initialDeposit').value) || 0;
        let monthly = parseFloat(document.getElementById('monthlyContribution').value) || 0;
        const years = parseInt(document.getElementById('years').value) || 1;
        const annualRate = (parseFloat(document.getElementById('annualReturn').value) || 0) / 100;
        const inflationRate = (parseFloat(document.getElementById('inflation').value) || 0) / 100;
        const compFreq = parseInt(document.getElementById('compounding').value);
        const stepY = parseInt(document.getElementById('stepYears').value) || 0;
        const stepA = parseFloat(document.getElementById('stepAmount').value) || 0;

        let labels = [];
        let dataNominal = [];
        let dataInvested = [];
        let dataReal = [];

        let currentBalance = initial;
        let totalInvested = initial;
        let currentMonthly = monthly;

        // Iterace po měsících pro maximální přesnost
        for (let m = 0; m <= years * 12; m++) {
            
            // Každý měsíc (kromě nultého)
            if (m > 0) {
                // Navýšení příspěvku (Step-up logika)
                if (stepY > 0 && m % (stepY * 12) === 1 && m > 1) {
                    currentMonthly += stepA;
                }

                totalInvested += currentMonthly;
                currentBalance += currentMonthly;

                // Úročení podle zvolené frekvence
                // Výpočet efektivní měsíční sazby pro zjednodušení iterace
                // Pokud je compounding roční, úrok se fakticky připíše jen v měsíci 12, 24...
                if (12 / compFreq === 1 || m % (12 / compFreq) === 0) {
                    // Zjednodušený model připisování poměrné části ročního úroku
                    let periodRate = annualRate / compFreq;
                    // Úročíme celý zůstatek v daném období
                    currentBalance += (currentBalance - (currentMonthly/2)) * periodRate; 
                }
            }

            // Ukládání dat pro graf (vždy na konci roku)
            if (m % 12 === 0) {
                labels.push("Rok " + (m / 12));
                dataNominal.push(currentBalance.toFixed(2));
                dataInvested.push(totalInvested.toFixed(2));
                
                // Výpočet reálné hodnoty (očištěné o inflaci)
                let realValue = currentBalance / Math.pow(1 + inflationRate, m / 12);
                dataReal.push(realValue.toFixed(2));
            }
        }

        // Update textových výsledků
        document.getElementById('resInvested').innerText = formatCZK(totalInvested);
        document.getElementById('resNominal').innerText = formatCZK(currentBalance);
        document.getElementById('resReal').innerText = formatCZK(dataReal[dataReal.length - 1]);

        updateChart(labels, dataNominal, dataInvested, dataReal);
    }

    function updateChart(labels, nominal, invested, real) {
        const ctx = document.getElementById('investmentChart').getContext('2d');
        
        if (myChart) {
            myChart.destroy();
        }

        myChart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [
                    {
                        label: 'Nominální hodnota (Kč)',
                        data: nominal,
                        borderColor: '#2563eb',
                        backgroundColor: 'rgba(37, 99, 235, 0.1)',
                        fill: true,
                        tension: 0.3
                    },
                    {
                        label: 'Vložený kapitál (Kč)',
                        data: invested,
                        borderColor: '#9ca3af',
                        borderDash: [5, 5],
                        fill: false,
                        tension: 0
                    },
                    {
                        label: 'Reálná hodnota (očištěná o inflaci)',
                        data: real,
                        borderColor: '#f59e0b',
                        backgroundColor: 'transparent',
                        fill: false,
                        tension: 0.3
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: { position: 'bottom' },
                    tooltip: {
                        callbacks: {
                            label: function(context) {
                                return context.dataset.label + ': ' + formatCZK(context.raw);
                            }
                        }
                    }
                },
                scales: {
                    y: {
                        beginAtZero: true,
                        ticks: {
                            callback: function(value) { return value.toLocaleString() + ' Kč'; }
                        }
                    }
                }
            }
        });
    }

    // Prvotní výpočet při načtení
    window.onload = calculate;
</script>

</body>
</html>
