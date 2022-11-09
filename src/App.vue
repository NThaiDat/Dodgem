<script>
export default {
  name: 'App',
  data() {
    return {
      isPvP: true,
      showModal: false,        // Có hiển thị bảng chọn lượt hay không
      turn: undefined,        // Lượt chơi, W ~ quân trắng đi, B ~ quân đen được đi
      depth: 10,              // Độ sâu cây trò chơi
      matrixSize: 3,          // Diện tích bàn chơi
      blackPoints: [          // Ma trận điểm cho quân đen
        [-10, -25, -40],
        [-5, -20, -35],
        [0, -15, -30],
      ],
      whitePoints: [          // Ma trận điểm cho quân trắng
        [30, 35, 40],
        [15, 20, 25],
        [0, 5, 10],
      ],
      state: [                // Trạng thái trò chơi, -1 ~ quân đen, 1 ~ quân trắng
        [-1, 0, 0],
        [-1, 0, 0],
        [0, 1, 1],
      ],
      pick: {},               // Tọa độ người chơi chọn để di chuyển quân
    }
  },
  watch: {
    /**
     * Mỗi khi có sự thay đổi lượt chơi
     * -> kiểm tra xem có đủ điều kiện kết thúc trò chơi -> thông báo
     * -> nếu lượt tiếp theo là máy -> cho máy tính toán di chuyển
     * */

    turn(value) {
      const checkEndGame = this.checkEndGame()
      if (checkEndGame == 0)
        this.notifyEndGame(false)
      else if (checkEndGame == 1)
        this.notifyEndGame()
      else if (value === 'B') {
        if (this.isPvP) return
        else {
          this.botMove();
        }
      }
    }
  },
  methods: {
    resetGame() {
      this.state = [
        [-1, 0, 0],
        [-1, 0, 0],
        [0, 1, 1],
      ]
      this.turn = 'W'
    },
    ChangeMode(Mode) {
      if (Mode == 'PvP') {
        this.isPvP = true;
        this.resetGame()
      } else {
        this.showModal = true;
        this.isPvP = false;
        this.resetGame()
      }
      console.log(this.isPvP)
    },
    isBlack(x, y) {
      return this.state[y][x] === -1;
    },
    isWhite(x, y) {
      return this.state[y][x] === 1;
    },
    switchTurn() {
      this.turn = this.turn === 'W' ? 'B' : 'W';
    },
    botMove() {
      this.state = this.minimax(this.depth)
      this.switchTurn()
    },
    playerPick(x, y) {
      if (this.isPvP === true) {
        if (this.state[y][x] === 1 && this.turn === 'W') {
          this.pick = {x, y}
        } else if (this.state[y][x] === -1 && this.turn === 'B') {
          this.pick = {x, y}
        }
      } else {
        if (this.state[y][x] === 1 && this.turn === 'W') {
          this.pick = {x, y}
        }
      }
    },
    playerMove(x, y) {
      if (this.turn === 'W') {
        // loại bỏ tình huống đi chéo
        if (this.pick.x !== x && this.pick.y !== y) {
          return;
        }
        // loại bỏ tình huống đi lùi hoặc đi quá 1 ô
        if (Math.abs(this.pick.x - x) > 1 || y - this.pick.y < -1 || y - this.pick.y > 0) {
          return;
        }
      } else {
        // loại bỏ tình huống đi chéo
        if (this.pick.x !== x && this.pick.y !== y) {
          return;
        }
        // loại bỏ tình huống đi lùi hoặc đi quá 1 ô
        if (x - this.pick.x < 0 || Math.abs(this.pick.x - x) > 1 || Math.abs(this.pick.y - y) > 1) {
          return;
        }
      }

      this.state[this.pick.y][this.pick.x] = 0
      if (this.turn === 'W') {
        this.state[y][x] = 1
      } else {
        this.state[y][x] = -1
      }
      this.pick = {};
      this.switchTurn();
    },
    playerMoveEdge() {
      // loại bỏ tình huống chưa đến rìa bản đồ đã ra ngoài bản đồ
      if (this.turn === 'W' && this.pick.y !== 0) {
        return;
      } else {
        if (this.turn === 'B' && this.pick.x !== 2) {
          return;
        }
      }
      this.state[this.pick.y][this.pick.x] = 0
      this.pick = {};
      this.switchTurn();
    },
    findNextStates(state, turn) {
      let states = [];
      for (let i = 0; i < this.matrixSize; i++) {
        for (let j = 0; j < this.matrixSize; j++) {
          if (turn === 'W' && state[i][j] === 1) {
            // trắng -> đi lên
            if (i === 0 || (i >= 1 && state[i - 1][j] === 0)) {
              const next = state.map(arr => arr.slice())
              next[i][j] = 0
              if (i !== 0)
                next[i - 1][j] = 1
              states.push(next)
            }
            // trắng -> sang trái
            if (j >= 1 && state[i][j - 1] === 0) {
              const next = state.map(arr => arr.slice())
              next[i][j] = 0
              next[i][j - 1] = 1
              states.push(next)
            }
            // trắng -> sang phải
            if (j + 1 < this.matrixSize && [i][j + 1] === 0) {
              const next = state.map(arr => arr.slice())
              next[i][j] = 0
              next[i][j + 1] = 1
              states.push(next)
            }
          } else if (turn === 'B' && state[i][j] === -1) {
            // đen -> sang phải
            if (j === this.matrixSize - 1 || (j + 1 < this.matrixSize && state[i][j + 1] === 0)) {
              const next = state.map(arr => arr.slice())
              next[i][j] = 0
              if (j !== this.matrixSize - 1)
                next[i][j + 1] = -1
              states.push(next)
            }
            // đen -> đi lên
            if (i >= 1 && state[i - 1][j] === 0) {
              const next = state.map(arr => arr.slice())
              next[i][j] = 0
              next[i - 1][j] = -1
              states.push(next)
            }
            // đen -> đi xuống
            if (i + 1 < this.matrixSize && state[i + 1][j] === 0) {
              const next = state.map(arr => arr.slice())
              next[i][j] = 0
              next[i + 1][j] = -1
              states.push(next)
            }
          }
        }
      }
      return states;
    },
    eval(state) {
      let numWhite = 2;
      let numBlack = 2;
      let val = 0;
      for (let i = 0; i < this.matrixSize; i++) {
        for (let j = 0; j < this.matrixSize; j++) {
          if (state[i][j] === 1) {
            numWhite--
            val += this.whitePoints[i][j];
            // kiểm tra đen chặn trực tiếp trắng
            if (i - 1 >= 0 && state[i - 1][j] === -1)
              val -= 40;
            // kiểm tra đen chặn gián tiếp trắng
            for (let s = i - 2; s >= 0; s--) {
              if (state[s][j] === -1)
                val -= 30;
            }
          } else if (state[i][j] === -1) {
            numBlack--
            val += this.blackPoints[i][j];
            // kiểm tra trắng chặn trực tiếp đen
            if (j + 1 < this.matrixSize && state[i][j + 1] === 1)
              val += 40;
            // kiểm tra trắng chặn gián tiếp đen
            for (let s = j + 2; s < this.matrixSize; s++) {
              if (state[i][s] === 1)
                val += 30;
            }
          }
        }
      }
      // trạng thái có quân ngoài bản đồ +/- 50 điểm cho mỗi quân
      val += numWhite * 50
      val -= numBlack * 50
      return val;
    },
    isEndState(state) {
      let numBlack = 0
      let numWhite = 0
      for (let i = 0; i < this.matrixSize; i++) {
        for (let j = 0; j < this.matrixSize; j++) {
          if (state[i][j] === 1)
            numWhite++
          else if (this.state[i][j] === -1)
            numBlack++
        }
      }
      return numBlack === 0 || numWhite === 0
    },
    maxVal(state, depth) {
      if (depth === 0 || this.isEndState(state))
        return this.eval(state)
      else {
        const nextStates = this.findNextStates(state, 'W')
        const minVals = nextStates.map(s => this.minVal(s, depth - 1))
        return Math.max(...minVals)
      }
    },
    minVal(state, depth) {
      if (depth === 0 || this.isEndState(state))
        return this.eval(state)
      else {
        const nextStates = this.findNextStates(state, 'B')
        const maxVals = nextStates.map(s => this.maxVal(s, depth - 1))
        return Math.min(...maxVals)
      }
    },
    minimax(depth) {
      let min = 9999
      let chosenState = null
      const nextStates = this.findNextStates(this.state, 'B')
      nextStates.forEach(s => {
        const maxVal = this.maxVal(s, depth - 1)
        if (min >= maxVal) {
          min = maxVal
          chosenState = s
        }
      })
      return chosenState
    },
    /**
     * 0 ~ máy thắng
     * 1 ~ bạn thắng
     * -1 ~ chưa kết thúc
     **/
    checkEndGame() {
      // số quân đen/trắng trên bàn
      let numBlack = 0
      let numWhite = 0
      // số quân đen trắng bị chặn không thể đi
      let deadBlack = 0
      let deadWhite = 0
      for (let i = 0; i < this.matrixSize; i++) {
        for (let j = 0; j < this.matrixSize; j++) {
          if (this.state[i][j] === 1) {
            if ((this.state?.[i - 1]?.[j] ?? 0) === -1 && (this.state?.[i]?.[j + 1] ?? -1) === -1 && (this.state?.[i]?.[j - 1] ?? -1) === -1)
              deadWhite++
            numWhite++
          } else if (this.state[i][j] === -1) {
            if ((this.state?.[i - 1]?.[j] ?? 1) === 1 && (this.state?.[i + 1]?.[j] ?? 1) === 1 && (this.state?.[i]?.[j + 1] ?? 0) === 1)
              deadBlack++
            numBlack++
          }
        }
      }
      // khi số quân đen/trắng == 0 hoặc 1 bên không thể di chuyển -> kết thúc
      if (numBlack === 0 || (numWhite > 0 && deadWhite === numWhite)) {
        return 0
      }
      if (numWhite === 0 || (numBlack > 0 && deadBlack === numBlack)) {
        return 1
      }
      return -1
    },
    notifyEndGame(win = true) {
      if (win) {
        if (this.isPvP) {
          alert('Quân trắng đã thắng!')
        } else {
          alert('Bạn đã thắng!')
        }
      } else if (this.isPvP) {
        alert('Quân đen đã thắng!')
      } else {
        alert('Bạn đã thua!')
      }
    },
  },

}
</script>

