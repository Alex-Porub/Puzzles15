<template>
  <div class="puzzle" :data-number="puzzle.number" :style="{ top: styleTop, left: styleLeft }">
    <span class="puzzle__filling">
      {{puzzle.number}}
    </span>
  </div>
</template>

<script>

export default {
  props:{
    puzzle:{
      type: Object,
      required: true,
    }
  },
  computed:{
    styleTop(){
      return 25*this.puzzle.row +"%";
    },
    styleLeft(){
      return 25*this.puzzle.col +"%";
    },
  }
};
</script>

<style lang="scss" scoped>
.puzzle {
  outline: 2px solid rgba(105, 59, 7, 0.384);
  outline-offset: -3px; 
  width: 25%;
  height: 25%;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 10%;
  background: rgb(236, 226, 181);
  position: absolute;
  user-select: none;
  box-shadow: inset 3px 3px 16px 2px rgba(105, 59, 7, 0.61);
  transition: all 0.3s linear 0s;
  cursor: pointer;
  touch-action: none;


  &__filling { 
    width: 80%;
    height: 80%;
    border-radius: 50%;
    border: 2px solid rgba(105, 59, 7, 0.404);
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 1.8em;
    font-weight: bold;
    color: rgb(124, 75, 19);
    box-shadow: inset 4px 4px 9px rgba(255, 255, 255, 0.459);
    @media(max-width:330px), (width > height) {
      font-size: 1.4em;
    }
  

  }
}
</style>
