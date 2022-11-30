<template>
  <!-- <NavBar></NavBar> -->
  <div class="app">
    <main>
      <GameTag></GameTag>
    </main>
  </div>
</template>

<script>
import GameTag from "@/components/GameTag";

export default {
  components: { GameTag },
};
</script>

<style lang="scss">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
.app {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: stretch;

  &__btns {
    margin: 10px 0;
    display: flex;
    justify-content: space-between;
    background: #dcf3f3;
  }
}
main {
  position: relative;
  padding: 1.5%;
  flex: 1 1 auto;
  overflow: hidden;
}

.page {
  position: absolute;
  inset: 0;
  padding: 1.5%;
  background: rgb(255, 253, 239);
  overflow: auto;
}
</style>
