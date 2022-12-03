<template>
  <div class="page">
  <div class="content">
    <h1>15 puzzles</h1>
    <div :class="{ fieldIndikator: true, good: isTrueField }"></div>
    <div :class="{ wrapper: true, trueField: isTrueField }">
      <div :class="{ container: true, trueField: isTrueField }" ref="container">
        <MyPuzzle
          v-for="puzzle in puzzles"
          :puzzle="puzzle"
          @pointerdown="onMousedown($event, puzzle)"
          @dragstart="onDragstart"
          :style="{ transitionDuration: puzzlesTransition + 's' }"
        >
        </MyPuzzle>
      </div>
    </div>
    <div class="btns">
      <button class="defaultBut active" @click="defaultState">default</button>
      <button
        :class="{ defaultBut: true, active: isShuffleButtonActive }"
        @click="isShuffleButtonActive ? shuffle() : null"
        title="shuffle as many times as necessary"
      >
        shuffle
      </button>
      <div :class="{ popUpCool: true, active: true }">
        <div :class="{ popUpCool__content: true, active: isTrueField }"></div>
      </div>
    </div>
  </div>
    
  </div>
</template>

<script>
import MyPuzzle from "@/components/gameTag/MyPuzzle.vue";

function makePuzzlesDefault() {
  let puzzlesDefault = [];
  for (let i = 1; i <= 15; i++) {
    let row = Math.floor((i - 1) / 4);
    let col = (i - 1) % 4;
    let puzzle = {};
    puzzle.number = i;
    puzzle.row = row;
    puzzle.col = col;
    puzzlesDefault.push(puzzle);
  }
  return puzzlesDefault;
}

const puzzlesDefault = makePuzzlesDefault();
const puzzlesTransitionDefault = 0.1;

function randomInteger(min, max) {
  let rand = min + Math.random() * (max + 1 - min);
  return Math.floor(rand);
}

function isTrueField(field) {
  let isTrueField = true;
  for (let row = 0; row <= 3; row++) {
    for (let col = 0; col <= 3; col++) {
      let trueNumber = col + 1 + row * 4 !== 16 ? col + 1 + row * 4 : 0;
      let number = field[row][col];
      if (trueNumber !== number) isTrueField = false;
    }
  }
  return isTrueField;
}

