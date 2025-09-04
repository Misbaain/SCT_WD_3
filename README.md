# SCT_WD_3
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Tic Tac Toe</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin: 0;
      padding: 0;
      background: linear-gradient(135deg, #89f7fe, #66a6ff);
      height: 100vh;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
    }

    h1 {
      color: #fff;
      text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
    }

    #game {
      display: grid;
      grid-template-columns: repeat(3, 100px);
      grid-template-rows: repeat(3, 100px);
      gap: 10px;
      margin: 20px auto;
    }

    .cell {
      background: #ffffffcc;
      border-radius: 20px;
      font-size: 2.5em;
      font-weight: bold;
      color: #333;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      transition: all 0.3s ease;
      box-shadow: 0 4px 6px rgba(0,0,0,0.2);
    }

    .cell:hover {
      background: #ffeaa7;
      transform: scale(1.05);
    }

    .cell.winner {
      background: #55efc4;
      color: #2d3436;
      box-shadow: 0 0 20px #00cec9;
    }

    #controls {
      margin-top: 15px;
    }

    button, select {
      margin: 5px;
      padding: 10px 15px;
      border: none;
      border-radius: 8px;
      background: #0984e3;
      color: white;
      font-size: 1em;
      cursor: pointer;
      transition: background 0.3s ease;
    }

    button:hover, select:hover {
      background: #74b9ff;
    }

    #status {
      margin-top: 15px;
      font-size: 1.2em;
      color: #fff;
      text-shadow: 1px 1px 3px rgba(0,0,0,0.3);
    }
  </style>
</head>
<body>
  <h1>Tic Tac Toe</h1>
  <div id="game"></div>
  <div id="controls">
    <button id="modeBtn">Switch to Vs Computer</button>
    <select id="difficulty">
      <option value="easy">Easy</option>
      <option value="medium">Medium</option>
      <option value="hard">Hard</option>
    </select>
    <button id="resetBtn">Reset</button>
  </div>
  <p id="status">Player X's turn</p>

  <script>
    const gameContainer = document.getElementById("game");
    const status = document.getElementById("status");
    const resetBtn = document.getElementById("resetBtn");
    const modeBtn = document.getElementById("modeBtn");
    const difficultySelect = document.getElementById("difficulty");

    let board = Array(9).fill("");
    let currentPlayer = "X";
    let vsComputer = false;
    let difficulty = "easy";
    let gameActive = true;

    // Winning combos
    const winCombos = [
      [0,1,2], [3,4,5], [6,7,8],
      [0,3,6], [1,4,7], [2,5,8],
      [0,4,8], [2,4,6]
    ];

    // Create board
    function createBoard() {
      gameContainer.innerHTML = "";
      board.forEach((val, idx) => {
        const cell = document.createElement("div");
        cell.classList.add("cell");
        cell.dataset.index = idx;
        cell.textContent = val;
        cell.addEventListener("click", handleClick);
        gameContainer.appendChild(cell);
      });
    }

    // Handle click
    function handleClick(e) {
      const idx = e.target.dataset.index;
      if (board[idx] !== "" || !gameActive) return;

      board[idx] = currentPlayer;
      e.target.textContent = currentPlayer;

      if (checkWinner(board, currentPlayer)) {
        endGame(`${currentPlayer} wins!`);
        highlightWinner(currentPlayer);
        return;
      } else if (!board.includes("")) {
        endGame("It's a draw!");
        return;
      }

      currentPlayer = currentPlayer === "X" ? "O" : "X";
      status.textContent = `Player ${currentPlayer}'s turn`;

      if (vsComputer && currentPlayer === "O") {
        setTimeout(computerMove, 500);
      }
    }

    // Computer move
    function computerMove() {
      let idx;
      if (difficulty === "easy") {
        let empty = board.map((v,i)=> v===""? i:null).filter(v=>v!==null);
        idx = empty[Math.floor(Math.random()*empty.length)];
      } else if (difficulty === "medium") {
        idx = mediumAI();
      } else {
        idx = bestMove();
      }
      board[idx] = "O";
      const cell = document.querySelector(`[data-index='${idx}']`);
      cell.textContent = "O";

      if (checkWinner(board,"O")) {
        endGame("Computer wins!");
        highlightWinner("O");
        return;
      } else if (!board.includes("")) {
        endGame("It's a draw!");
        return;
      }
      currentPlayer = "X";
      status.textContent = "Player X's turn";
    }

    // Medium AI: block or random
    function mediumAI() {
      for (let combo of winCombos) {
        let [a,b,c] = combo;
        if (board[a]==="X" && board[b]==="X" && board[c]==="") return c;
        if (board[a]==="X" && board[c]==="X" && board[b]==="") return b;
        if (board[b]==="X" && board[c]==="X" && board[a]==="") return a;
      }
      let empty = board.map((v,i)=> v===""? i:null).filter(v=>v!==null);
      return empty[Math.floor(Math.random()*empty.length)];
    }

    // Hard AI: minimax
    function bestMove() {
      let bestScore = -Infinity;
      let move;
      for (let i=0;i<9;i++) {
        if (board[i]==="") {
          board[i] = "O";
          let score = minimax(board,0,false);
          board[i] = "";
          if (score>bestScore) {
            bestScore=score;
            move=i;
          }
        }
      }
      return move;
    }

    function minimax(board, depth, isMaximizing) {
      if (checkWinner(board,"O")) return 10-depth;
      if (checkWinner(board,"X")) return depth-10;
      if (!board.includes("")) return 0;

      if (isMaximizing) {
        let bestScore=-Infinity;
        for (let i=0;i<9;i++) {
          if (board[i]==="") {
            board[i]="O";
            let score=minimax(board,depth+1,false);
            board[i]="";
            bestScore=Math.max(score,bestScore);
          }
        }
        return bestScore;
      } else {
        let bestScore=Infinity;
        for (let i=0;i<9;i++) {
          if (board[i]==="") {
            board[i]="X";
            let score=minimax(board,depth+1,true);
            board[i]="";
            bestScore=Math.min(score,bestScore);
          }
        }
        return bestScore;
      }
    }

    // Check winner
    function checkWinner(bd, player) {
      return winCombos.some(combo => combo.every(i => bd[i]===player));
    }

    // Highlight winner
    function highlightWinner(player) {
      for (let combo of winCombos) {
        if (combo.every(i=>board[i]===player)) {
          combo.forEach(i=>{
            document.querySelector(`[data-index='${i}']`).classList.add("winner");
          });
        }
      }
    }

    // End game
    function endGame(msg) {
      gameActive=false;
      status.textContent=msg;
      setTimeout(()=>{
        if (confirm(msg+" Play again?")) resetGame();
      },300);
    }

    // Reset
    function resetGame() {
      board=Array(9).fill("");
      currentPlayer="X";
      gameActive=true;
      status.textContent="Player X's turn";
      createBoard();
    }

    // Events
    resetBtn.addEventListener("click",resetGame);
    modeBtn.addEventListener("click",()=>{
      vsComputer=!vsComputer;
      modeBtn.textContent = vsComputer? "Switch to Vs Player" : "Switch to Vs Computer";
      resetGame();
    });
    difficultySelect.addEventListener("change",(e)=>{
      difficulty=e.target.value;
    });

    createBoard();
  </script>
</body>
</html>
