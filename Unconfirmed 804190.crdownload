
const canvas = document.getElementById('puzzleCanvas');
const ctx = canvas.getContext('2d');
const imageInput = document.getElementById('imageInput');
const difficultySelect = document.getElementById('difficulty');
const startButton = document.getElementById('startGame');

let pieces = [];
let img = null;
let gridSize = 5;
let pieceWidth;
let pieceHeight;
let selectedPiece = null;
let isGameComplete = false;
let message = '';

canvas.width = 600;
canvas.height = 600;

function showMessage(text) {
  message = text;
  ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
  ctx.fillRect(0, canvas.height/2 - 30, canvas.width, 60);
  ctx.fillStyle = 'white';
  ctx.font = '24px Arial';
  ctx.textAlign = 'center';
  ctx.fillText(text, canvas.width/2, canvas.height/2 + 10);
}

class Piece {
  constructor(x, y, correctX, correctY, width, height, index) {
    this.x = x;
    this.y = y;
    this.correctX = correctX;
    this.correctY = correctY;
    this.width = width;
    this.height = height;
    this.index = index;
  }

  draw() {
    const sx = (this.index % gridSize) * pieceWidth;
    const sy = Math.floor(this.index / gridSize) * pieceHeight;
    ctx.drawImage(img, sx, sy, pieceWidth, pieceHeight, this.x, this.y, this.width, this.height);
    ctx.strokeRect(this.x, this.y, this.width, this.height);
  }
}

function loadImage(file) {
  return new Promise((resolve) => {
    const reader = new FileReader();
    reader.onload = (e) => {
      img = new Image();
      img.onload = () => resolve();
      img.src = e.target.result;
    };
    reader.readAsDataURL(file);
  });
}

function initializePieces() {
  pieces = [];
  pieceWidth = canvas.width / gridSize;
  pieceHeight = canvas.height / gridSize;

  let positions = Array.from({ length: gridSize * gridSize }, (_, i) => i);
  positions = shuffleArray(positions);

  for (let i = 0; i < gridSize * gridSize; i++) {
    const correctX = (i % gridSize) * pieceWidth;
    const correctY = Math.floor(i / gridSize) * pieceHeight;
    const randomPos = positions[i];
    const x = (randomPos % gridSize) * pieceWidth;
    const y = Math.floor(randomPos / gridSize) * pieceHeight;
    pieces.push(new Piece(x, y, correctX, correctY, pieceWidth, pieceHeight, i));
  }
}

function shuffleArray(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
  return array;
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  pieces.forEach(piece => piece.draw());
  if (message) {
    showMessage(message);
  }
}

function isComplete() {
  return pieces.every(piece => 
    piece.x === piece.correctX && piece.y === piece.correctY
  );
}

function resizeImage() {
  const scale = Math.min(canvas.width / img.width, canvas.height / img.height);
  const newWidth = img.width * scale;
  const newHeight = img.height * scale;
  
  const tempCanvas = document.createElement('canvas');
  tempCanvas.width = canvas.width;
  tempCanvas.height = canvas.height;
  const tempCtx = tempCanvas.getContext('2d');
  
  tempCtx.drawImage(img, 
    (canvas.width - newWidth) / 2, 
    (canvas.height - newHeight) / 2, 
    newWidth, 
    newHeight
  );
  
  img = tempCanvas;
}

canvas.addEventListener('mousedown', (e) => {
  const rect = canvas.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;

  if (isGameComplete) return;
  
  // Reverse the array to select the top-most piece
  [...pieces].reverse().some(piece => {
    if (x > piece.x && x < piece.x + piece.width &&
        y > piece.y && y < piece.y + piece.height) {
      selectedPiece = piece;
      return true;
    }
    return false;
  });
});

canvas.addEventListener('mousemove', (e) => {
  if (!selectedPiece) return;
  
  const rect = canvas.getBoundingClientRect();
  selectedPiece.x = e.clientX - rect.left - pieceWidth / 2;
  selectedPiece.y = e.clientY - rect.top - pieceHeight / 2;
  draw();
});

canvas.addEventListener('mouseup', () => {
  if (!selectedPiece) return;

  // Snap to grid
  const gridX = Math.round(selectedPiece.x / pieceWidth) * pieceWidth;
  const gridY = Math.round(selectedPiece.y / pieceHeight) * pieceHeight;

  // Check if space is occupied
  const occupiedPiece = pieces.find(piece => 
    piece !== selectedPiece && piece.x === gridX && piece.y === gridY
  );

  if (occupiedPiece) {
    // Swap positions between selected piece and occupied piece
    const tempX = selectedPiece.x;
    const tempY = selectedPiece.y;
    selectedPiece.x = occupiedPiece.x;
    selectedPiece.y = occupiedPiece.y;
    occupiedPiece.x = tempX;
    occupiedPiece.y = tempY;
  } else {
    // Move selected piece to new position if space is empty
    selectedPiece.x = gridX;
    selectedPiece.y = gridY;
  }

  selectedPiece = null;
  draw();
  
  sounds.switch.play();

  if (isComplete()) {
    isGameComplete = true;
    message = 'Congratulations! Puzzle completed!';
    sounds.win.play();
    draw();
  }
});

startButton.addEventListener('click', async () => {
  if (!imageInput.files[0]) {
    alert('Please select an image first!');
    return;
  }
  sounds.start.play();

  const difficulty = difficultySelect.value;
  gridSize = difficulty === '2' ? 3 : difficulty === '3' ? 5 : difficulty === '4' ? 10 : 20;
  message = '';
  isGameComplete = false;
  
  await loadImage(imageInput.files[0]);
  resizeImage();
  initializePieces();
  draw();
});
