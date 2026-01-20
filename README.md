<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>CryptoWatch - Dashboard Crypto</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
/* BASE DU SITE */
body {
    margin: 0;
    font-family: 'Segoe UI', Arial, sans-serif;
    background-color: #131722;
    color: #f0f0f0;
}

/* HEADER */
header {
    background-color: #0b0c10;
    padding: 20px;
    text-align: center;
}

header h1 {
    color: #fcd535; /* couleur Binance */
    margin: 0;
    font-size: 2rem;
}

header p {
    color: #cfd2d8;
    margin-top: 8px;
}

/* CONTENEUR */
.container {
    max-width: 1200px;
    margin: auto;
    padding: 20px;
}

/* INPUT RECHERCHE */
input {
    width: 100%;
    padding: 12px;
    margin: 20px 0;
    border-radius: 8px;
    border: none;
    font-size: 16px;
    background-color: #1f212b;
    color: #f0f0f0;
}

/* TABLE STYLE BINANCE */
table {
    width: 100%;
    border-collapse: collapse;
}

th, td {
    padding: 12px;
    text-align: left;
}

th {
    background-color: #1f212b;
    color: #fcd535;
    text-transform: uppercase;
    font-size: 0.9rem;
}

tr {
    transition: background 0.2s;
}

tr:nth-child(even) {
    background-color: #181a20;
}

tr:hover {
    background-color: #272b38;
}

/* VARIATION DE PRIX */
.green { color: #22c55e; }
.red { color: #ef4444; }

/* CLIQUE SUR CRYPTO */
.crypto-name {
    cursor: pointer;
    display: flex;
    align-items: center;
}

.crypto-name img {
    margin-right: 8px;
    width: 20px;
}

/* GRAPHIQUE */
canvas {
    background-color: #1f212b;
    border-radius: 10px;
    margin-top: 20px;
}

/* FOOTER */
footer {
    text-align: center;
    padding: 15px;
    margin-top: 40px;
    color: #94a3b8;
    font-size: 0.9rem;
}

/* RESPONSIVE TABLE */
@media(max-width:768px) {
    table, thead, tbody, th, td, tr { display: block; }
    tr { margin-bottom: 15px; }
    th { display: none; }
    td {
        padding-left: 50%;
        position: relative;
    }
    td:before {
        content: attr(data-label);
        position: absolute;
        left: 15px;
        font-weight: bold;
        color: #fcd535;
    }
}
</style>
</head>

<body>

<header>
    <h1>CryptoWatch</h1>
    <p>Dashboard style Binance pour suivre les cours des cryptomonnaies en temps réel. Cliquez sur une crypto pour voir son graphique 7 jours.</p>
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

<footer>
    &copy; 2026 CryptoWatch | Données par CoinGecko
</footer>

<script>
const API = "https://api.coingecko.com/api/v3/coins/markets?vs_currency=eur&order=market_cap_desc&per_page=100&page=1&sparkline=false";

let cryptos = [];
let chart;

async function loadCryptos() {
    try {
        const res = await fetch(API);
        cryptos = await res.json();
        displayCryptos(cryptos);
    } catch (err) {
        console.error("Erreur API :", err);
    }
}

function displayCryptos(data) {
    const table = document.getElementById("crypto-table");
    table.innerHTML = "";
    data.forEach((coin, i) => {
        table.innerHTML += `
        <tr>
            <td data-label="#">${i + 1}</td>
            <td data-label="Crypto" class="crypto-name" onclick="loadChart('${coin.id}')">
                <img src="${coin.image}"> ${coin.name} (${coin.symbol.toUpperCase()})
            </td>
            <td data-label="Prix">${coin.current_price.toLocaleString()} €</td>
            <td data-label="24h" class="${coin.price_change_percentage_24h >= 0 ? 'green' : 'red'}">
                ${coin.price_change_percentage_24h.toFixed(2)}%
            </td>
            <td data-label="Market Cap">${co
