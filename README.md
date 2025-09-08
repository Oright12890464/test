<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>退料多功能轉換工具</title>
  <style>
    body {
      font-family: "Microsoft JhengHei", sans-serif;
      background: #f5f5f5;
      padding: 2rem;
      display: flex;
      justify-content: center;
      align-items: flex-start;
      min-height: 100vh;
    }
    .container {
      width: 100%;
      max-width: 400px;
      background: white;
      padding: 2rem 2.5rem;
      border-radius: 12px;
      box-shadow: 0 4px 15px rgba(0,0,0,0.1);
      box-sizing: border-box;
      position: relative; /* 使彈窗相對於容器定位 */
    }
    h2 {
      text-align: center;
      margin-bottom: 1.5rem;
      color: #222;
    }
    label {
      display: block;
      margin-top: 1rem;
      font-weight: 600;
      color: #444;
    }
    input[type=number] {
      width: 100%;
      padding: 0.5rem 0.75rem;
      margin-top: 0.25rem;
      border: 1px solid #ccc;
      border-radius: 8px;
      font-size: 1rem;
      box-sizing: border-box;
      transition: border-color 0.3s;
    }
    input[type=number]:focus {
      outline: none;
      border-color: #4CAF50;
      box-shadow: 0 0 6px #4CAF50aa;
    }
    button {
      margin-top: 15px;
      padding: 12px;
      font-size: 16px;
      background-color: #4CAF50;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      width: 48%;
      transition: background-color 0.3s ease;
    }
    button:hover {
      background-color: #3a9d3a;
    }
    .result {
      margin-top: 20px;
      background: #e8f5e9;
      border: 2px solid #4CAF50;
      border-radius: 5px;
      padding: 15px;
      font-size: 16px;
      color: #2e7d32;
      white-space: pre-line;
    }
    .readonly {
      background-color: #f4f4f4;
    }
    .button-group {
      display: flex;
      justify-content: space-between;
      gap: 10px;
      flex-wrap: wrap;
    }
    
    /* 浮動按鈕樣式 */
    .floating-button {
      position: fixed;
      bottom: 20px;
      right: 20px;
      background-color: #4CAF50;
      color: white;
      border: none;
      border-radius: 50%;
      width: 50px;
      height: 50px;
      font-size: 24px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
      cursor: pointer;
      display: none; /* 預設隱藏 */
    }

    @media (max-width: 480px) {
      .floating-button {
        display: block; /* 手機上顯示 */
      }
    }
  </style>
</head>
<body>

<div class="container">
    <div id="returnCalculator">
        <h2>退料</h2>
        <label>原始重量：
            <input type="number" id="totalWeight" step="any" min="0" placeholder="請輸入原始重量" oninput="calculateRightAuto()">
        </label>
        <label>軸心重量：
            <input type="number" id="coreWeight" step="any" min="0" placeholder="請輸入軸心重量" oninput="calculateRightAuto()">
        </label>
        <label>原始捲數：
            <input type="number" id="originalRolls" min="0" placeholder="請輸入原始單捲數量" oninput="calculateRightAuto()">
        </label>
        <label>退料重量：
            <input type="number" id="returnWeight" step="any" min="0" placeholder="請輸入實際退料重量" oninput="calculateRightAuto()">
        </label>
        <label>換算貼紙張數：
            <input type="number" id="calculatedResult" class="readonly" readonly>
        </label>
        <div class="button-group">
            <button onclick="calculateLeft()">計算</button>
            <button onclick="clearAll()">清除</button>
        </div>
        <div class="result" id="leftResult" style="display:none;"></div>
    </div>
</div>

<!-- 浮動的 "123" 按鈕 -->
<button class="floating-button" id="numberKeyboardButton">123</button>

<script>
    const numberKeyboardButton = document.getElementById('numberKeyboardButton');
    let currentInput = null;

    // 當用戶聚焦到任何輸入框時，記錄當前輸入框
    document.querySelectorAll('input[type="number"]').forEach(input => {
        input.addEventListener('focus', function() {
            currentInput = this; // 記錄當前輸入框
        });
    });

    // 點擊浮動按鈕時切換鍵盤
    numberKeyboardButton.addEventListener('click', function() {
        if (currentInput) {
            currentInput.focus(); // 將焦點設置到當前輸入框
            currentInput.type = 'number'; // 將輸入框類型設置為數字
        }
    });

    // 退料計算邏輯
    function calculateRightAuto() {
        const totalWeight = parseFloat(document.getElementById('totalWeight').value) || 0;
        const coreWeight = parseFloat(document.getElementById('coreWeight').value) || 0;
        const originalRolls = parseInt(document.getElementById('originalRolls').value) || 0;
        const returnWeight = parseFloat(document.getElementById('returnWeight').value) || 0;

        if (totalWeight <= coreWeight || originalRolls <= 0 || returnWeight < coreWeight) {
            document.getElementById('calculatedResult').value = '';
            return;
        }

        const val1 = totalWeight - coreWeight;
        const val2 = originalRolls / val1;
        const rawResult = (returnWeight - coreWeight) * val2;
        const fractional = rawResult % 1;
        const finalResult = fractional > 0 ? Math.ceil(rawResult) : rawResult;

        document.getElementById('calculatedResult').value = finalResult;
    }

    function calculateLeft() {
        const prepared = parseInt(document.getElementById('preparedTotal').value) || 0;
        const workOrder = parseInt(document.getElementById('workOrder').value) || 0;
        const actualReturn = parseInt(document.getElementById('actualReturn').value) || 0;
        const blueReturnInput = parseInt(document.getElementById('blueReturn').value) || 0;

        const shouldReturn = prepared - workOrder;
        let goodReturn = 0, badReturn = 0, requirePick = 0;

        if (actualReturn > shouldReturn) {
            goodReturn = actualReturn - shouldReturn;
            badReturn = blueReturnInput - goodReturn;
        } else if (actualReturn < shouldReturn) {
            requirePick = shouldReturn - actualReturn;
            badReturn = blueReturnInput + requirePick;
        } else {
            badReturn = blueReturnInput;
        }

        const outputLines = [
            `領料單: ${requirePick}`,
            `藍單良品: ${goodReturn}`,
            `藍單不良品: ${badReturn}`
        ];

        const leftResultDiv = document.getElementById('leftResult');
        leftResultDiv.textContent = outputLines.join('\n');
        leftResultDiv.style.display = 'block';
    }

    function clearAll() {
        const inputs = document.querySelectorAll('input[type=number]');
        inputs.forEach(input => {
            input.value = '';
        });
        document.getElementById('leftResult').style.display = 'none';
    }
</script>

</body>
</html>
