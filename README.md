<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Mines Web Bot</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      background-color: #121212;
      color: #f3f4f6;
      margin: 0;
      padding: 0;
    }
    .grid {
      display: grid;
      grid-template-columns: repeat(5, 60px);
      gap: 10px;
      justify-content: center;
      margin-top: 20px;
    }
    .cell {
      width: 60px;
      height: 60px;
      background-color: #2a2a2a;
      border: 2px solid #9ca3af;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      font-size: 18px;
      border-radius: 8px;
      color: white;
    }
    .safe {
      background-color: #22c55e;
    }
    .mine {
      background-color: #ef4444;
    }
    button {
      margin: 10px;
      padding: 10px 20px;
      font-size: 16px;
      cursor: pointer;
      border: none;
      border-radius: 8px;
      background-color: #3b82f6;
      color: white;
    }
    .controls {
      margin-top: 10px;
    }
    .stats {
      margin-top: 10px;
      font-size: 16px;
    }
    @media (max-width: 600px) {
      .cell {
        width: 45px;
        height: 45px;
        font-size: 14px;
      }
      .grid {
        gap: 6px;
      }
    }
  </style>
</head>
<body>
  <h1>Mines Web Bot</h1>
  <div class="controls">
    <button onclick="startGame()">🔄 Начать заново</button>
    <button onclick="suggestNextMove(true)">🤖 Авто-ход</button>
    <label>
      💣 Мины:
      <select id="mineSelect" onchange="updateMineCount()">
        <option value="3">3</option>
        <option value="5">5</option>
        <option value="7">7</option>
      </select>
    </label>
  </div>
  <div class="stats">
    🏆 Победы: <span id="wins">0</span> | 💥 Поражения: <span id="losses">0</span> | ⏱ Время: <span id="timer">0</span> сек
  </div>
  <div class="grid" id="grid"></div>

  <script>
    const gridSize = 5;
    let mineCount = 3;
    let safeCells = [];
    let revealed = new Set();
    let timer = 0;
    let interval;
    let wins = 0, losses = 0;

    function updateMineCount() {
      mineCount = parseInt(document.getElementById("mineSelect").value);
      startGame();
    }

    function startGame() {
      clearInterval(interval);
      timer = 0;
      document.getElementById("timer").textContent = timer;
      interval = setInterval(() => {
        timer++;
        document.getElementById("timer").textContent = timer;
      }, 1000);

      const grid = document.getElementById("grid");
      grid.innerHTML = "";
      safeCells = generateSafeCells();
      revealed.clear();

      for (let i = 0; i < gridSize * gridSize; i++) {
        const cell = document.createElement("div");
        cell.className = "cell";
        cell.dataset.index = i;
        cell.onclick = () => revealCell(cell);
        grid.appendChild(cell);
      }
    }

    function generateSafeCells() {
      let cells = Array.from({ length: gridSize * gridSize }, (_, i) => i);
      let mines = [];
      while (mines.length < mineCount) {
        const index = Math.floor(Math.random() * cells.length);
        mines.push(cells.splice(index, 1)[0]);
      }
      return cells; // безопасные клетки
    }

    function revealCell(cell) {
      const index = parseInt(cell.dataset.index);
      if (revealed.has(index)) return;
      revealed.add(index);

      if (safeCells.includes(index)) {
        cell.classList.add("safe");
        cell.textContent = "✓";
        if (revealed.size === safeCells.length) {
          clearInterval(interval);
          wins++;
          document.getElementById("wins").textContent = wins;
          alert("🎉 Победа! Все безопасные клетки открыты.");
        }
      } else {
        cell.classList.add("mine");
        cell.textContent = "💣";
        clearInterval(interval);
        losses++;
        document.getElementById("losses").textContent = losses;
        alert("💥 Попалась мина! Игра окончена.");
      }
      cell.onclick = null;
    }

    function suggestNextMove(auto = false) {
      const unrevealedSafe = safeCells.filter(i => !revealed.has(i));
      if (unrevealedSafe.length === 0) return;
      const index = unrevealedSafe[Math.floor(Math.random() * unrevealedSafe.length)];
      const cell = document.querySelector(`.cell[data-index='${index}']`);
      cell.style.border = "3px solid gold";
      if (auto) cell.click();
    }

    startGame();
  </script>
</body>
</html>