export default {
  components: { MyPuzzle },

  data() {
    return {
      field: [[], [], [], []],
      puzzles: [],
      puzzlesTransitionDefault: puzzlesTransitionDefault,
      puzzlesTransition: puzzlesTransitionDefault,
      isShuffleButtonActive: true,
      isTrueField: false,
    };
  },

  methods: {
    isEmptyNearby(row, col) {
      if (row > 0 && this.field[row - 1][col] === 0) return "up";
      else if (col < 3 && this.field[row][col + 1] === 0) return "right";
      else if (row < 3 && this.field[row + 1][col] === 0) return "down";
      else if (col > 0 && this.field[row][col - 1] === 0) return "left";
      return false;
    },

    defaultState() {
      this.puzzles.forEach((puzzle) => {
        let i = puzzle.number;
        let row = Math.floor((i - 1) / 4);
        let col = (i - 1) % 4;
        puzzle.row = row;
        puzzle.col = col;
        this.field[row][col] = i;
      });
      this.field[3][3] = 0;
    },

    onDragstart() {
      return false;
    },

    emptyCell() {
      let row, col;
      for (let i in this.field) {
        let findIndex = this.field[i].findIndex((collItem) => collItem === 0);
        if (findIndex >= 0) {
          row = parseInt(i);
          col = findIndex;
          break;
        }
      }
      return {
        row: row,
        col: col,
      };
    },

    puzzlesNearby(cell) {
      let puzzlesNearby = [];

      let variants = [
        this.puzzles.find(
          (puzzle) => puzzle.row === cell.row - 1 && puzzle.col === cell.col
        ),
        this.puzzles.find(
          (puzzle) => puzzle.row === cell.row && puzzle.col === cell.col + 1
        ),
        this.puzzles.find(
          (puzzle) => puzzle.row === cell.row + 1 && puzzle.col === cell.col
        ),
        this.puzzles.find(
          (puzzle) => puzzle.row === cell.row && puzzle.col === cell.col - 1
        ),
      ];

      for (let puzzle of variants) {
        if (puzzle) {
          puzzlesNearby.push(puzzle);
        }
      }
      return puzzlesNearby;
    },

    movePuzzle(puzzle, emptyCell) {
      let prevRow = puzzle.row;
      let prevCol = puzzle.col;
      let currentRow = emptyCell.row;
      let currentCol = emptyCell.col;
      puzzle.row = currentRow;
      puzzle.col = currentCol;
      this.field[prevRow][prevCol] = 0;
      this.field[currentRow][currentCol] = puzzle.number;
    },

    shuffle() {
      this.isShuffleButtonActive = false;
      let shuffleTransition = 0.06;
      this.puzzlesTransition = 0.15;
      let emptyCell = this.emptyCell();
      let prevRandomPuzzle;
      let i = 1;
      let shuffleInterval = setInterval(() => {
        let puzzlesNearby = this.puzzlesNearby(emptyCell);
        if (i > 1)
          puzzlesNearby = puzzlesNearby.filter(
            (puzzle) => puzzle.number !== prevRandomPuzzle.number
          );
        let randomPuzzle = puzzlesNearby[randomInteger(0, puzzlesNearby.length - 1)];
        prevRandomPuzzle = { ...randomPuzzle };
        this.movePuzzle(randomPuzzle, emptyCell);
        emptyCell.row = prevRandomPuzzle.row;
        emptyCell.col = prevRandomPuzzle.col;
        if (i >= 20) {
          this.puzzlesTransition = this.puzzlesTransitionDefault;
          clearInterval(shuffleInterval);
          this.isShuffleButtonActive = true;
        }
        i++;
      }, shuffleTransition * 1000);
    },

    onMousedown(event, puzzle) {
      let puzzlesTransitionDefault = this.puzzlesTransitionDefault;
      const movePuzzle = this.movePuzzle;
      let empty = this.isEmptyNearby(puzzle.row, puzzle.col);
      if (empty) {
        const element = event.target.closest(".puzzle");
        const elementSize = element.offsetWidth;
        const mouseStartX = event.pageX;
        const mouseStartY = event.pageY;
        let deltaX, deltaY;
        const computedLeft = parseInt(getComputedStyle(element).left);
        const computedTop = parseInt(getComputedStyle(element).top);

        document.addEventListener("pointermove", onMousemove);
        document.addEventListener("pointerup", onMouseup, { once: true });

        function onMouseup() {
          document.removeEventListener("pointermove", onMousemove);
          element.style.transition = `all ${puzzlesTransitionDefault}s linear 0s`;

          switch (empty) {
            case "up":
              if (deltaY > elementSize * 0.35) {
                movePuzzle(puzzle, { row: puzzle.row - 1, col: puzzle.col, number: 0 });
                // puzzle.row = puzzle.row - 1;
              } else element.style.top = 25 * puzzle.row + "%";
              break;
            case "right":
              if (-deltaX > elementSize * 0.35) {
                movePuzzle(puzzle, { row: puzzle.row, col: puzzle.col + 1, number: 0 });
                // puzzle.col = puzzle.col + 1;
              } else element.style.left = 25 * puzzle.col + "%";
              break;
            case "down":
              if (-deltaY > elementSize * 0.35) {
                movePuzzle(puzzle, { row: puzzle.row + 1, col: puzzle.col, number: 0 });
                // puzzle.row = puzzle.row + 1;
              } else element.style.top = 25 * puzzle.row + "%";
              break;
            case "left":
              if (deltaX > elementSize * 0.35) {
                movePuzzle(puzzle, { row: puzzle.row, col: puzzle.col - 1, number: 0 });
                // puzzle.col = puzzle.col - 1;
              } else element.style.left = 25 * puzzle.col + "%";
              break;
          }
        }

        function onMousemove(event) {
          element.style.transition = "none";
          deltaX = mouseStartX - event.pageX;
          deltaY = mouseStartY - event.pageY;

          if (empty) {
            switch (empty) {
              case "up":
                if (0 < deltaY && deltaY < elementSize) {
                  element.style.top = computedTop - deltaY + "px";
                }
                break;
              case "right":
                if (0 > deltaX && -deltaX < elementSize) {
                  element.style.left = computedLeft - deltaX + "px";
                }
                break;
              case "down":
                if (0 > deltaY && -deltaY < elementSize) {
                  element.style.top = computedTop - deltaY + "px";
                }
                break;
              case "left":
                if (0 < deltaX && deltaX < elementSize) {
                  element.style.left = computedLeft - deltaX + "px";
                }
                break;
            }
          }
        }
      }
    },
  },

  computed: {},

  watch: {
    field: {
      handler(newField) {       
        this.isTrueField = isTrueField(newField);
      },
      deep: true,
    },
  },

  mounted() {
    this.puzzles = puzzlesDefault;
    this.defaultState();   
  },
};
</script>