<template>
  <h2 class="text-center p-2 textHeader" style="font-size: 50px; font-family: 'Lobster', cursive;
">Dodgem</h2>
  <div class="buttonList">
    <button class="btnGame" @click="ChangeMode('PvP')" :class="isPvP?'text-red-500':'text-white'">Chơi 2 người</button>
    <button class="btnGame" @click="ChangeMode('PvE')" :class="isPvP?'text-white':'text-red-500'">Chơi với máy</button>
  </div>
  <p class="text-center text-2xl pt-6">Lượt của quân: {{ turn === "B" ? "Đen" : "Trắng" }}</p>
  <div id="game">
    <div class="map-wrapper">
      <div class="map-space" @click="playerMoveEdge()"></div>
      <div class="map">
        <div v-for="(_y, y) in matrixSize" class="d-flex gap-1">
          <div v-for="(_x, x) in matrixSize" class="map_block">
            <div v-if="isWhite(x, y)" @click="playerPick(x, y)" class="circle"></div>
            <div v-else-if="isBlack(x, y)" @click="playerPick(x, y)" class="circle circle-black"></div>
            <div v-else @click="playerMove(x, y)" class="w-100 h-100"></div>
          </div>
        </div>
        <div class="map-space" @click="playerMoveEdge()"
             style="position:absolute ;width: 25%;height: 50%;right: 210px"></div>
      </div>

    </div>
    <div v-if="showModal" class="modal fade show d-block" data-bs-backdrop="static" id="exampleModal" tabindex="-1"
         aria-labelledby="exampleModalLabel" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-body text-center">
            <h5>Chọn lượt chơi của bạn</h5>
          </div>
          <div class="modal-footer d-flex justify-content-center">
            <button type="button" class="btn bg-blue-400 text-white" @click="turn = 'W'; showModal = false">Đi
              trước
            </button>
            <button type="button" class="btn bg-red-500 text-white" @click="turn = 'B'; showModal = false">Đi
              sau
            </button>
          </div>
        </div>
      </div>
    </div>
    <div v-if="showModal" class="modal-backdrop fade show"></div>
  </div>
