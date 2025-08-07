<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Worldwide Crypto Price Tracker</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f0f2f5;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
        }
        h1 {
            text-align: center;
            color: #333;
        }
        .crypto-table {
            width: 100%;
            border-collapse: collapse;
            background: white;
            border-radius: 8px;
            overflow: hidden;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background-color: #4CAF50;
            color: white;
        }
        tr:hover {
            background-color: #f5f5f5;
        }
        .price-up {
            color: #2ecc71;
        }
        .price-down {
            color: #e74c3c;
        }
        .refresh-btn {
            display: block;
            margin: 20px auto;
            padding: 10px 20px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }
        .refresh-btn:hover {
            background-color: #45a049;
        }
        .loading {
            text-align: center;
            padding: 20px;
            display: none;
        }
        .search-bar {
            margin: 20px 0;
            text-align: center;
        }
        .search-bar input {
            padding: 10px;
            width: 300px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
        }
        .currency-select {
            margin: 20px auto;
            text-align: center;
        }
        .currency-select select {
            padding: 10px;
            font-size: 16px;
        }
        .pagination {
            text-align: center;
            margin: 20px 0;
        }
        .pagination button {
            padding: 8px 16px;
            margin: 0 5px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        .pagination button:hover {
            background-color: #45a049;
        }
        .pagination button:disabled {
            background-color: #cccccc;
            cursor: not-allowed;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Cryptocurrency Price Tracker (Worldwide)</h1>
        <div class="currency-select">
            <select id="currencySelect" onchange="fetchCryptoData(currentPage)">
                <option value="usd">US Dollar (USD)</option>
                <option value="inr">Indian Rupee (INR)</option>
                <option value="eur">Euro (EUR)</option>
                <option value="gbp">British Pound (GBP)</option>
                <option value="jpy">Japanese Yen (JPY)</option>
                <option value="aud">Australian Dollar (AUD)</option>
                <option value="cad">Canadian Dollar (CAD)</option>
                <option value="chf">Swiss Franc (CHF)</option>
                <option value="cny">Chinese Yuan (CNY)</option>
                <option value="sgd">Singapore Dollar (SGD)</option>
                <!-- Add more currencies as needed -->
            </select>
        </div>
        <div class="search-bar">
            <input type="text" id="searchInput" placeholder="Search cryptocurrencies..." onkeyup="filterTable()">
        </div>
        <button class="refresh-btn" onclick="fetchCryptoData(currentPage)">Refresh Prices</button>
        <div class="loading" id="loading">Loading...</div>
        <table class="crypto-table" id="cryptoTable">
            <thead>
                <tr>
                    <th>#</th>
                    <th>Name</th>
                    <th>Symbol</th>
                    <th>Price</th>
                    <th>24h Change</th>
                    <th>Market Cap</th>
                </tr>
            </thead>
            <tbody id="cryptoTableBody"></tbody>
        </table>
        <div class="pagination" id="pagination"></div>
    </div>

    <script>
        let currentPage = 1;
        const perPage = 50; // Number of coins per page
        let currentCurrency = 'usd';

        async function fetchCryptoData(page) {
            const loading = document.getElementById('loading');
            const tableBody = document.getElementById('cryptoTableBody');
            const pagination = document.getElementById('pagination');
            loading.style.display = 'block';
            tableBody.innerHTML = '';

            const currency = document.getElementById('currencySelect').value;
            currentCurrency = currency; // Update the current currency

            try {
                const response = await fetch(`https://api.coingecko.com/api/v3/coins/markets?vs_currency=${currency}&order=market_cap_desc&per_page=${perPage}&page=${page}&sparkline=false`);
                const data = await response.json();

                data.forEach((coin, index) => {
                    const row = document.createElement('tr');
                    const priceChangeClass = coin.price_change_percentage_24h >= 0 ? 'price-up' : 'price-down';
                    
                    row.innerHTML = `
                        <td>${(page - 1) * perPage + index + 1}</td>
                        <td><img src="${coin.image}" alt="${coin.name}" width="24" height="24"> ${coin.name}</td>
                        <td>${coin.symbol.toUpperCase()}</td>
                        <td>${currentCurrency === 'usd' ? '$' : currencySymbol(currency)}${coin.current_price.toFixed(2)}</td>
                        <td class="${priceChangeClass}">${coin.price_change_percentage_24h.toFixed(2)}%</td>
                        <td>${currentCurrency === 'usd' ? '$' : currencySymbol(currency)}${coin.market_cap.toLocaleString()}</td>
                    `;
                    tableBody.appendChild(row);
                });

                // Update pagination
                pagination.innerHTML = `
                    <button onclick="fetchCryptoData(${page - 1})" ${page === 1 ? 'disabled' : ''}>Previous</button>
                    <span>Page ${page}</span>
                    <button onclick="fetchCryptoData(${page + 1})">Next</button>
                `;
            } catch (error) {
                console.error('Error fetching data:', error);
                tableBody.innerHTML = '<tr><td colspan="6">Error loading data. Please try again.</td></tr>';
            } finally {
                loading.style.display = 'none';
            }
        }

        function currencySymbol(currency) {
            const symbols = {
                inr: '₹',
                eur: '€',
                gbp: '£',
                jpy: '¥',
                aud: 'A$',
                cad: 'C$',
                chf: 'CHF',
                cny: '¥',
                sgd: 'S$',
                // Add more currency symbols as necessary
            };
            return symbols[currency] || '';
        }

        function filterTable() {
            const searchInput = document.getElementById('searchInput').value.toLowerCase();
            const rows = document.getElementById('cryptoTableBody').getElementsByTagName('tr');

            for (let row of rows) {
                const name = row.getElementsByTagName('td')[1].textContent.toLowerCase();
                const symbol = row.getElementsByTagName('td')[2].textContent.toLowerCase();
                if (name.includes(searchInput) || symbol.includes(searchInput)) {
                    row.style.display = '';
                } else {
                    row.style.display = 'none';
                }
            }
        }

        // Initial fetch
        fetchCryptoData(currentPage);

        // Auto-refresh every 60 seconds
        setInterval(() => fetchCryptoData(currentPage), 60000);
    </script>
</body>
</html>
