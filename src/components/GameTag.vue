<template>
    <div class="page" :class="{ page: true, [theme]: true }">
        <div class="content">
            <h1>{{ this.textcontent.header[this.lang] }}</h1>
            <div :class="{ fieldIndikator: true, good: isTrueField }"></div>
            <div :class="{ wrapper: true, trueField: isTrueField }">
                <div :class="{ container: true, trueField: isTrueField }" ref="container">
                    <MyPuzzle
                        v-for="puzzle in puzzles"
                        :puzzle="puzzle"
                        @pointerdown="onMousedown($event, puzzle)"
                        @dragstart="() => false"
                        :style="{ transitionDuration: puzzlesTransition + 's' }"
                        :theme="this.theme"
                        :key="puzzle.number"
                        @emit-ref="puzzle.element = $event"
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
            <div class="settings">
                <button :class="['settings_btn', 'settings_btn_showInfo', theme]" @click="infoToggle" title="show info">?</button>
                <button :class="{ settings_btn: true, settings_btn_theme: true, [theme]: true }" @click="themeToggle" title="change theme">
                    <img :src="iconThemeSrc" />
                </button>
                <button :class="['settings_btn', 'settings_btn_language', theme]" @click="langToggle" title="change language">
                    {{ this.lang === "ukr" ? "en" : "ukr" }}
                </button>
            </div>
            <MyPopUpInfo :infoVisible="infoVisible" :infoToggle="this.infoToggle" :lang="this.lang" :theme="this.theme" />
        </div>
    </div>
</template>

<script>
import MyPuzzle from "@/components/gameTag/MyPuzzle.vue";
import MyPopUpInfo from "@/components/gameTag/MyPopUpInfo.vue";
import iconSrc_moon from "@/sourses/images/icons8-crescent-moon-50.png";
import iconSrc_sun from "@/sourses/images/icons8-sun-60.png";

// helper
function randomInteger(min, max) {
    const rand = min + Math.random() * (max + 1 - min);
    return Math.floor(rand);
}

const puzzlesTransitionDefault = 0.1;

