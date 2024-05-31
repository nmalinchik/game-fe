<template>
  <div class="container">
    <div v-if="status !== 'IN_PROGRESS'" class="status">
      <div v-if="gameFinished">Game ended with status: {{ status }}</div>
      <div v-if="winner" class="winner">Winner: {{ winner }}</div>
    </div>
    <div class="board">
      <div v-for="(row, rowIndex) in board" :key="rowIndex" class="row">
        <div v-for="(cell, cellIndex) in row" :key="cellIndex" class="cell">
          {{ cell }}
        </div>
      </div>
    </div>
    <div v-if="gameFinished" class="game-over-container">
      <div class="game-over">GAME OVER</div>
      <button class="start-new-game" @click="startNewGame">Start New Game</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      board: [['', '', ''], ['', '', ''], ['', '', '']],
      intervalId: null,
      status: null,
      gameFinished: false,
      winner: null,
    };
  },
  created() {
    console.log('Component created');
    this.startGameStatusPolling();
  },
  methods: {
    async getGameStatus() {
      try {
        const response = await axios.get('http://localhost:8080/game');
        console.log('Game status:', response.data);
        this.board = response.data.board;
        if (response.data.status !== 'IN_PROGRESS') {
          this.gameFinished = true;
          this.status = response.data.status;
          this.winner = response.data.winner;
          clearInterval(this.intervalId);
        }
      } catch (error) {
        console.error(error);
      }
    },
    startGameStatusPolling() {
      this.intervalId = setInterval(this.getGameStatus, 1000);
    },
    async startNewGame() {
      try {
        await axios.post('http://localhost:8080/game/new');
        this.gameFinished = false;
        this.board = [['', '', ''], ['', '', ''], ['', '', '']];
        console.log('New game started');
        setTimeout(() => {
          this.startGameStatusPolling();
          this.status = null;
          this.winner = null;
        }, 1000);
      } catch (error) {
        console.error(error);
      }
    }
  },
  beforeDestroy() {
    clearInterval(this.intervalId);
  }
};
</script>

<style>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 350px;
  margin: 0 auto;
  background-color: #ffffff; /* Светлый фон */
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* Тень для контейнера */
}

.board {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
  margin-top: 20px;
}

.row {
  display: contents; /* Обеспечиваем выравнивание рядов и клеток в сетке */
}

.cell {
  width: 100px;
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #000; /* Черные границы */
  font-size: 24px;
  background-color: #f0f0f0; /* Светло-серый фон для клеток */
  border-radius: 8px; /* Слегка закругленные углы */
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* Тень для клеток */
}

.status {
  text-align: center;
  margin-bottom: 20px;
  font-size: 18px;
  font-weight: bold;
  color: #333;
}

.winner {
  color: green;
  font-size: 20px;
  margin-top: 10px;
  font-weight: bold;
}

.game-over-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
}

.game-over {
  text-align: center;
  font-size: 24px;
  color: red;
  margin-bottom: 10px;
  font-weight: bold;
}

.start-new-game {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
  text-align: center;
  border-radius: 12px; /* Закругленные углы */
  transition: background-color 0.3s;
}

.start-new-game:hover {
  background-color: #45a049;
}
</style>