</template>

<style scoped>
html,
body {
  height: 100%;
}

#game {
  height: 100%;
}

.map-wrapper {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.map-space {
  flex-grow: 1;
  width: 100%;
  height: 100px;
}

.map {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.map_block {
  width: 7rem;
  height: 7rem;
  border: 2px solid black;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
}

.circle {
  width: 5rem;
  height: 5rem;
  border-radius: 50%;
  border: 3px solid black;
}

.circle-black {
  background-color: black;
}

.modal-dialog {
  height: calc(100% - 1.75rem * 2);
  display: flex;
  align-items: center;
}

.buttonList {
  display: flex;
  justify-content: space-evenly;
  margin-top: 3%;
}


.btnGame {
  padding: 0.6em 2em;
  border: none;
  outline: none;
  cursor: pointer;
  position: relative;
  z-index: 0;
  border-radius: 10px;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
}
.textHeader:before{
  content: "Dodgem";
  background: linear-gradient(
      45deg,
      #ff0000,
      #ff7300,
      #fffb00,
      #48ff00,
      #00ffd5,
      #002bff,
      #7a00ff,
      #ff00c8,
      #ff0000
  );
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  position: absolute;
  left: -2px;
  background-size: 400%;
  z-index: -1;
  filter: blur(5px);
  -webkit-filter: blur(5px);
  width: calc(100% + 4px);
  height: calc(100% + 4px);
  animation: glowing-btn 20s linear infinite;
  transition: opacity 0.3s ease-in-out;
  border-radius: 10px;
}
.btnGame:before {
  color: transparent;
  content: "";
  background: linear-gradient(
      45deg,
      #ff0000,
      #ff7300,
      #fffb00,
      #48ff00,
      #00ffd5,
      #002bff,
      #7a00ff,
      #ff00c8,
      #ff0000
  );
  position: absolute;
  top: -2px;
  left: -2px;
  background-size: 400%;
  z-index: -1;
  filter: blur(5px);
  -webkit-filter: blur(5px);
  width: calc(100% + 4px);
  height: calc(100% + 4px);
  animation: glowing-btn 20s linear infinite;
  transition: opacity 0.3s ease-in-out;
  border-radius: 10px;
}

@keyframes glowing-btn {
  0% {
    background-position: 0 0;
  }
  50% {
    background-position: 400% 0;
  }
  100% {
    background-position: 0 0;
  }
}

.btnGame:after {
  z-index: -1;
  content: "";
  position: absolute;
  width: 100%;
  height: 100%;
  background: #222;
  left: 0;
  top: 0;
  border-radius: 10px;
}

</style>
