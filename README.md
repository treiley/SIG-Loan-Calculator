# SIG-Loan-Calculator
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Reiley Loan Calculator</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 2rem;
      background-color: #121212;
      color: #f1f1f1;
    }
    .container {
      max-width: 500px;
      margin: auto;
      background-color: #1e1e1e;
      padding: 2rem;
      border-radius: 12px;
      box-shadow: 0 0 10px rgba(0,0,0,0.8);
    }
    h2 {
      text-align: center;
    }
    label {
      margin-top: 1rem;
      display: block;
    }
    input {
      width: 96.5%;
      padding: 0.5rem;
      margin-top: 0.5rem;
      background-color: #333;
      border: 1px solid #555;
      color: white;
      border-radius: 5px;
    }
    button {
      margin-top: 1.5rem;
      width: 100%;
      padding: 0.75rem;
      background-color: #007bff;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
    }
    button:hover {
      background-color: #0056b3;
    }
    .result {
      margin-top: 1rem;
      font-weight: bold;
    }
    .positive { color: #28a745; }
    .negative { color: #dc3545; }
    .neutral { color: #f1f1f1; }
    ..signature {
    position: fixed;
    top: 20px;
    left: 20px;
    font-size: 1.05rem;
    color: #fff;
    background: rgba(0, 0, 0, 0.6); /* Slightly transparent background */
    padding: 10px;
    border-radius: 8px;
    line-height: 1.2;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.4);
  }
  .signature a {
    color: #66b2ff;
    text-decoration: none;
  }
  </style>
</head>
<body>
  <div class="signature">
  <strong>Ty Reiley</strong><br />
  Senior Capital Markets Advisor, Sands Investment Group<br/>
  <a href="mailto:treiley@sandsig.com">treiley@sandsig.com</a> | 512.649.2421 | 
  <a href="https://www.SandsIG.com" target="_blank">www.SandsIG.com</a>
</div>

  <div class="container">
    <h2>Loan Calculator</h2>

    <label for="purchasePrice">Purchase Price ($):</label>
    <input type="number" id="purchasePrice" placeholder="Enter purchase price" />

    <label for="loanAmount">Loan Amount ($):</label>
    <input type="number" id="loanAmount" placeholder="Enter loan amount" />

    <label for="interestRate">Annual Interest Rate (%):</label>
    <input type="number" id="interestRate" step="0.01" placeholder="e.g. 6" />

    <label for="amortization">Amortization (years):</label>
    <input type="number" id="amortization" placeholder="e.g. 25" />

    <label for="noi">Net Operating Income (NOI) ($):</label>
    <input type="number" id="noi" placeholder="Enter NOI" />

    <button onclick="calculateLoan()">Calculate</button>

    <div class="result" id="capRateOutput"></div>
    <div class="result" id="downPaymentOutput"></div>
    <div class="result" id="annualDebtServiceOutput"></div>
    <div class="result" id="cashOnCashOutput"></div>
    <div class="result" id="leverageStatus"></div>
    <div class="result" id="dscrOutput"></div>
    <div class="result" id="ltvOutput"></div>
    <div class="result" id="maxLtvOutput"></div>
  </div>

  <script>
    function formatNumber(num) {
      return num.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 });
    }

    function calculateLoan() {
      const purchasePrice = parseFloat(document.getElementById("purchasePrice").value);
      const loanAmount = parseFloat(document.getElementById("loanAmount").value);
      const interestRate = parseFloat(document.getElementById("interestRate").value) / 100;
      const amortization = parseInt(document.getElementById("amortization").value);
      const noi = parseFloat(document.getElementById("noi").value);

      if ([purchasePrice, loanAmount, interestRate, amortization, noi].some(isNaN)) {
        alert("Please enter all input fields correctly.");
        return;
      }

      const downPayment = purchasePrice - loanAmount;
      const monthlyRate = interestRate / 12;
      const n = amortization * 12;
      const monthlyPayment = loanAmount * monthlyRate / (1 - Math.pow(1 + monthlyRate, -n));
      const annualDebtService = monthlyPayment * 12;
      const loanConstant = (annualDebtService / loanAmount) * 100;
      const capRate = (noi / purchasePrice) * 100;

      const cashFlow = noi - annualDebtService;
      const cashOnCashReturn = (cashFlow / downPayment) * 100;
      const dscr = noi / annualDebtService;
      const ltv = (loanAmount / purchasePrice) * 100;

      const maxAnnualDebtService = noi / 1.25;
      const loanConstantDecimal = annualDebtService / loanAmount;
      const maxLoanAmount = maxAnnualDebtService / loanConstantDecimal;
      const maxLTV = (maxLoanAmount / purchasePrice) * 100;

      document.getElementById("capRateOutput").innerText = `Cap Rate: ${formatNumber(capRate)}%`;
      document.getElementById("downPaymentOutput").innerText = `Down Payment: $${formatNumber(downPayment)}`;
      document.getElementById("annualDebtServiceOutput").innerText = `Annual Debt Service: $${formatNumber(annualDebtService)}`;
      document.getElementById("cashOnCashOutput").innerText = `Cash on Cash Return: ${formatNumber(cashOnCashReturn)}%`;

      const leverageEl = document.getElementById("leverageStatus");
      if (loanConstant < capRate) {
        leverageEl.innerText = "Positive Leverage";
        leverageEl.className = "result positive";
      } else if (loanConstant > capRate) {
        leverageEl.innerText = "Negative Leverage";
        leverageEl.className = "result negative";
      } else {
        leverageEl.innerText = "Neutral Leverage";
        leverageEl.className = "result neutral";
      }

      const dscrEl = document.getElementById("dscrOutput");
      dscrEl.innerText = `DSCR: ${formatNumber(dscr)}`;
      dscrEl.className = "result " + (dscr >= 1.25 ? "positive" : "negative");

      document.getElementById("ltvOutput").innerText = `LTV: ${formatNumber(ltv)}%`;
      document.getElementById("maxLtvOutput").innerText = `Max LTV (1.25 DSCR): ${formatNumber(maxLTV)}%`;
    }
  </script>
</body>
</html>
