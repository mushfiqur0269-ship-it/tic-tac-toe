<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Tic‑Tac‑Toe — O & X</title>
  <style>
    :root{--bg:#0b1220;--card:#071220;--accent:#60a5fa;--accent2:#34d399;--text:#e6eef8}
    *{box-sizing:border-box}
    body{margin:0;min-height:100vh;display:flex;align-items:center;justify-content:center;background:linear-gradient(180deg,#071422,#021018);color:var(--text);font-family:Inter,system-ui,Segoe UI,Arial}
    .wrap{width:360px;padding:20px;border-radius:12px;background:linear-gradient(180deg,rgba(255,255,255,0.02),rgba(255,255,255,0.01));box-shadow:0 8px 30px rgba(2,6,23,0.6)}
    h1{margin:0 0 12px;font-size:20px;text-align:center}
    .controls{display:flex;gap:8px;justify-content:center;margin-bottom:12px}
    .controls button{padding:8px 12px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:var(--text);cursor:pointer}
    .controls button.active{background:var(--accent);color:#012138}
    .board{display:grid;grid-template-columns:repeat(3,100px);grid-template-rows:repeat(3,100px);gap:8px;justify-content:center}
    .cell{width:100px;height:100px;background:rgba(255,255,255,0.02);display:flex;align-items:center;justify-content:center;font-size:48px;cursor:pointer;border-radius:10px;border:1px solid rgba(255,255,255,0.03);user-select:none}
    .cell.x{color:#ffd166}
    .cell.o{color:#60a5fa}
    .info{margin-top:12px;text-align:center;color:rgba(230,238,248,0.9)}
    .footer{display:flex;justify-content:space-between;align-items:center;margin-top:12px}
    .small{font-size:13px;color:rgba(159,176,200,0.9)}
    .restart{padding:8px 10px;border-radius:8px;border:none;background:var(--accent2);color:#002318;cursor:pointer}
    @media(max-width:420px){.wrap{width:96vw}.board{grid-template-columns:repeat(3,1fr);grid-auto-rows:calc((96vw - 40px)/3)}}
  </style>
</head>
<body>
  <div class="wrap">
    <h1>Tic‑Tac‑Toe — O &amp; X</h1>

    <div class="controls">
      <button id="twoPlayerBtn" class="active">Two Player</button>
      <button id="vsAiBtn">Vs Computer</button>
    </div>

    <div class="board" id="board">
      <!-- 9 cells inserted by JS -->
    </div>

    <div class="info" id="info">Current: <strong id="turn">X</strong></div>

    <div class="footer">
      <div class="small" id="status">Make a move</div>
      <button class="restart" id="restartBtn">Restart</button>
    </div>
  </div>

  <script>
    // Simple Tic-Tac-Toe with optional AI (minimax)
    const boardEl = document.getElementById('board');
    const infoTurn = document.getElementById('turn');
    const statusEl = document.getElementById('status');
    const restartBtn = document.getElementById('restartBtn');
    const twoPlayerBtn = document.getElementById('twoPlayerBtn');
    const vsAiBtn = document.getElementById('vsAiBtn');

    let board = Array(9).fill(null); // null, 'X' or 'O'
    let current = 'X';
    let running = true;
    let mode = 'two'; // 'two' or 'ai'

    // Create cells
    for (let i = 0; i < 9; i++){
      const cell = document.createElement('div');
      cell.className = 'cell';
      cell.dataset.index = i;
      cell.addEventListener('click', onCellClick);
      boardEl.appendChild(cell);
    }

    function onCellClick(e){
      const idx = Number(e.currentTarget.dataset.index);
      if (!running) return;
      if (board[idx]) return; // occupied

      makeMove(idx, current);

      if (mode === 'ai' && running){
        // small delay for AI move to feel natural
        setTimeout(() => {
          const aiIdx = getBestMove(board, 'O');
          if (aiIdx !== null) makeMove(aiIdx, 'O');
        }, 300);
      }
    }

    function makeMove(idx, player){
      board[idx] = player;
      renderBoard();
      const winner = checkWinner(board);
      if (winner){
        running = false;
        if (winner === 'draw'){
          statusEl.textContent = 'It\'s a draw!';
        } else {
          statusEl.textContent = `Player ${winner} wins!`;
          highlightWin(winner);
        }
        infoTurn.textContent = '-';
        return;
      }

      // switch
      current = player === 'X' ? 'O' : 'X';
      infoTurn.textContent = current;
      statusEl.textContent = `Player ${current} to move`;

      // If AI mode and it's AI's turn, handled in click handler after human move
    }

    function renderBoard(){
      const cells = document.querySelectorAll('.cell');
      cells.forEach((cell, i) => {
        cell.textContent = board[i] || '';
        cell.classList.remove('x','o');
        if (board[i] === 'X') cell.classList.add('x');
        if (board[i] === 'O') cell.classList.add('o');
      })
    }

    function checkWinner(b){
      const lines = [
        [0,1,2],[3,4,5],[6,7,8],
        [0,3,6],[1,4,7],[2,5,8],
        [0,4,8],[2,4,6]
      ];
      for (let line of lines){
        const [a,b1,c] = line;
        if (b[a] && b[a] === b[b1] && b[a] === b[c]) return b[a];
      }
      if (b.every(v => v)) return 'draw';
      return null;
    }

    function highlightWin(player){
      const lines = [
        [0,1,2],[3,4,5],[6,7,8],
        [0,3,6],[1,4,7],[2,5,8],
        [0,4,8],[2,4,6]
      ];
      for (let line of lines){
        const [a,b1,c] = line;
        if (board[a] && board[a] === board[b1] && board[a] === board[c] && board[a] === player){
          line.forEach(i => document.querySelector(`.cell[data-index=\"${i}\"]`).style.boxShadow = '0 8px 20px rgba(96,165,250,0.18)');
          break;
        }
      }
    }

    restartBtn.addEventListener('click', resetGame);
    twoPlayerBtn.addEventListener('click', () => { setMode('two'); });
    vsAiBtn.addEventListener('click', () => { setMode('ai'); });

    function setMode(m){
      mode = m;
      twoPlayerBtn.classList.toggle('active', m === 'two');
      vsAiBtn.classList.toggle('active', m === 'ai');
      resetGame();
    }

    function resetGame(){
      board = Array(9).fill(null);
      current = 'X';
      running = true;
      statusEl.textContent = 'Make a move';
      infoTurn.textContent = current;
      document.querySelectorAll('.cell').forEach(c => { c.style.boxShadow = 'none'; });
      renderBoard();
    }

    // --- Simple minimax for Tic-Tac-Toe (AI plays 'O') ---
    function getBestMove(b, aiPlayer){
      // If board empty, choose center or corner
      if (b.every(v => !v)) return 4;

      const human = aiPlayer === 'O' ? 'X' : 'O';

      function minimax(boardState, player){
        const winner = checkWinner(boardState);
        if (winner === aiPlayer) return {score: 10};
        if (winner === human) return {score: -10};
        if (winner === 'draw') return {score: 0};

        const moves = [];
        for (let i = 0; i < 9; i++){
          if (!boardState[i]){
            const move = {index: i};
            boardState[i] = player;
            const result = minimax(boardState, player === aiPlayer ? human : aiPlayer);
            move.score = result.score;
            boardState[i] = null;
            moves.push(move);
          }
        }

        let bestMove;
        if (player === aiPlayer){
          // maximize
          let bestScore = -Infinity;
          for (let m of moves) if (m.score > bestScore){ bestScore = m.score; bestMove = m; }
        } else {
          // minimize
          let bestScore = Infinity;
          for (let m of moves) if (m.score < bestScore){ bestScore = m.score; bestMove = m; }
        }
        return bestMove;
      }

      const result = minimax(b.slice(), aiPlayer);
      return result ? result.index : null;
    }

    // init
    resetGame();
  </script>
</body>
</html>
