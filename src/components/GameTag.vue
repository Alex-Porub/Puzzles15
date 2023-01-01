<template>
    <div class="page">
        <div class="content">
            <h1>{{ this.textcontent.header[this.lang] }}</h1>
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
                <button :class="{ defaultBut: true, active: !onShuffle }" @click="!onShuffle ? defaultState() : null" title="build">
                    {{ this.textcontent.build[this.lang] }}
                </button>
                <button
                    :class="{ defaultBut: true, active: !onShuffle }"
                    @click="!onShuffle ? shuffle() : null"
                    title="shuffle as many times as necessary"
                >
                    {{ this.textcontent.shuffle[this.lang] }}
                </button>
                <div :class="{ popUpCool: true, active: true }">
                    <div :class="{ popUpCool__content: true, active: isTrueField }"></div>
                </div>
            </div>
            <button class="btn btn_showInfo" @click="infoToggle" title="help">?</button>
            <button class="btn btn_language" @click="langToggle" title="change language">{{ this.lang }}</button>
            <MyPopUpInfo :infoVisible="infoVisible" :infoToggle="this.infoToggle" :lang="this.lang" />
        </div>
    </div>
</template>

<script>
import MyPuzzle from "@/components/gameTag/MyPuzzle.vue";
import MyPopUpInfo from "@/components/gameTag/MyPopUpInfo.vue";

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

const textcontent = {
    header: {
        ukr: "П'ятнашки",
        en: "The Tag",
    },
    build: {
        ukr: "зібрати",
        en: "build",
    },
    shuffle: {
        ukr: "перемішати",
        en: "shuffle",
    },
};

