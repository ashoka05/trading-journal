<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Trading Journal</title>
    <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Kamal&display=swap">
    <style>
        body {
            font-family: 'Kamal', sans-serif;
            margin: 20px;
            background-image: url('https://i.ibb.co/kghFPst/6256878.jpg');
            background-size: cover;
            background-position: center;
            background-repeat: no-repeat;
            display: flex;
            flex-direction: column;
            align-items: center;
            color: #000;
        }

        h2 {
            color: #26a69a;
            font-weight: bold;
            margin-bottom: 20px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }

        form {
            max-width: 600px;
            margin: auto;
            background-color: rgba(255, 255, 255, 0.9);
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }

        label {
            display: block;
            margin-bottom: 8px;
            font-weight: bold;
        }

        input, select, textarea {
            width: 100%;
            padding: 10px;
            margin-bottom: 15px;
            box-sizing: border-box;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        button {
            background-color: #26a69a;
            color: white;
            padding: 12px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            transition: background-color 0.3s;
        }

        button:hover {
            background-color: #208c7a;
        }

        .time-action-container {
            display: flex;
            justify-content: space-between;
            margin-bottom: 15px;
        }

        .time-action-container label {
            width: 48%;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
            background-color: rgba(255, 255, 255, 0.9);
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }

        th, td {
            border: 1px solid #ddd;
            padding: 12px;
            text-align: center;
        }

        th {
            background-color: #26a69a;
            color: white;
        }

        .image-column img {
            max-width: 80px;
            max-height: 80px;
            border-radius: 4px;
        }

        #exportButtons {
            display: flex;
            justify-content: center;
            margin-top: 20px;
        }
    </style>
</head>
<body>

    <h2>Trading Journal</h2>

    <form id="tradingForm" enctype="multipart/form-data">
        <label for="market">Market:</label>
        <select id="market" name="market" required>
            <option value="" disabled selected>Select Market</option>
            <option value="NIFTY50">Nifty50</option>
            <option value="BANKNIFTY">BankNifty</option>
            <option value="MIDCAP">Midcap</option>
            <option value="SENSEX">Sensex</option>
        </select>

        <div class="time-action-container">
            <label for="time">Time:</label>
            <input type="time" id="time" name="time" required>

            <label for="action">Action:</label>
            <select id="action" name="action" required>
                <option value="call">Call</option>
                <option value="put">Put</option>
            </select>
        </div>

        <label for="quantity">Quantity:</label>
        <input type="number" id="quantity" name="quantity" required>

        <label for="price">Price:</label>
        <input type="number" id="price" name="price" step="0.01" required>

        <label for="buyingCost">Buying Cost:</label>
        <input type="text" id="buyingCost" name="buyingCost" readonly>

        <label for="note">Note:</label>
        <textarea id="note" name="note" rows="4"></textarea>

        <label for="image">Upload Image:</label>
        <input type="file" id="image" name="image" accept="image/*">

        <button type="button" onclick="addTrade()">Add Trade</button>
    </form>

    <table id="tradesTable">
        <thead>
            <tr>
                <th>Market</th>
                <th>Time</th>
                <th>Action</th>
                <th>Quantity</th>
                <th>Price</th>
                <th>Buying Cost</th>
                <th>Note</th>
                <th>Image</th>
            </tr>
        </thead>
        <tbody id="tradeData">
            <!-- Trade entries will be added here -->
        </tbody>
    </table>

    <div id="exportButtons">
        <button type="button" onclick="exportToExcel()">Export to Excel</button>
    </div>

    <script src="https://unpkg.com/xlsx/dist/xlsx.full.min.js"></script>

    <script>
        function addTrade() {
            var market = document.getElementById("market").value;
            var time = document.getElementById("time").value;
            var action = document.getElementById("action").value;
            var quantity = document.getElementById("quantity").value;
            var price = document.getElementById("price").value;
            var note = document.getElementById("note").value;

            var imageInput = document.getElementById("image");
            var image = imageInput.files[0];

            var buyingCost = (quantity * price).toFixed(2);

            var trades = JSON.parse(localStorage.getItem('trades')) || [];

            // Read the image using FileReader
            var reader = new FileReader();
            reader.onload = function(e) {
                var trade = {
                    market: market,
                    time: time,
                    action: action,
                    quantity: quantity,
                    price: price,
                    buyingCost: buyingCost,
                    note: note,
                    image: e.target.result // Set the image data URL
                };
                trades.push(trade);

                localStorage.setItem('trades', JSON.stringify(trades));
                displayTrades();
            };
            reader.readAsDataURL(image); // Read the image as a data URL
        }

        function displayTrades() {
            var table = document.getElementById("tradeData");
            table.innerHTML = "";

            var trades = JSON.parse(localStorage.getItem('trades')) || [];

            trades.forEach(function(trade) {
                var row = table.insertRow(table.rows.length);
                Object.keys(trade).forEach(function(key) {
                    var cell = row.insertCell(-1);

                    if (key === 'image') {
                        var imageCell = document.createElement("div");
                        imageCell.className = "image-column";
                        var img = document.createElement("img");
                        img.src = trade[key];
                        img.alt = "Trade Image";
                        imageCell.appendChild(img);
                        cell.appendChild(imageCell);
                    } else {
                        cell.innerHTML = trade[key];
                    }
                });
            });
        }

        function clearTrades() {
            localStorage.removeItem('trades');
            displayTrades();
        }

        window.onload = function() {
            clearTrades();
        };

        function exportToExcel() {
            var trades = JSON.parse(localStorage.getItem('trades')) || [];
            var worksheet = XLSX.utils.json_to_sheet(trades);
            var workbook = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(workbook, worksheet, 'Trades');
            XLSX.writeFile(workbook, 'trades.xlsx');
        }

        // Calculate buying cost on quantity and price change
        document.getElementById("quantity").addEventListener("input", calculateBuyingCost);
        document.getElementById("price").addEventListener("input", calculateBuyingCost);

        function calculateBuyingCost() {
            var quantity = document.getElementById("quantity").value;
            var price = document.getElementById("price").value;
            var buyingCost = (quantity * price).toFixed(2);
            document.getElementById("buyingCost").value = buyingCost;
        }
    </script>

</body>
</html>
