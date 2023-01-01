<template>
    <div :class="{ popUpInfo: true, visible: infoVisible }" @click="infoToggle">
        <div :class="{ popUpInfo__content: true, visible: infoVisible }" @click.stop>
            <button class="popUpInfo__content_buttonClose" @click="infoToggle" title="close">
                <span></span>
                <span></span>
            </button>
            <article>
                <p>
                    {{ this.textcontent.p1[this.lang] }}
                </p>
                <img class="img-retro" src="@/sourses/images/puzzles15-retro.jpg"/>

                <p>
                    {{ this.textcontent.p2[this.lang] }}
                </p>
                <img class="img-retro" src="@/sourses/images/pointerMoving.jpg"/>
                
            </article>
        </div>
    </div>
</template>


<script>

const textcontent = {
    p1:{
        ukr: "Це компьютерна імплементація добре відомої гри П'ятнашки. Раніше вона виглядала ось так:",
        en: "This is a computer implementation of the well-known game Tag. It used to look like this:",
    },

    p2:{
        ukr: "Задача гри - зібрати всі пазли від 1 до 15 по порядку. Якщо натиснути кнопку ЗІБРАТИ - то гра сама за вас збере пазли у правильну послідовність. На початку гри, щоб перемішати пазли, ви можете натиснути кнупку ПЕРЕМІШАТИ. Якщо бажаєте перемішати більше - натисніть ще раз. І так далі. Управління можливе як за допомогою миши так і за допомогою жестів на сенсорній панелі. Для того щоб пересунути пазл із цифрой, утримуйте на ньому вказівник і переміщуйте у бік порожньої комірки. Ви можете переміщувати відразу декілька пазлів.",
        en: "The aim of the game is to collect all puzzles from 1 to 15 in correct order. If you press the BUILD button, the game will collect the puzzles in the correct sequence for you. At the beginning of the game, you can press the SHUFFLE button to shuffle the puzzles. If you want to mix more - press again. And so on. Control is possible with the mouse or with gestures on the touch panel. To move a puzzle with a number, hold the pointer on it and move it toward an empty cell. You can move several puzzles at once.",
    },
}

export default {
    props: {
        infoVisible: {
            type: Boolean,
            required: true,
        },
        infoToggle: {
            type: Function,
            required: true,
        },
        lang: {
            type: String,
            required: true,
        },
    },
    data(){
        return{
            textcontent: textcontent,
        }
    },

    methods: {},
};
</script>

<style lang="scss" scoped>
.popUpInfo {
    position: fixed;
    inset: 0 0 0 0;
    background: rgba(65, 65, 65, 0.295);
    padding: 3%;
    opacity: 0;
    transition: 0.3s;
    backdrop-filter: blur(3px);
    -webkit-backdrop-filter: blur(3px);
    pointer-events: none;

    @media (max-width: 480px){
          padding:0;
    }
      

    &__content {
        max-width: 600px;
        background: white;
        padding: 6px;
        margin-top: 3vh;
        box-shadow: 0 12px 30px rgba(0, 0, 0, 0.329), inset 0 0 10px 3px rgba(88, 42, 11, 0.5);
        margin: 0 auto;
        transition: 0.3s;
        transform: scale(0.6, 0);
        max-height: 100%;
        overflow: auto;

        @media (max-width: 480px){
            width: 100%;
            height: 100%;
            
        }
    

        &_buttonClose {
            --buttonSize: 35px;
            --borderSize: 2px;
            width: calc(var(--buttonSize)*2.5);
            height: var(--buttonSize);
            background: rgb(231, 199, 180);
            display: block;
            position: sticky;
            margin: 0 auto;
            top: -1px;
            right: -3px;
            border: var(--borderSize) solid rgb(163, 135, 119);
            border-radius: 4px;
            box-shadow: 1px 5px 10px 2px rgba(0, 0, 0, 0.3);
            cursor: pointer;

            span {
                --spanHeight: 3px;
                --spanWidth: 50%;
                width: var(--spanWidth);
                height: var(--spanHeight);
                background: rgb(65, 51, 44);
                position: absolute;
                left: calc((100% - var(--spanWidth)) / 2);
                top: calc(50% - var(--spanHeight) / 2);
                border-radius: 3px;
                box-shadow: 1px 2px 4px rgba(0, 0, 0, 0.35);
                
                &:nth-child(1) {
                    transform: rotate(36deg);
                }
                &:nth-child(2) {
                    transform: rotate(-36deg);
                }
            }

            &:hover{
                background: rgb(255, 234, 222);
                span {
                    box-shadow: 1px 3px 4px rgba(0, 0, 0, 0.45);
                }

            }
        }

        &.visible {
            transform: scale(1);
        }

        article {
            padding: 3%;
            font-size: 18px;
            // margin-top: -10px;
       
            p{
                text-indent: 1.5em
            }

            img{
                display: block;
                margin: 10px auto;
                width: 250px;
                max-width: 100%;
            }
        }
    }

    &.visible {
        opacity: 1;
        pointer-events: initial;
    }
}
</style>
