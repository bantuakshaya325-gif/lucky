const puzzleGrid = document.getElementById('puzzle-grid');
const moveCountEl = document.getElementById('move-count');
const timerEl = document.getElementById('timer');
const statusEl = document.getElementById('status');
const resetBtn = document.getElementById('reset-btn');
const difficultyButtons = [...document.querySelectorAll('.difficulty')];

let boardSize = 3;
let board = [];
let moves = 0;
let timer = 0;
let timerId = null;
let started = false;

function createSolvedBoard(size) {
  const total = size * size;
  return Array.from({ length: total }, (_, index) => (index + 1) % total);
}

function isSolvedBoard(currentBoard) {
  for (let i = 0; i < currentBoard.length - 1; i += 1) {
    if (currentBoard[i] !== i + 1) return false;
  }
  return currentBoard[currentBoard.length - 1] === 0;
}

function getEmptyIndex() {
  return board.indexOf(0);
}

function isAdjacent(indexA, indexB) {
  const rowA = Math.floor(indexA / boardSize);
  const colA = indexA % boardSize;
  const rowB = Math.floor(indexB / boardSize);
  const colB = indexB % boardSize;

  return Math.abs(rowA - rowB) + Math.abs(colA - colB) === 1;
}

function renderBoard() {
  puzzleGrid.style.setProperty('--grid-size', boardSize);
  puzzleGrid.innerHTML = '';

  board.forEach((value, index) => {
    const tile = document.createElement('button');
    tile.className = 'tile';

    if (value === 0) {
      tile.classList.add('empty');
      tile.setAttribute('aria-label', 'Empty tile');
      tile.disabled = true;
    } else {
      tile.textContent = String(value);
      tile.setAttribute('aria-label', `Tile ${value}`);
      tile.addEventListener('click', () => moveTile(index));
    }

    puzzleGrid.appendChild(tile);
  });
}

function updateStats() {
  moveCountEl.textContent = String(moves);
  const minutes = String(Math.floor(timer / 60)).padStart(2, '0');
  const seconds = String(timer % 60).padStart(2, '0');
  timerEl.textContent = `${minutes}:${seconds}`;
}

function startTimer() {
  if (timerId) return;

  timerId = setInterval(() => {
    timer += 1;
    updateStats();
  }, 1000);
}

function stopTimer() {
  if (timerId) {
    clearInterval(timerId);
    timerId = null;
  }
}

function resetPuzzleState() {
  stopTimer();
  started = false;
  moves = 0;
  timer = 0;
  board = createSolvedBoard(boardSize);
  updateStats();
  statusEl.textContent = 'Press Shuffle to start.';
  shuffleBoard();
}

function shuffleBoard() {
  let currentBoard = createSolvedBoard(boardSize);
  let emptyIndex = currentBoard.indexOf(0);

  for (let step = 0; step < (boardSize * boardSize * 30); step += 1) {
    const neighbors = [];

    for (let i = 0; i < currentBoard.length; i += 1) {
      if (currentBoard[i] === 0) continue;
      if (isAdjacent(i, emptyIndex)) {
        neighbors.push(i);
      }
    }

    const randomIndex = Math.floor(Math.random() * neighbors.length);
    const selectedTile = neighbors[randomIndex];

    [currentBoard[emptyIndex], currentBoard[selectedTile]] = [currentBoard[selectedTile], currentBoard[emptyIndex]];
    emptyIndex = selectedTile;
  }

  if (isSolvedBoard(currentBoard)) {
    shuffleBoard();
    return;
  }

  board = currentBoard;
  renderBoard();
  statusEl.textContent = 'Puzzle shuffled. Start sliding!';
}

function moveTile(index) {
  const empty = getEmptyIndex();

  if (!isAdjacent(index, empty)) {
    return;
  }

  if (!started) {
    started = true;
    startTimer();
  }

  [board[index], board[empty]] = [board[empty], board[index]];
  moves += 1;
  updateStats();
  renderBoard();

  if (isSolvedBoard(board)) {
    stopTimer();
    statusEl.textContent = `You solved it in ${moves} moves and ${timer} seconds!`;
  }
}

function setBoardSize(size) {
  boardSize = size;
  difficultyButtons.forEach((button) => {
    const isActive = Number(button.dataset.size) === size;
    button.classList.toggle('active', isActive);
  });
  resetPuzzleState();
}

resetBtn.addEventListener('click', () => {
  resetPuzzleState();
});

difficultyButtons.forEach((button) => {
  button.addEventListener('click', () => {
    setBoardSize(Number(button.dataset.size));
  });
});

updateStats();
setBoardSize(boardSize);
