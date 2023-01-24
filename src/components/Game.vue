<script setup lang="ts">
import { computed, reactive } from "vue";
import Cell from "./Cell.vue";

const state = reactive({
  historique: [],
  board: initBoard(),
  joueurActif: "blanc",
  selection: null,
});

const message = computed(() => {
  if (state.selection) {
    return "Sélectionne la destination";
  } else {
    return `Aux ${state.joueurActif}s de jouer`;
  }
});

interface Case {
  r: Number
  c: Number
  black: Boolean
}
function initBoard() {
  const board: Case[][] = [];
  let currentBlack = false;
  for (var r = 0; r < 8; r++) {
    board[r] = [];
    for (var c = 0; c < 8; c++) {
      board[r][c] = { r, c, black: currentBlack };
      currentBlack = !currentBlack;
    }
    currentBlack = !currentBlack;
  }
  return board;
}

</script>

<template>
  <div class="greetings">
    <h1 class="green">Vuejs/ts chess</h1>
    <table>
      <tr v-for="(row, i) in state.board" :key="i">
        <td v-for="(cell, j) in row" :key="`${i}-${j}`">
          <Cell v-bind="cell" />
        </td>
      </tr>
    </table>

    <h3>
      You’ve successfully created a project with
      <a href="https://vitejs.dev/" target="_blank" rel="noopener">Vite</a> +
      <a href="https://vuejs.org/" target="_blank" rel="noopener">Vue 3</a>.
    </h3>
  </div>
  <div><Cell v-bind="state.board[0][0]" />
    {{ message }}
  </div>
</template>

<style scoped>
h1 {
  font-weight: 500;
  font-size: 2.6rem;
  top: -10px;
}

h3 {
  font-size: 1.2rem;
}

.greetings h1,
.greetings h3 {
  text-align: center;
}

@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
