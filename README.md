# Tic-tac-toe<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tic Tac Toe Game</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: 'Arial', sans-serif;
    }
    body {
      background: #f2f2f2;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
      min-height: 100vh;
      padding: 20px;
      transition: background 0.3s ease;
    }
    body.dark { background: #222; color: white; }
    body.blue { background: #1e90ff; color: white; }
    body.purple { background: #6a5acd; color: white; }
    body.gray { background: #808080; color: white; }
    h1 {
      margin-top: 30px;
      font-size: 2rem;
      margin-bottom: 20px;
    }
    .board {
      display: grid;
      grid-template-columns: repeat(3, 100px);
      gap: 10px;
      margin: 20px 0;
      opacity: 1;
      transition: opacity 0.5s ease;
    }
    .board.reset { opacity: 0; }
    .board.draw { animation: shake 0.5s ease; }
    .cell {
      width: 100px;
      height: 100px;
      background: white;
      border: 2px solid #333;
      font-size: 2.5rem;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      transition: all 0.2s ease;
      position: relative;
      transform: scale(1);
      overflow: hidden;
    }
    .cell.clicked { animation: cellClick 0.3s ease; }
    .cell.ripple::after {
      content: '';
      position: absolute;
      top: 50%;
      left: 50%;
      width: 0;
      height: 0;
      background: rgba(0, 255, 255, 0.5);
      border-radius: 50%;
      transform: translate(-50%, -50%);
      animation: rippleEffect 0.6s ease-out;
    }
    .cell.winner {
      animation: winnerBlink 0.5s ease infinite;
    }
    .cell.dark { background: #444; border: 2px solid #777; }
    .cell.dark:hover { background: #555; }
    .cell:hover { background: #e0e0e0; }
    .scoreboard {
      margin-top: 15px;
      font-size: 1.2rem;
      opacity: 1;
      transition: opacity 0.5s ease;
    }
    .scoreboard.update {
      opacity: 0;
      animation: fadeIn 0.5s ease forwards;
    }
    .menu-button {
      position: absolute;
      top: 20px;
      right: 20px;
      font-size: 24px;
      cursor: pointer;
      background: none;
      border: none;
      transition: transform 0.2s ease;
    }
    .menu-button:hover { transform: scale(1.2); }
    .menu-panel {
      position: fixed;
      top: -100%;
      left: 0;
      right: 0;
      background: white;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
      padding: 20px;
      transition: top 0.4s ease-in-out, transform 0.4s ease-in-out;
      z-index: 10;
      transform: translateY(0);
      max-height: 80vh;
      overflow-y: auto;
    }
    .menu-panel.dark { background: #333; color: white; }
    .menu-panel.open {
      top: 0;
      transform: translateY(0);
    }
    .floating-panel {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%) scale(0);
      background: white;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
      padding: 20px;
      border-radius: 10px;
      z-index: 20;
      transition: transform 0.3s ease, opacity 0.3s ease;
      opacity: 0;
      max-height: 70vh;
      overflow-y: auto;
      min-width: 200px;
    }
    .floating-panel.dark { background: #333; color: white; }
    .floating-panel.open {
      transform: translate(-50%, -50%) scale(1);
      opacity: 1;
    }
    .close-button {
      position: absolute;
      top: 10px;
      right: 10px;
      font-size: 24px;
      cursor: pointer;
      background: none;
      border: none;
      transition: transform 0.2s ease;
    }
    .close-button:hover { transform: scale(1.2); }
    .setting-group {
      margin-bottom: 15px;
    }
    label {
      display: block;
      font-weight: bold;
      margin-bottom: 5px;
    }
    select, input[type=range] {
      width: 100%;
      padding: 5px;
      border-radius: 5px;
      cursor: pointer;
    }
    .new-game, .reset-score, .undo-button, .redo-button, .dev-button, .remove-history, .back-button {
      padding: 10px 20px;
      border: none;
      border-radius: 5px;
      font-size: 1rem;
      cursor: pointer;
      margin-top: 10px;
      transition: transform 0.2s ease, background-color 0.2s ease;
    }
    .new-game, .undo-button, .redo-button, .dev-button, .back-button {
      background-color: #4CAF50;
      color: white;
    }
    .new-game:hover, .undo-button:hover, .redo-button:hover, .dev-button:hover, .back-button:hover {
      background-color: #45a049;
      transform: scale(1.05);
    }
    .reset-score, .remove-history {
      background-color: #f44336;
      color: white;
      padding: 8px 15px;
      font-size: 0.9rem;
    }
    .reset-score:hover, .remove-history:hover {
      background-color: #d32f2f;
      transform: scale(1.05);
    }
    .win-popup, .draw-popup {
      position: fixed;
      top: 20%;
      left: 50%;
      transform: translateX(-50%) scale(0);
      background: #4CAF50;
      color: white;
      padding: 15px 30px;
      border-radius: 10px;
      font-size: 1.5rem;
      font-weight: bold;
      z-index: 20;
      opacity: 0;
      transition: transform 0.3s ease, opacity 0.3s ease;
    }
    .draw-popup { background: #ff9800; }
    .win-popup.show, .draw-popup.show {
      transform: translateX(-50%) scale(1);
      opacity: 1;
      animation: bounce 0.5s ease;
    }
    .game-history {
      margin-top: 20px;
      font-size: 0.9rem;
    }
    .game-history ul {
      list-style: none;
      padding: 0;
    }
    .game-history li {
      margin: 5px 0;
    }
    .controls {
      display: flex;
      gap: 10px;
      margin-top: 10px;
    }
    #gamePage, #devPage {
      display: none;
    }
    #gamePage.active, #devPage.active {
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    .dev-info {
      margin: 20px;
      font-size: 1.1rem;
      text-align: left;
      max-width: 600px;
    }
    .dev-info a {
      color: #2196f3;
      text-decoration: none;
    }
    .dev-info a:hover {
      text-decoration: underline;
    }

    /* Animations */
    @keyframes cellClick {
      0% { transform: scale(1); }
      50% { transform: scale(1.2); }
      100% { transform: scale(1); }
    }
    @keyframes rippleEffect {
      0% { width: 0; height: 0; opacity: 0.5; }
      100% { width: 150px; height: 150px; opacity: 0; }
    }
    @keyframes winnerBlink {
      0% { opacity: 1; }
      50% { opacity: 0.5; }
      100% { opacity: 1; }
    }
    @keyframes shake {
      0% { transform: translateX(0); }
      25% { transform: translateX(-5px); }
      50% { transform: translateX(5px); }
      75% { transform: translateX(-5px); }
      100% { transform: translateX(0); }
    }
    @keyframes fadeIn {
      0% { opacity: 0; }
      100% { opacity: 1; }
    }
    @keyframes bounce {
      0% { transform: translateX(-50%) scale(0); }
      50% { transform: translateX(-50%) scale(1.2); }
      100% { transform: translateX(-50%) scale(1); }
    }
  </style>
</head>
<body>
  <div id="gamePage" class="active">
    <button class="menu-button" onclick="toggleMenu()">☰</button>
    <div class="menu-panel" id="menuPanel">
      <button class="close-button" id="closeButton" onclick="toggleMenu()">✕</button>
      <div class="setting-group">
        <label>Settings</label>
        <button onclick="openFloatingPanel('volume')" class="new-game">Volume</button>
        <button onclick="openFloatingPanel('xColor')" class="new-game">Player X Color</button>
        <button onclick="openFloatingPanel('oColor')" class="new-game">Player O Color</button>
        <button onclick="openFloatingPanel('background')" class="new-game">Background</button>
        <button onclick="openFloatingPanel('bot')" class="new-game">Bot Difficulty</button>
        <button onclick="openFloatingPanel('history')" class="new-game">Game History</button>
      </div>
      <button onclick="startNewGame()" class="new-game">New Game</button>
      <button onclick="resetScore()" class="reset-score">Reset Score</button>
    </div>
    <div class="floating-panel" id="floatingPanel">
      <button class="close-button" id="floatingCloseButton" onclick="closeFloatingPanel()">✕</button>
      <div class="setting-group" id="floatingPanelContent"></div>
    </div>
    
    <h1>Tic Tac Toe</h1>
    <div class="board" id="board"></div>
    <div class="scoreboard" id="score">X: 0 | O: 0</div>
    <div class="controls">
      <button class="undo-button" onclick="undoMove()">Undo</button>
      <button class="redo-button" onclick="redoMove()">Redo</button>
      <button class="dev-button" onclick="showDevPage()">Dev</button>
    </div>
    <div class="win-popup" id="winPopup"></div>
    <div class="draw-popup" id="drawPopup"></div>
  </div>

  <div id="devPage">
    <h1>Developer Information</h1>
    <div class="dev-info">
      <p><strong>Dev-Name:</strong> Sagor</p>
      <p><strong>Region:</strong> BD</p>
      <p><strong>Religion:</strong> Islam</p>
      <p><strong>Job:</strong> Student (SSC 2026)</p>
      <p><strong>Social Media:</strong> <a href="https://www.facebook.com/aura.sagor.88" target="_blank">FB: aura.sagor.88</a></p>
      <p><strong>Version:</strong> 1.01</p>
      <p><strong>Thanks for using our product</strong></p>
    </div>
    <button class="back-button" onclick="showGamePage()">Back</button>
  </div>

  <script>
    const board = document.getElementById('board');
    const score = document.getElementById('score');
    const menuPanel = document.getElementById('menuPanel');
    const floatingPanel = document.getElementById('floatingPanel');
    const floatingPanelContent = document.getElementById('floatingPanelContent');
    const winPopup = document.getElementById('winPopup');
    const drawPopup = document.getElementById('drawPopup');
    const closeButton = document.getElementById('closeButton');
    const floatingCloseButton = document.getElementById('floatingCloseButton');
    const gamePage = document.getElementById('gamePage');
    const devPage = document.getElementById('devPage');

    let cells = Array(9).fill('');
    let currentPlayer = 'X';
    let scores = { X: 0, O: 0 };
    let botDifficulty = 'off';
    let gameHistory = [];
    let moveHistory = [];
    let redoStack = [];
    let voices = [];
    let playerColors = { X: 'green', O: 'red' };
    let backgroundTheme = 'light';
    let volume = 50;

    // Load voices when available
    speechSynthesis.onvoiceschanged = () => {
      voices = speechSynthesis.getVoices();
    };

    // Initialize game
    function initGame() {
      updateBoard();
      updateBackground();
      updatePlayerColors();
      voices = speechSynthesis.getVoices();
    }

    // Update background and cross button color
    function updateBackground() {
      const backgrounds = ['light', 'dark', 'blue', 'purple', 'gray'];
      document.body.classList.remove(...backgrounds);
      document.body.classList.add(backgroundTheme);
      menuPanel.classList.toggle('dark', backgroundTheme === 'dark');
      floatingPanel.classList.toggle('dark', backgroundTheme === 'dark');
      const buttonColor = backgroundTheme === 'light' ? '#333' : '#fff';
      closeButton.style.color = buttonColor;
      floatingCloseButton.style.color = buttonColor;
      updateBoard();
    }

    // Update player colors
    function updatePlayerColors() {
      updateBoard();
    }

    // Toggle menu visibility
    function toggleMenu() {
      menuPanel.classList.toggle('open');
      floatingPanel.classList.remove('open');
    }

    // Open floating panel with specific settings
    function openFloatingPanel(setting) {
      floatingPanelContent.innerHTML = '';
      let html = '';
      switch (setting) {
        case 'volume':
          html = `
            <label for="volume">Volume</label>
            <input type="range" id="volume" min="0" max="100" value="${volume}">
          `;
          break;
        case 'xColor':
          html = `
            <label for="xColor">Player X Color</label>
            <select id="xColor">
              <option value="green" ${playerColors.X === 'green' ? 'selected' : ''}>Green</option>
              <option value="red" ${playerColors.X === 'red' ? 'selected' : ''}>Red</option>
              <option value="orange" ${playerColors.X === 'orange' ? 'selected' : ''}>Orange</option>
              <option value="yellow" ${playerColors.X === 'yellow' ? 'selected' : ''}>Yellow</option>
              <option value="blue" ${playerColors.X === 'blue' ? 'selected' : ''}>Blue</option>
            </select>
          `;
          break;
        case 'oColor':
          html = `
            <label for="oColor">Player O Color</label>
            <select id="oColor">
              <option value="red" ${playerColors.O === 'red' ? 'selected' : ''}>Red</option>
              <option value="green" ${playerColors.O === 'green' ? 'selected' : ''}>Green</option>
              <option value="orange" ${playerColors.O === 'orange' ? 'selected' : ''}>Orange</option>
              <option value="yellow" ${playerColors.O === 'yellow' ? 'selected' : ''}>Yellow</option>
              <option value="blue" ${playerColors.O === 'blue' ? 'selected' : ''}>Blue</option>
            </select>
          `;
          break;
        case 'background':
          html = `
            <label for="background">Background</label>
            <select id="background">
              <option value="light" ${backgroundTheme === 'light' ? 'selected' : ''}>Light</option>
              <option value="dark" ${backgroundTheme === 'dark' ? 'selected' : ''}>Dark</option>
              <option value="blue" ${backgroundTheme === 'blue' ? 'selected' : ''}>Blue</option>
              <option value="purple" ${backgroundTheme === 'purple' ? 'selected' : ''}>Purple</option>
              <option value="gray" ${backgroundTheme === 'gray' ? 'selected' : ''}>Gray</option>
            </select>
          `;
          break;
        case 'bot':
          html = `
            <label for="bot">Bot Difficulty</label>
            <select id="bot">
              <option value="off" ${botDifficulty === 'off' ? 'selected' : ''}>Off</option>
              <option value="easy" ${botDifficulty === 'easy' ? 'selected' : ''}>Easy</option>
              <option value="normal" ${botDifficulty === 'normal' ? 'selected' : ''}>Normal</option>
              <option value="difficult" ${botDifficulty === 'difficult' ? 'selected' : ''}>Difficult</option>
            </select>
          `;
          break;
        case 'history':
          html = `
            <label>Game History</label>
            <div class="game-history">
              <ul id="historyList"></ul>
            </div>
            <button onclick="removeHistory()" class="remove-history">Remove History</button>
          `;
          break;
      }
      floatingPanelContent.innerHTML = html;
      menuPanel.classList.remove('open');
      floatingPanel.classList.add('open');

      // Populate history if needed
      if (setting === 'history') {
        const historyList = document.getElementById('historyList');
        historyList.innerHTML = '';
        gameHistory.forEach(result => {
          const li = document.createElement('li');
          li.textContent = result;
          historyList.appendChild(li);
        });
      }

      // Add event listeners for floating panel inputs
      const input = floatingPanelContent.querySelector('input, select');
      if (input) {
        if (setting === 'volume') {
          input.addEventListener('input', () => {
            volume = input.value;
          });
        } else if (setting === 'xColor') {
          input.addEventListener('change', () => {
            playerColors.X = input.value;
            updatePlayerColors();
            closeFloatingPanel();
          });
        } else if (setting === 'oColor') {
          input.addEventListener('change', () => {
            playerColors.O = input.value;
            updatePlayerColors();
            closeFloatingPanel();
          });
        } else if (setting === 'background') {
          input.addEventListener('change', () => {
            backgroundTheme = input.value;
            updateBackground();
            closeFloatingPanel();
          });
        } else if (setting === 'bot') {
          input.addEventListener('change', () => {
            botDifficulty = input.value;
            closeFloatingPanel();
          });
        }
      }
    }

    // Close floating panel
    function closeFloatingPanel() {
      floatingPanel.classList.remove('open');
    }

    // Remove game history
    function removeHistory() {
      gameHistory = [];
      const historyList = document.getElementById('historyList');
      historyList.innerHTML = '';
      speak('Game history cleared');
    }

    // Show developer page
    function showDevPage() {
      gamePage.classList.remove('active');
      devPage.classList.add('active');
    }

    // Show game page
    function showGamePage() {
      devPage.classList.remove('active');
      gamePage.classList.add('active');
    }

    // Speak function
    function speak(text) {
      if (volume === 0) return;
      const utterance = new SpeechSynthesisUtterance(text);
      utterance.volume = volume / 100;
      if (voices.length > 0) {
        utterance.voice = voices.find(v => v.lang.includes('en-US')) || voices[0];
      }
      speechSynthesis.speak(utterance);
    }

    // Show win pop-up
    function showWinPopup(player) {
      winPopup.textContent = `Player ${player} wins!`;
      winPopup.classList.add('show');
      setTimeout(() => {
        winPopup.classList.remove('show');
      }, 3000);
    }

    // Show draw pop-up
    function showDrawPopup() {
      drawPopup.textContent = "It's a draw!";
      drawPopup.classList.add('show');
      setTimeout(() => {
        drawPopup.classList.remove('show');
      }, 3000);
    }

    // Start a new game
    function startNewGame() {
      cells = Array(9).fill('');
      currentPlayer = 'X';
      moveHistory = [];
      redoStack = [];
      board.classList.add('reset');
      setTimeout(() => {
        updateBoard();
        board.classList.remove('reset');
      }, 500);
      toggleMenu();
      speak('New game started');
    }

    // Reset scores and history
    function resetScore() {
      scores = { X: 0, O: 0 };
      gameHistory = [];
      updateScore();
      updateHistory();
      speak('Score and history reset');
    }

    // Update score display
    function updateScore() {
      score.textContent = `X: ${scores.X} | O: ${scores.O}`;
      score.classList.add('update');
      setTimeout(() => score.classList.remove('update'), 500);
    }

    // Update game history
    function updateHistory(result) {
      if (result) {
        gameHistory.push(result);
      }
    }

    // Undo move
    function undoMove() {
      if (moveHistory.length === 0 || checkWin()) return;
      const lastMove = moveHistory.pop();
      redoStack.push({ cells: [...cells], currentPlayer });
      cells = lastMove.cells;
      currentPlayer = lastMove.currentPlayer;
      updateBoard();
      speak(`Undo move, Player ${currentPlayer}'s turn`);
    }

    // Redo move
    function redoMove() {
      if (redoStack.length === 0 || checkWin()) return;
      const nextMove = redoStack.pop();
      moveHistory.push({ cells: [...cells], currentPlayer });
      cells = nextMove.cells;
      currentPlayer = nextMove.currentPlayer;
      updateBoard();
      speak(`Redo move, Player ${currentPlayer}'s turn`);
    }

    // Check for win or draw
    function checkWin() {
      const winPatterns = [
        [0, 1, 2], [3, 4, 5], [6, 7, 8],
        [0, 3, 6], [1, 4, 7], [2, 5, 8],
        [0, 4, 8], [2, 4, 6]
      ];
      const colorMap = {
        green: '#4CAF50',
        red: '#f44336',
        orange: '#ff9800',
        yellow: '#ffeb3b',
        blue: '#2196f3'
      };

      for (const pattern of winPatterns) {
        const [a, b, c] = pattern;
        if (cells[a] && cells[a] === cells[b] && cells[a] === cells[c]) {
          scores[cells[a]]++;
          updateScore();
          highlightWinningCells([a, b, c], colorMap[playerColors[cells[a]]]);
          showWinPopup(cells[a]);
          updateHistory(`Player ${cells[a]} won`);
          speak(`${cells[a]} wins!`);
          setTimeout(startNewGame, 3500);
          return true;
        }
      }

      if (!cells.includes('')) {
        board.classList.add('draw');
        showDrawPopup();
        updateHistory("Draw");
        speak("It's a draw!");
        setTimeout(() => {
          board.classList.remove('draw');
          startNewGame();
        }, 3500);
        return true;
      }

      return false;
    }

    // Highlight winning cells with player's color
    function highlightWinningCells(winningCells, color) {
      board.querySelectorAll('.cell').forEach((cell, index) => {
        if (winningCells.includes(index)) {
          cell.classList.add('winner');
          cell.style.backgroundColor = color;
        }
      });
    }

    // Handle player move
    function handleClick(index) {
      if (cells[index] || checkWin()) return;

      moveHistory.push({ cells: [...cells], currentPlayer });
      redoStack = [];
      cells[index] = currentPlayer;
      const cellElement = board.children[index];
      cellElement.classList.add('clicked', 'ripple');
      updateBoard();
      
      if (!checkWin()) {
        currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
        speak(`Player ${currentPlayer}'s turn`);
        
        if (botDifficulty !== 'off' && currentPlayer === 'O') {
          setTimeout(() => botMove(botDifficulty), 500);
        }
      }
    }

    // Bot move based on difficulty
    function botMove(difficulty) {
      if (difficulty === 'easy') {
        botMoveEasy();
      } else if (difficulty === 'normal') {
        botMoveNormal();
      } else if (difficulty === 'difficult') {
        botMoveDifficult();
      }
    }

    // Easy bot: Random move
    function botMoveEasy() {
      const emptyCells = cells
        .map((cell, index) => cell === '' ? index : null)
        .filter(index => index !== null);

      if (emptyCells.length === 0) return;

      const randomIndex = Math.floor(Math.random() * emptyCells.length);
      cells[emptyCells[randomIndex]] = 'O';
      const cellElement = board.children[emptyCells[randomIndex]];
      cellElement.classList.add('clicked', 'ripple');
      currentPlayer = 'X';
      updateBoard();
      checkWin();
    }

    // Normal bot: Win, block, or random
    function botMoveNormal() {
      const winPatterns = [
        [0, 1, 2], [3, 4, 5], [6, 7, 8],
        [0, 3, 6], [1, 4, 7], [2, 5, 8],
        [0, 4, 8], [2, 4, 6]
      ];

      for (const pattern of winPatterns) {
        const [a, b, c] = pattern;
        if (cells[a] === 'O' && cells[b] === 'O' && cells[c] === '') return makeBotMove(c);
        if (cells[a] === 'O' && cells[c] === 'O' && cells[b] === '') return makeBotMove(b);
        if (cells[b] === 'O' && cells[c] === 'O' && cells[a] === '') return makeBotMove(a);
      }

      for (const pattern of winPatterns) {
        const [a, b, c] = pattern;
        if (cells[a] === 'X' && cells[b] === 'X' && cells[c] === '') return makeBotMove(c);
        if (cells[a] === 'X' && cells[c] === 'X' && cells[b] === '') return makeBotMove(b);
        if (cells[b] === 'X' && cells[c] === 'X' && cells[a] === '') return makeBotMove(a);
      }

      botMoveEasy();
    }

    // Difficult bot: Minimax algorithm
    function botMoveDifficult() {
      const bestMove = minimax(cells, 'O').index;
      makeBotMove(bestMove);
    }

    // Helper function to make bot move
    function makeBotMove(index) {
      moveHistory.push({ cells: [...cells], currentPlayer });
      redoStack = [];
      cells[index] = 'O';
      const cellElement = board.children[index];
      cellElement.classList.add('clicked', 'ripple');
      currentPlayer = 'X';
      updateBoard();
      checkWin();
    }

    // Minimax algorithm
    function minimax(board, player) {
      const winPatterns = [
        [0, 1, 2], [3, 4, 5], [6, 7, 8],
        [0, 3, 6], [1, 4, 7], [2, 5, 8],
        [0, 4, 8], [2, 4, 6]
      ];

      const emptyCells = board
        .map((cell, index) => cell === '' ? index : null)
        .filter(index => index !== null);

      for (const pattern of winPatterns) {
        const [a, b, c] = pattern;
        if (board[a] === 'O' && board[a] === board[b] && board[a] === board[c]) {
          return { score: 10 };
        }
        if (board[a] === 'X' && board[a] === board[b] && board[a] === board[c]) {
          return { score: -10 };
        }
      }
      if (emptyCells.length === 0) {
        return { score: 0 };
      }

      let moves = [];
      for (const index of emptyCells) {
        let move = {};
        move.index = index;
        board[index] = player;

        if (player === 'O') {
          let result = minimax(board, 'X');
          move.score = result.score;
        } else {
          let result = minimax(board, 'O');
          move.score = result.score;
        }

        board[index] = '';
        moves.push(move);
      }

      let bestMove;
      if (player === 'O') {
        let bestScore = -Infinity;
        for (const move of moves) {
          if (move.score > bestScore) {
            bestScore = move.score;
            bestMove = move;
          }
        }
      } else {
        let bestScore = Infinity;
        for (const move of moves) {
          if (move.score < bestScore) {
            bestScore = move.score;
            bestMove = move;
          }
        }
      }

      return bestMove;
    }

    // Update the game board
    function updateBoard() {
      board.innerHTML = '';
      const colorMap = {
        green: '#4CAF50',
        red: '#f44336',
        orange: '#ff9800',
        yellow: '#ffeb3b',
        blue: '#2196f3'
      };
      cells.forEach((cell, index) => {
        const cellElement = document.createElement('div');
        cellElement.className = `cell${backgroundTheme === 'dark' ? ' dark' : ''}`;
        cellElement.textContent = cell;
        if (cell === 'X') {
          cellElement.style.color = colorMap[playerColors.X];
        } else if (cell === 'O') {
          cellElement.style.color = colorMap[playerColors.O];
        }
        cellElement.onclick = () => handleClick(index);
        board.appendChild(cellElement);
      });
      updateScore();
    }

    // Initialize the game
    initGame();
  </script>
</body>
</html>by