export default {
    components: { MyPuzzle, MyPopUpInfo },

    data() {
        return {
            field: [[], [], [], []],
            puzzles: puzzlesDefault,
            puzzlesTransitionDefault: puzzlesTransitionDefault,
            puzzlesTransition: puzzlesTransitionDefault,
            isTrueField: false,
            emptyCell: {},
            indexesToMove: [],
            onShuffle: false,
            infoVisible: false,
            lang: "ukr",
            textcontent: textcontent,
        };
    },

    methods: {
        infoToggle() {
            this.infoVisible = !this.infoVisible;
        },
        langToggle() {
            this.lang = this.lang == "ukr" ? "en" : "ukr";
        },

        isEmptyNearby(row, col) {
            if (row > 0 && this.field[row - 1][col] === 0) return "up";
            else if (col < 3 && this.field[row][col + 1] === 0) return "right";
            else if (row < 3 && this.field[row + 1][col] === 0) return "down";
            else if (col > 0 && this.field[row][col - 1] === 0) return "left";
            return false;
        },

        pushIndexToMove(index) {
            this.indexesToMove.push(index);
            this.puzzles[index].computedLeft = parseInt(getComputedStyle(this.puzzles[index].element).left);
            this.puzzles[index].computedTop = parseInt(getComputedStyle(this.puzzles[index].element).top);
        },

        emptyInAccess(row, col) {
            let empty_col = this.emptyCell.col;
            let empty_row = this.emptyCell.row;

            if (row === empty_row) {
                for (let i = col; Math.abs(i - empty_col) > 0; i = i + (empty_col - i) / Math.abs(empty_col - i)) {
                    let index = this.puzzles.findIndex((puz) => puz.row === row && puz.col === i);
                    this.pushIndexToMove(index);
                }
                return col > empty_col ? "left" : "right";
            } else if (col === empty_col) {
                for (let i = row; Math.abs(i - empty_row) > 0; i = i + (empty_row - i) / Math.abs(empty_row - i)) {
                    let index = this.puzzles.findIndex((puz) => puz.col === col && puz.row === i);
                    this.pushIndexToMove(index);
                }
                return row > empty_row ? "up" : "down";
            }
            this.indexesToMove.length = 0;
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

        puzzlesNearby(cell) {
            let puzzlesNearby = [];
            let variants = [
                this.puzzles.find((puzzle) => puzzle.row === cell.row - 1 && puzzle.col === cell.col),
                this.puzzles.find((puzzle) => puzzle.row === cell.row && puzzle.col === cell.col + 1),
                this.puzzles.find((puzzle) => puzzle.row === cell.row + 1 && puzzle.col === cell.col),
                this.puzzles.find((puzzle) => puzzle.row === cell.row && puzzle.col === cell.col - 1),
            ];

            for (let puzzle of variants) {
                if (puzzle) {
                    puzzlesNearby.push(puzzle);
                }
            }
            return puzzlesNearby;
        },

        movePuzzle(puzzle, toCell) {
            let prevRow = puzzle.row;
            let prevCol = puzzle.col;
            let currentRow = toCell.row;
            let currentCol = toCell.col;
            puzzle.row = currentRow;
            puzzle.col = currentCol;
            this.field[prevRow][prevCol] = 0;
            this.field[currentRow][currentCol] = puzzle.number;
        },

        movePuzzles(puzzles, direction) {
            let prevRow = puzzles[0].row;
            let prevCol = puzzles[0].col;
            this.field[prevRow][prevCol] = 0;
            for (let i = 0; i < puzzles.length; i++) {
                let currentRow = puzzles[i].row;
                let currentCol = puzzles[i].col;
                switch (direction) {
                    case "up":
                        currentRow--;
                        break;
                    case "right":
                        currentCol++;
                        break;
                    case "down":
                        currentRow++;
                        break;
                    case "left":
                        currentCol--;
                        break;
                }
                puzzles[i].row = currentRow;
                puzzles[i].col = currentCol;
                this.field[currentRow][currentCol] = puzzles[i].number;
            }
        },

        shuffle(deep = 20) {
            this.onShuffle = true;
            let shuffleTransition = 0.09;
            this.puzzlesTransition = 0.11;
            let emptyCell = this.emptyCell;
            let prevRandomPuzzle;
            let i = 0;
            let shuffleInterval = setInterval(() => {
                i++;
                let puzzlesNearby = this.puzzlesNearby(emptyCell);
                if (i > 1) puzzlesNearby = puzzlesNearby.filter((puzzle) => puzzle.number !== prevRandomPuzzle.number);
                let randomPuzzle = puzzlesNearby[randomInteger(0, puzzlesNearby.length - 1)];
                prevRandomPuzzle = { ...randomPuzzle };
                this.movePuzzle(randomPuzzle, emptyCell);
                emptyCell.row = prevRandomPuzzle.row;
                emptyCell.col = prevRandomPuzzle.col;
                if (i >= deep) {
                    clearInterval(shuffleInterval);
                    this.puzzlesTransition = this.puzzlesTransitionDefault;
                    this.onShuffle = false;
                }
            }, shuffleTransition * 1000);
        },

        onMousedown(event, puzzle) {
            const puzzlesTransitionDefault = this.puzzlesTransitionDefault;
            const movePuzzles = this.movePuzzles;
            const indexesToMove = this.indexesToMove;
            const puzzles = this.puzzles;
            const empty = this.emptyInAccess(puzzle.row, puzzle.col);

            if (empty) {
                const element = puzzle.element;
                const elementSize = element.offsetWidth;
                const mouseStartX = event.pageX;
                const mouseStartY = event.pageY;
                let deltaX, deltaY;

                document.addEventListener("pointermove", onMousemove);
                document.addEventListener("pointerup", onMouseup, { once: true });

                function onMousemove(event) {
                    indexesToMove.forEach((index) => {
                        puzzles[index].element.style.transition = "none";
                    });
                    deltaX = mouseStartX - event.pageX;
                    deltaY = mouseStartY - event.pageY;

                    switch (empty) {
                        case "up":
                            if (0 < deltaY && deltaY < elementSize) {
                                indexesToMove.forEach((index) => {
                                    puzzles[index].element.style.top = puzzles[index].computedTop - deltaY + "px";
                                });
                            }
                            break;
                        case "right":
                            if (0 > deltaX && -deltaX < elementSize) {
                                indexesToMove.forEach((index) => {
                                    puzzles[index].element.style.left = puzzles[index].computedLeft - deltaX + "px";
                                });
                            }
                            break;
                        case "down":
                            if (0 > deltaY && -deltaY < elementSize) {
                                indexesToMove.forEach((index) => {
                                    puzzles[index].element.style.top = puzzles[index].computedTop - deltaY + "px";
                                });
                            }
                            break;
                        case "left":
                            if (0 < deltaX && deltaX < elementSize) {
                                indexesToMove.forEach((index) => {
                                    puzzles[index].element.style.left = puzzles[index].computedLeft - deltaX + "px";
                                });
                            }
                            break;
                    }
                }

                function onMouseup() {
                    document.removeEventListener("pointermove", onMousemove);

                    indexesToMove.forEach((index) => {
                        puzzles[index].element.style.transition = `all ${puzzlesTransitionDefault}s linear 0s`;
                    });
                    let puzzlesToMove = indexesToMove.map((index) => puzzles[index]);

                    if (empty === "up" && !(deltaY > elementSize * 0.35)) {
                        indexesToMove.forEach((index) => {
                            puzzles[index].element.style.top = puzzles[index].computedTop + "px";
                        });
                    } else if (empty === "right" && !(-deltaX > elementSize * 0.35)) {
                        indexesToMove.forEach((index) => {
                            puzzles[index].element.style.left = puzzles[index].computedLeft + "px";
                        });
                    } else if (empty === "down" && !(-deltaY > elementSize * 0.35)) {
                        indexesToMove.forEach((index) => {
                            puzzles[index].element.style.top = puzzles[index].computedTop + "px";
                        });
                    } else if (empty === "left" && !(deltaX > elementSize * 0.35)) {
                        indexesToMove.forEach((index) => {
                            puzzles[index].element.style.left = puzzles[index].computedLeft + "px";
                        });
                    } else movePuzzles(puzzlesToMove, empty);

                    indexesToMove.length = 0;
                }
            }
        },
    },

    computed: {},

    watch: {
        field: {
            handler(newField) {
                let isTrueField = true;
                let emptyCell = {};
                for (let row = 0; row <= 3; row++) {
                    for (let col = 0; col <= 3; col++) {
                        let trueNumber = col + 1 + row * 4 !== 16 ? col + 1 + row * 4 : 0;
                        let number = newField[row][col];
                        if (number === 0) {
                            emptyCell.row = row;
                            emptyCell.col = col;
                        }
                        if (trueNumber !== number) isTrueField = false;
                    }
                }
                this.isTrueField = isTrueField;
                this.emptyCell = emptyCell;
            },
            deep: true,
        },
    },

    mounted() {
        this.defaultState();
        this.puzzles.forEach((puzzle) => {
            puzzle.element = document.querySelector(`.puzzle[data-number="${puzzle.number}"]`);
        });
    },
};
</script>

<style lang="scss" scoped>
.page {
    .content {
        margin-top: 1.6vh;
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 3vh;

        @media (max-height: 675px) {
            gap: 1.4vh;
            margin-top: 0.6vh;
        }
    }

    h1 {
        text-align: center;
        font-size: 2.5em;
        color: #2c5383;

        @media (max-height: 670px), (max-width: 360px) {
            font-size: 1.8em;
            margin-top: 30px;
            letter-spacing: 0em;
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

        @media (max-height: 660px) {
            width: 11vh;
            height: 11vh;
        }

        @media (max-width: 320px) {
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
        position: relative;
        overflow: hidden;

        @media (max-height: 680px) {
            width: 57vh;
        }

        &.trueField {
            border-color: #e71775;
            background: #f5bc1fb2;
            box-shadow: 4px 4px 50px 5px #e403689a;
            animation: scaleDone 0.5s ease 1;

            &:before {
                --height: 200%;
                --width: 20%;
                --color:#6899dfc9;
                content: "";
                position: absolute;
                width: var(--width);
                height: var(--height);
                background: var(--color);
                left: calc(50% - (var(--width) / 2));
                top: calc(50% - (var(--height) / 2));
                box-shadow: 0 0 60px 60px var(--color);
                animation: 1.5s animatedBorder linear infinite;
            }
        }

        .container {
            --borderWidth: 3px;
            width: 100%;
            padding-top: calc(100% - var(--borderWidth) * 2);
            border: var(--borderWidth) solid rgba(114, 114, 114, 0.884);
            border-radius: 6px;
            position: relative;
            background: rgb(255, 255, 255);

            &.trueField {
                border-color: #e71775;
                animation: swing 0.5s ease 1;
            }
        }

        @keyframes animatedBorder {
            0% {
                transform: rotate(0deg);
            }
            100% {
                transform: rotate(360deg);
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
            font-size: 1.23em;
            text-align: center;
            min-width: 130px;
            padding: 6px 2px;
            line-height: 1em;
            box-shadow: 3px 3px 20px rgba(0, 0, 0, 0.39);
            background-color: #b5ceec;
            border: 2px solid #98a9be;
            border-radius: 20px;
            color: gray;
            transition: 0.1s;
            font-weight: bold;
            cursor: pointer;

            &.active {
                border: 2px solid #587496;
                color: #354e6d;
                background-color: #9cc8ff;

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

    .btn {
        position: absolute;
        z-index: 5;
        width: 60px;
        height: 30px;
        background: #416592;
        color: white;
        border: none;
        border-radius: 3px;
        font-size: 25px;
        font-weight: bold;
        box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.473);
        cursor: pointer;

        &_showInfo {
            top: 4px;
            left: 1.5vw;
        }
        &_language {
            top: 4px;
            left: initial;
            right: 1.5vw;
            font-size: 22px;
            font-weight: normal;
            letter-spacing: 0.05em;
        }

        &:hover {
            background: white;
            color: #416592;
            outline: 3px solid #416592;
            outline-offset: -2px;
        }
    }
}
</style>
