<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>CryptoWatch</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #0b1120;
    color: #e5e7eb;
}

header {
    padding: 20px;
    text-align: center;
    background: #020617;
}

input {
    width: 100%;
    padding: 12px;
    margin: 15px 0;
    border-radius: 6px;
    border: none;
    font-size: 16px;
}

.container {
    max-width: 1200px;
    margin: auto;
    padding: 20px;
}

table {
    width: 100%;
    border-collapse: collapse;
}

th, td {
    padding: 12px;
}

th {
    background: #020617;
    text-align: left;
}

tr:nth-child(even) {
    background: #020617;
}

.green { color: #22c55e; }
.red { color: #ef4444; }

.crypto-name {
    cursor: pointer;
}

canvas {
    background: #020617;
    border-radius: 8px;
    margin-top: 20px;
}
</style>
</head>

<body>

<header>
    <h1>📊 CryptoWatch</h1>
    <p>Cours des cryptomonnaies en temps réel</p>
</header>

<div class="container">
    <input type="text" id="search" placeholder="Rechercher une crypto...">

    <table>
        <thead>
            <tr>
                <th>#</th>
                <th>Crypto</th>
                <th>Prix (€)</th>
                <th>24h</th>
                <th>Market Cap</th>
            </tr>
        </thead>
        <tbody id="crypto-table"></tbody>
    </table>

    <canvas id="chart" height="100"></canvas>
</div>

<script>
const API =
"https://api.coingecko.com/api/v3/coins/markets?vs_currency=eur&order=market_cap_desc&per_page=100&page=1&sparkline=false";

let cryptos = [];
let chart;

async function loadCryptos() {
    const res = await fetch(API);
    cryptos = await res.json();
    displayCryptos(cryptos);
}

function displayCryptos(data) {
    const table = document.getElementById("crypto-table");
    table.innerHTML = "";

    data.forEach((coin, i) => {
        table.innerHTML += `
        <tr>
            <td>${i + 1}</td>
            <td class="crypto-name" onclick="loadChart('${coin.id}')">
                <img src="${coin.image}" width="18"> 
                ${coin.name} (${coin.symbol.toUpperCase()})
            </td>
            <td>${coin.current_price.toLocaleString()} €</td>
            <td class="${coin.price_change_percentage_24h >= 0 ? 'green' : 'red'}">
                ${coin.price_change_percentage_24h.toFixed(2)}%
            </td>
            <td>${coin.market_cap.toLocaleString()} €</td>
        </tr>`;
    });
}

document.getElementById("search").addEventListener("input", e => {
    const v = e.target.value.toLowerCase();
    displayCryptos(
        cryptos.filter(c =>
            c.name.toLowerCase().includes(v) ||
            c.symbol.toLowerCase().includes(v)
        )
    );
});

async function loadChart(id) {
    const res = await fetch(
        `https://api.coingecko.com/api/v3/coins/${id}/market_chart?vs_currency=eur&days=7`
    );
    const data = await res.json();

    const labels = data.prices.map(p =>
        new Date(p[0]).toLocaleDateString()
    );
    const prices = data.prices.map(p => p[1]);

    if (chart) chart.destroy();

    chart = new Chart(document.getElementById("chart"), {
        type: "line",
        data: {
            labels,
            datasets: [{
                label: "Prix (€)",
                data: prices,
                borderColor: "#38bdf8",
                tension: 0.4
            }]
        }
    });
}

loadCryptos();
setInterval(loadCryptos, 60000); // refresh 1 min
</script>

</body>
</html>