// creates the default puzzle array
const puzzlesDefault = (() => {
    const puzzlesDefault = [];
    for (let i = 1; i <= 15; i++) {
        const puzzle = {};
        puzzle.number = i;
        puzzlesDefault.push(puzzle);
    }
    return puzzlesDefault;
})();

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
            theme: "brightTheme",
        };
    },

    methods: {
        infoToggle() {
            this.infoVisible = !this.infoVisible;
        },
        langToggle() {
            this.lang = this.lang == "ukr" ? "en" : "ukr";
        },
        themeToggle() {
            this.theme = this.theme == "brightTheme" ? "darkTheme" : "brightTheme";
        },

        // adds an index to array of puzzle indexes to move
        pushIndexToMove(index) {
            this.indexesToMove.push(index);
            this.puzzles[index].computedLeft = parseInt(getComputedStyle(this.puzzles[index].element).left);
            this.puzzles[index].computedTop = parseInt(getComputedStyle(this.puzzles[index].element).top);
        },

        // Detects if is there any empty cell in the column or the row to move puzzles to.
        // Tt also finds puzzles need to be moved
        emptyInAccess(row, col) {
            const empty_col = this.emptyCell.col;
            const empty_row = this.emptyCell.row;

            if (row === empty_row) {
                const increment = empty_col - col > 0 ? 1 : -1;
                for (let i = col; Math.abs(i - empty_col) > 0; i += increment) {
                    const index = this.puzzles.findIndex((puz) => puz.row === row && puz.col === i);
                    this.pushIndexToMove(index);
                }
                return col > empty_col ? "left" : "right";
            } else if (col === empty_col) {
                const increment = empty_row - row > 0 ? 1 : -1;
                for (let i = row; Math.abs(i - empty_row) > 0; i += increment) {
                    const index = this.puzzles.findIndex((puz) => puz.col === col && puz.row === i);
                    this.pushIndexToMove(index);
                }
                return row > empty_row ? "up" : "down";
            }
            this.indexesToMove.length = 0;
            return false;
        },

        // sets puzzles in default positions
        defaultState() {
            this.puzzles.forEach((puzzle) => {
                const i = puzzle.number;
                const row = Math.floor((i - 1) / 4);
                const col = (i - 1) % 4;
                puzzle.row = row;
                puzzle.col = col;
                this.field[row][col] = i;
            });
            this.field[3][3] = 0;
        },

        // defines puzzles beside an empty cell
        puzzlesBeside(cell) {
            const puzzlesBeside = [];
            const applicants = [
                this.puzzles.find((puzzle) => puzzle.row === cell.row - 1 && puzzle.col === cell.col),
                this.puzzles.find((puzzle) => puzzle.row === cell.row && puzzle.col === cell.col + 1),
                this.puzzles.find((puzzle) => puzzle.row === cell.row + 1 && puzzle.col === cell.col),
                this.puzzles.find((puzzle) => puzzle.row === cell.row && puzzle.col === cell.col - 1),
            ];

            for (let applicant of applicants) {
                if (applicant) {
                    puzzlesBeside.push(applicant);
                }
            }
            return puzzlesBeside;
        },

        // moves the puzzle to the cell
        movePuzzle(puzzle, toCell) {
            const prevRow = puzzle.row;
            const prevCol = puzzle.col;
            const currentRow = toCell.row;
            const currentCol = toCell.col;
            puzzle.row = currentRow;
            puzzle.col = currentCol;
            this.field[prevRow][prevCol] = 0;
            this.field[currentRow][currentCol] = puzzle.number;
        },

        // moves the puzzles in the direction
        movePuzzles(puzzles, direction) {
            const prevRow = puzzles[0].row;
            const prevCol = puzzles[0].col;
            this.field[prevRow][prevCol] = 0;

            for (let puzzle of puzzles) {
                let currentRow = puzzle.row;
                let currentCol = puzzle.col;
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
                puzzle.row = currentRow;
                puzzle.col = currentCol;
                this.field[currentRow][currentCol] = puzzle.number;
            }
        },

        // moves one of the closest puzzles to an empty cell a predetermined (variable 'deep') number of times, shuffling the puzzles
        shuffle(deep = 20) {
            this.onShuffle = true;
            const shuffleTransition = 0.09; /*sec*/
            this.puzzlesTransition = 0.11; /*sec*/
            const emptyCell = this.emptyCell;
            let prevRandomPuzzle;
            let i = 0;
            const shuffleInterval = setInterval(() => {
                i++;
                let puzzlesBeside = this.puzzlesBeside(emptyCell);
                if (i > 1) puzzlesBeside = puzzlesBeside.filter((puzzle) => puzzle.number !== prevRandomPuzzle.number);
                let randomPuzzle = puzzlesBeside[randomInteger(0, puzzlesBeside.length - 1)];
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
            const empty = this.emptyInAccess(puzzle.row, puzzle.col);
            const puzzlesToMove = indexesToMove.map((index) => this.puzzles[index]);

            if (empty) {
                const element = puzzle.element;
                const elementSize = element.offsetWidth;
                const mouseStartX = event.pageX;
                const mouseStartY = event.pageY;
                let deltaX, deltaY;
                document.addEventListener("pointermove", onMousemove);
                document.addEventListener("pointerup", onMouseup, { once: true });

                //moves the puzzle together with the pointer
                function onMousemove(event) {
                    puzzlesToMove.forEach((puzzleToMove) => {
                        puzzleToMove.element.style.transition = "none";
                    });
                    deltaX = mouseStartX - event.pageX;
                    deltaY = mouseStartY - event.pageY;

                    const motion =
                        (0 < deltaY && deltaY < elementSize && empty == "up") || (0 > deltaY && -deltaY < elementSize && empty == "down")
                            ? "vertical"
                            : (0 > deltaX && -deltaX < elementSize && empty == "right") ||
                              (0 < deltaX && deltaX < elementSize && empty == "left")
                            ? "horizontal"
                            : false;

                    if (motion) {
                        puzzlesToMove.forEach((puzzleToMove) => {
                            if (motion == "vertical") {
                                puzzleToMove.element.style.top = puzzleToMove.computedTop - deltaY + "px";
                            } else {
                                puzzleToMove.element.style.left = puzzleToMove.computedLeft - deltaX + "px";
                            }
                        });
                    }
                }

                // When releasing the pointer, if pointer moved up the puzzles behind the "boundaryLine" ,
                // this means that the puzzles need to change the location. Otherwise -  they must return.
                function onMouseup() {
                    document.removeEventListener("pointermove", onMousemove);
                    puzzlesToMove.forEach((puzzleToMove) => {
                        puzzleToMove.element.style.transition = `all ${puzzlesTransitionDefault}s linear 0s`;
                    });
                    const boundaryLine = elementSize * 0.35;

                    const takeBack =
                        (empty === "up" && !(deltaY > boundaryLine)) || (empty === "down" && !(-deltaY > boundaryLine))
                            ? "vertical"
                            : (empty === "right" && !(-deltaX > boundaryLine)) || (empty === "left" && !(deltaX > boundaryLine))
                            ? "horizontal"
                            : false;

                    if (takeBack) {
                        puzzlesToMove.forEach((puzzleToMove) => {
                            if (takeBack == "vertical") {
                                puzzleToMove.element.style.top = 25 * puzzleToMove.row + "%";
                            } else {
                                puzzleToMove.element.style.left = 25 * puzzleToMove.col + "%";
                            }
                        });
                    } else movePuzzles(puzzlesToMove, empty);

                    indexesToMove.length = 0;
                }
            }
        },
    },

    computed: {
        iconThemeSrc() {
            return this.theme === "darkTheme" ? iconSrc_sun : iconSrc_moon;
        },
    },

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
    },
};
</script>