<style lang="scss" scoped>
.page {

  .content{
    margin-top: 1.5vh;
    display: flex;
  flex-direction: column;
  align-items: center; 
  gap: 3vh;

  @media (max-height:660px) {
    gap: 1.4vh;
    }
  }

  h1{
    text-align: center;
    font-size: 2.5em;
    color: #2c5383;

    @media (max-height:670px), (max-width:360px) {
     font-size: 2.1em;
    }
  }

  .fieldIndikator {
    width: 80px;
    height: 80px;
    border: 2px solid black;
    border-radius: 50%;
    background-image: url("/src/sourses/images/icon-thinking-color-982997.png");
    background-position: center;
    background-size: 100%;
    filter: grayscale(0.5) contrast(0.4) brightness(1.2);
    flex: 0 0 auto; 

    @media (max-height:660px) {
      width: 11vh;
    height: 11vh;
    }

    @media (max-width:320px) {
      width: 20vw;
    height: 20vw;
    }

    &.good {   
      background-image: url("/src/sourses/images/icon-smile-743211.png");
      background-position: center;
      background-size: 100%;
      filter: none;
      animation: scaleDoneSmile 0.5s ease 1;
    }
  }

  .wrapper { 
    width: 400px;
    max-width: 100%;
    display: flex;
    padding: 10px;
    border: 4px solid rgb(119, 119, 119);
    border-radius: 10px;
    background: rgba(128, 128, 128, 0.555);
    box-shadow: 4px 4px 40px rgba(0, 0, 0, 0.39);

    @media (max-height:660px) {
        width: 60vh;
    }

    &.trueField {
      border-color: #e71775;
      background: #f5bc1fb2;
      box-shadow: 4px 4px 50px 5px #e403689a;
      animation: scaleDone 0.5s ease 1;
      
    }
    .container {
      width: 100%;
      padding-top: 100%;
      border: 3px solid rgba(114, 114, 114, 0.884);
      border-radius: 6px;
      position: relative;
      background: rgb(255, 255, 255);
 

      &.trueField {
        border-color: #e71775;
        animation: swing 0.5s ease 1;
      }
    }
  }

  .btns {  
    width: 400px;
    max-width: 100%;
    display: flex;
    justify-content: space-around;
    align-items: center;
    flex-wrap: wrap;
    gap: 10px;

    button {
      font-size: 1.3em;

      width: 110px;
      height: 35px;
      padding: 4px 10px;
      box-shadow: 3px 3px 20px rgba(0, 0, 0, 0.39);
      background-color: #b5ceec;
      border: 1px solid rgb(119, 119, 119);
      color: gray;
      border: none;
      transition: 0.1s;

      &.active {
        border: 2px solid #587496;
        color: black;
        background-color: #91c3ff;

        &:hover {
          background: #cce3ff;
          font-weight: bold;
          color: rgb(0, 3, 156);
        }
      }
    }
  }

  @keyframes swing {
    15% {
      -webkit-transform: translateX(9px);
      transform: translateX(9px);
    }
    30% {
      -webkit-transform: translateX(-9px);
      transform: translateX(-9px);
    }
    40% {
      -webkit-transform: translateX(6px);
      transform: translateX(6px);
    }
    50% {
      -webkit-transform: translateX(-6px);
      transform: translateX(-6px);
    }
    65% {
      -webkit-transform: translateX(3px);
      transform: translateX(3px);
    }
    100% {
      -webkit-transform: translateX(0);
      transform: translateX(0);
    }
  }

  @keyframes scaleDone {
    0% {
      transform: scale(1);
    }

    50% {
      transform: scale(1.05);
    }

    100% {
      transform: scale(1);
    }
  }

  @keyframes scaleDoneSmile {
    0% {
      transform: scale(1);
    }

    50% {
      transform: scale(1.2);
    }

    100% {
      transform: scale(1);
    }
  }

  @keyframes selfAppearance {
    0% {
      transform: translateX(calc(50vw + 50%));
      opacity: 0;
    }

    25% {
      transform: none;
      opacity: 1;
    }

    75% {
      transform: none;
      opacity: 1;
    }

    100% {
      transform: translateX(calc(-50vw - 50%));
      opacity: 0;
    }
  }

  .popUpCool {
    position: absolute;
    inset: 0 0 0 0;
    z-index: 3;
    user-select: none;
    pointer-events: none;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;

    &__content {
      width: 299px;
      position: relative;
      top: -40px;
      max-width: 100%;
      height: 333px;
      background-image: url("/src/sourses/images/foto-self2-half.png");
      background-repeat: no-repeat;
      background-position: center;
      background-size: 100%;
      transition: 1s;
      transform: translateX(calc(50vw + 50%));
      opacity: 0;

      &.active {
        animation: selfAppearance 1.5s ease 0.5s 1; 
      }
    }
  }
}
</style>
