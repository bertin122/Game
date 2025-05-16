<template>
    <div class="container">
      <header>
        <h1>Rwandan Historical Figures</h1>
        <p>Learn about Rwandan history at <span class="museum-name">X Rwandan Museum</span></p>
      </header>
      
      <div class="game-container">
        <div class="level-selector">
          <select v-model="selectedImageIndex" class="select-image" @change="startNewGame">
            <option v-for="(image, index) in images" :key="index" :value="index">
              {{ image.name }}
            </option>
          </select>
          <button @click="unlockAllImages" v-if="unlockedImages.length < images.length">
            Unlock All
          </button>
        </div>
        
        <div class="game-info">
          <div class="moves">Moves: {{ moveCount }}</div>
          <div class="score">Score: {{ score }}</div>
        </div>
        
        <div class="progress-container">
          <div class="progress-text">
            <span>Progress:</span>
            <span>{{ unlockedImages.length }} / {{ images.length }}</span>
          </div>
          <div class="progress-bar">
            <div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
          </div>
        </div>
        
        <div class="puzzle-container">
          <div class="puzzle-grid">
            <div 
              v-for="(piece, index) in puzzle" 
              :key="index"
              :class="['puzzle-piece', piece.isEmpty ? 'empty' : '']"
              @click="movePiece(index)"
            >
              <template v-if="!piece.isEmpty">
                <span class="piece-number">{{ piece.correctPosition + 1 }}</span>
                <div class="piece-image-container" :style="getPieceStyle(piece)"></div>
              </template>
            </div>
          </div>
        </div>
        
        <div class="win-message" v-if="hasWon">
          Congratulations! Puzzle Solved!
        </div>
        
        <div class="controls">
          <button @click="startNewGame">New Game</button>
          <button @click="shufflePuzzle">Shuffle</button>
          <button @click="showSolution" v-if="!showingSolution">Hint</button>
          <button @click="hideSolution" v-if="showingSolution">Hide Hint</button>
        </div>
        
        <div class="historical-fact" v-if="hasWon && currentImage">
          <h3>{{ currentImage.name }}</h3>
          <p>{{ currentImage.fact }}</p>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'SlidingPuzzle',
    data() {
      return {
        // Historical figures data with facts
        images: [
          {
            name: "King Rwabugiri",
            url: require("../assets/placeholders/m1.jpeg"),
            fact: "King Kigeli IV Rwabugiri was one of Rwanda's most powerful kings who ruled from 1853 to 1895. He expanded Rwanda's territory and centralized the kingdom's political structure.",
            unlocked: true
          },
          {
            name: "Queen Rosalie Gicanda",
            url: require("../assets/placeholders/m2.jpeg"),
            fact: "Queen Rosalie Gicanda was the queen consort of Mwami Mutara III of Rwanda. She was a symbol of unity and was tragically killed during the 1994 genocide.",
            unlocked: false
          },
          {
            name: "Fred Rwigema",
            url: require("../assets/placeholders/m3.jpeg"),
            fact: "Fred Rwigema was a military officer who helped found the Rwandan Patriotic Front. He led the RPF's invasion of Rwanda in 1990 but was killed on the second day of fighting.",
            unlocked: false
          },
          {
            name: "Dian Fossey",
            url: require("../assets/placeholders/m4.jpeg"),
            fact: "Though not Rwandan by birth, Dian Fossey made significant contributions to Rwanda through her work studying and protecting mountain gorillas in the Virunga Mountains.",
            unlocked: false
          },
          {
            name: "King Mutara III Rudahigwa",
            url: require("../assets/placeholders/m5.jpeg"),
            fact: "King Mutara III Rudahigwa ruled Rwanda from 1931 to 1959. He abolished forced labor, converted to Catholicism, and worked to modernize Rwanda during his reign.",
            unlocked: false
          },
          {
            name: "Agathe Uwilingiyimana",
            url: require("../assets/placeholders/m6.jpeg"),
            fact: "Agathe Uwilingiyimana was Rwanda's first and only female prime minister. She was a proponent of democracy and was assassinated during the first day of the 1994 genocide.",
            unlocked: false
          },
          {
            name: "Michel Rwagasana",
            url: require("../assets/placeholders/m7.jpeg"),
            fact: "Michel Rwagasana was a Rwandan politician who advocated for independence from colonial rule. He was secretary to King Mutara III and later worked for democratic reforms.",
            unlocked: false
          },
          {
            name: "Josephine Murekezi",
            url: require("../assets/placeholders/m8.jpeg"),
            fact: "Josephine Murekezi was one of Rwanda's first female educators who helped establish schools for girls across the country in the mid-20th century.",
            unlocked: false
          },
          {
            name: "King Yuhi V Musinga",
            url: require("../assets/placeholders/m9.jpeg"),
            fact: "King Yuhi V Musinga ruled Rwanda from 1896 to 1931. He resisted European colonization but was eventually deposed by Belgian authorities for refusing to convert to Christianity.",
            unlocked: false
          },
          {
            name: "Félicité Niyitegeka",
            url: require('../assets/placeholders/m10.jpeg'),
            fact: "Félicité Niyitegeka was a Rwandan nun who saved many lives during the 1994 genocide by sheltering Tutsis. She refused to abandon those she was protecting and was killed for her heroism.",
            unlocked: false
          }
        ],
        puzzle: [],
        emptyIndex: 8, // Initially, the 9th position (index 8) is empty
        moveCount: 0,
        score: 0,
        hasWon: false,
        selectedImageIndex: 0,
        showingSolution: false,
        solutionTimer: null
      };
    },
    computed: {
      currentImage() {
        return this.images[this.selectedImageIndex];
      },
      unlockedImages() {
        return this.images.filter(img => img.unlocked);
      },
      progressPercentage() {
        return (this.unlockedImages.length / this.images.length) * 100;
      }
    },
    methods: {
      // Initialize the puzzle
      initializePuzzle() {
        this.puzzle = [];
        for (let i = 0; i < 9; i++) {
          this.puzzle.push({
            currentPosition: i,
            correctPosition: i,
            isEmpty: i === 8, // The 9th position is empty
            x: i % 3,
            y: Math.floor(i / 3)
          });
        }
      },
      
      // Shuffle the puzzle pieces
      shufflePuzzle() {
        this.hideSolution();
        this.moveCount = 0;
        this.hasWon = false;
        
        // Create a solvable puzzle by making random valid moves
        let randomMoves = 100; // Number of random moves to make
        while (randomMoves > 0) {
          let possibleMoves = this.getValidMoves();
          if (possibleMoves.length > 0) {
            let randomMove = possibleMoves[Math.floor(Math.random() * possibleMoves.length)];
            this.swapPieces(randomMove, this.emptyIndex, false);
          }
          randomMoves--;
        }
      },
      
      // Get valid moves from the current position
      getValidMoves() {
        const validMoves = [];
        const emptyX = this.emptyIndex % 3;
        const emptyY = Math.floor(this.emptyIndex / 3);
        
        // Check left, right, up, down
        if (emptyX > 0) validMoves.push(this.emptyIndex - 1); // Left
        if (emptyX < 2) validMoves.push(this.emptyIndex + 1); // Right
        if (emptyY > 0) validMoves.push(this.emptyIndex - 3); // Up
        if (emptyY < 2) validMoves.push(this.emptyIndex + 3); // Down
        
        return validMoves;
      },
      
      // Move a piece if it's adjacent to the empty space
      movePiece(index) {
        if (this.hasWon) return;
        
        const x1 = index % 3;
        const y1 = Math.floor(index / 3);
        const x2 = this.emptyIndex % 3;
        const y2 = Math.floor(this.emptyIndex / 3);
        
        // Check if the piece is adjacent to the empty space
        const isAdjacent = 
          (Math.abs(x1 - x2) === 1 && y1 === y2) || 
          (Math.abs(y1 - y2) === 1 && x1 === x2);
        
        if (isAdjacent) {
          this.swapPieces(index, this.emptyIndex, true);
          this.checkWinCondition();
        }
      },
      
      // Swap two pieces
      swapPieces(index1, index2, countMove = true) {
        const temp = {...this.puzzle[index1]};
        this.puzzle[index1] = {...this.puzzle[index2]};
        this.puzzle[index2] = temp;
        
        // Update positions
        this.puzzle[index1].currentPosition = index1;
        this.puzzle[index2].currentPosition = index2;
        
        // Update empty index
        this.emptyIndex = index1;
        
        if (countMove) {
          this.moveCount++;
        }
        
        this.hideSolution();
      },
      
      // Check if the puzzle is solved
      checkWinCondition() {
        const solved = this.puzzle.every((piece, index) => {
          return piece.correctPosition === index;
        });
        
        if (solved) {
          this.hasWon = true;
          // Calculate score based on moves
          const baseScore = 1000;
          const movesPenalty = this.moveCount * 5;
          this.score += Math.max(baseScore - movesPenalty, 100);
          
          // Unlock the next image if available
          this.unlockNextImage();
        }
      },
      
      // Unlock the next image
      unlockNextImage() {
        const currentIndex = this.selectedImageIndex;
        let nextImageUnlocked = false;
        
        // Find the next locked image and unlock it
        for (let i = 0; i < this.images.length; i++) {
          const nextIndex = (currentIndex + i + 1) % this.images.length;
          if (!this.images[nextIndex].unlocked) {
            this.images[nextIndex].unlocked = true;
            nextImageUnlocked = true;
            break;
          }
        }
        
        // If all images are already unlocked, add a bonus score
        if (!nextImageUnlocked) {
          this.score += 500;
        }
      },
      
      // Start a new game with the selected image
      startNewGame() {
        this.hideSolution();
        this.initializePuzzle();
        this.shufflePuzzle();
        this.hasWon = false;
        this.moveCount = 0;
      },
      
      // Show solution temporarily
      showSolution() {
        this.showingSolution = true;
        
        // Hide solution after 3 seconds
        this.solutionTimer = setTimeout(() => {
          this.hideSolution();
        }, 3000);
      },
      
      // Hide solution
      hideSolution() {
        this.showingSolution = false;
        if (this.solutionTimer) {
          clearTimeout(this.solutionTimer);
          this.solutionTimer = null;
        }
      },
      
      // Unlock all images for demo purposes
      unlockAllImages() {
        this.images.forEach(image => {
          image.unlocked = true;
        });
      },
      
      // Get CSS style for puzzle piece
      getPieceStyle(piece) {
        if (piece.isEmpty) return {};
        
        const imgWidth = 300;
        const imgHeight = 300;
        const pieceWidth = imgWidth / 3;
        const pieceHeight = imgHeight / 3;
        
        // Calculate the background position based on the correct position
        const x = (piece.correctPosition % 3) * -pieceWidth;
        const y = Math.floor(piece.correctPosition / 3) * -pieceHeight;
        
        return {
          width: '100%',
          height: '100%',
          backgroundImage: `url(${this.currentImage.url})`,
          backgroundSize: '300% 300%',
          backgroundPosition: `${x}px ${y}px`
        };
      }
    },
    mounted() {
      this.initializePuzzle();
      this.shufflePuzzle();
    }
  };
  </script>
  
  <style scoped>
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background-color: #f5f5f5;
    margin: 0;
    padding: 20px;
    color: #333;
  }
  .container {
    max-width: 800px;
    margin: 0 auto;
    background-color: #fff;
    padding: 30px;
    border-radius: 10px;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  }
  header {
    text-align: center;
    margin-bottom: 30px;
  }
  h1 {
    color: #2c3e50;
    margin-bottom: 10px;
  }
  .museum-name {
    color: #e74c3c;
    font-weight: bold;
  }
  .game-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 20px;
  }
  .puzzle-container {
    position: relative;
    width: 300px;
    height: 300px;
    border: 3px solid #2c3e50;
    border-radius: 5px;
    overflow: hidden;
  }
  .puzzle-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(3, 1fr);
    width: 100%;
    height: 100%;
    gap: 1px;
    background-color: #34495e;
  }
  .puzzle-piece {
    position: relative;
    width: 100%;
    height: 100%;
    background-color: #3498db;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 20px;
    font-weight: bold;
    color: white;
    cursor: pointer;
    transition: all 0.2s ease;
    overflow: hidden;
  }
  .puzzle-piece:hover {
    background-color: #2980b9;
  }
  .puzzle-piece.empty {
    background-color: #34495e;
    cursor: default;
  }
  .piece-number {
    position: absolute;
    top: 5px;
    left: 5px;
    background-color: rgba(0,0,0,0.5);
    color: white;
    width: 22px;
    height: 22px;
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 14px;
  }
  .controls {
    display: flex;
    gap: 15px;
    margin-bottom: 20px;
  }
  button {
    background-color: #3498db;
    color: white;
    border: none;
    padding: 10px 15px;
    border-radius: 5px;
    cursor: pointer;
    font-weight: bold;
    transition: background-color 0.2s;
  }
  button:hover {
    background-color: #2980b9;
  }
  .level-selector {
    margin-bottom: 20px;
  }
  .select-image {
    padding: 10px;
    border-radius: 5px;
    border: 1px solid #ccc;
    width: 250px;
    margin-right: 10px;
  }
  .game-info {
    display: flex;
    justify-content: space-between;
    width: 300px;
    margin-bottom: 15px;
  }
  .score, .moves {
    font-size: 18px;
    font-weight: bold;
  }
  .historical-fact {
    margin-top: 20px;
    padding: 15px;
    background-color: #f8f9fa;
    border-left: 4px solid #3498db;
    border-radius: 5px;
  }
  .win-message {
    text-align: center;
    font-size: 24px;
    font-weight: bold;
    color: #27ae60;
    margin: 20px 0;
    animation: pulse 1.5s infinite;
  }
  .progress-container {
    width: 300px;
    margin-top: 10px;
  }
  .progress-text {
    display: flex;
    justify-content: space-between;
    margin-bottom: 5px;
  }
  .progress-bar {
    height: 10px;
    background-color: #ecf0f1;
    border-radius: 5px;
    overflow: hidden;
  }
  .progress-fill {
    height: 100%;
    background-color: #27ae60;
    transition: width 0.3s ease;
  }
  @keyframes pulse {
    0% { transform: scale(1); }
    50% { transform: scale(1.05); }
    100% { transform: scale(1); }
  }
  @media (max-width: 500px) {
    .puzzle-container {
      width: 270px;
      height: 270px;
    }
    .game-info, .progress-container {
      width: 270px;
    }
  }
  </style>