<style lang="scss" scoped>
.page {
    padding: 40px 1% 0 1%;
    .content {
        margin-top: 1.5vh;
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 3vh;

        @media (max-height: 675px) {
            gap: 1.4vh;
            margin-top: 0.3vh;
        }
        .popUpInfo {
            z-index: 3;
        }
    }

    h1 {
        text-align: center;
        font-size: 2.5em;
        color: #2c5383;
        // margin: 0;

        @media (max-height: 670px), (max-width: 360px) {
            font-size: 1.8em;
            // margin-top: 30px;
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
                --color: #6899dfc9;
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
            border-radius: 8px;
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

    .settings {
        display: flex;
        justify-content: space-between;
        position: absolute;
        inset: 4px 1.3vw auto 1.5vw;

        .settings_btn {
            z-index: 5;
            min-width: 60px;
            min-height: 30px;
            padding: 2px;
            background: #416592;
            color: white;
            border: none;
            border-radius: 3px;
            font-size: 29px;
            font-weight: 400;
            box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.473);
            line-height: 0;
            cursor: pointer;

            &.brightTheme {
                background: #a6c2e2;
                color: #3a5b83;
                font-weight: bold;
            }

            &_language {
                font-size: 24px;
            }

            &_theme {
                img {
                    height: 30px;
                }
            }

            &:hover {
                background: white;
                color: #416592;
                outline: 3px solid #416592;
                outline-offset: -2px;
            }
        }
    }

    &.darkTheme {
        background: #3f3f3f;

        h1 {
            color: #d9eaff;
        }

        .wrapper {
            --borderColor: #c2bfa6;
            --borderColorTrue: #eedec8;

            border-color: var(--borderColor);
            background: rgba(128, 128, 128, 0.555);

            &.trueField {
                border-color: var(--borderColorTrue);
                background: #233c5a;
                box-shadow: 4px 4px 50px 5px #eee7c867;

                &:before {
                    --color: #ddd018b2;
                    box-shadow: 0 0 40px 40px var(--color);
                }
            }

            .container {
                border-color: var(--borderColor);
                border-radius: 6px;
                position: relative;
                background: #506681;

                &.trueField {
                    border-color: var(--borderColorTrue);
                }
            }
        }
    }
}
</style>
