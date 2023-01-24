<script setup lang="ts">
import { computed, reactive } from "vue";
import Cell from "./Cell.vue";

const PION = 1;
const TOUR = 2;
const ROI = 3;
const FOU = 4;
const CHEVAL = 5;
const REINE = 6;
const NOIR = "noir";
const BLANC = "blanc";
const mvtsPossibles = [];
let indexOrigine;

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

interface Piece {
  type: number;
  couleur: string;
}

interface Case {
  r: number;
  c: number;
  black: boolean;
  piece?: Piece;
}

function initPartie() {
  state.historique = [];
  state.joueurActif = "blanc";
  initBoardLayout();
  // calculeMouvements();
}

initPartie();

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

function initBoardLayout() {
  //mvtsPossibles[NOIR] = [];
  //mvtsPossibles[BLANC] = [];
  const layout = [TOUR, CHEVAL, FOU, REINE, ROI, FOU, CHEVAL, TOUR];
  for (var c = 0; c < 8; c++) {
    state.board[0][c].piece = { type: layout[c], couleur: NOIR };
    // ajouteOrigine(NOIR, 0, c);
    state.board[1][c].piece = { type: PION, couleur: NOIR };
    // ajouteOrigine(NOIR, 1, c);
    state.board[6][c].piece = { type: PION, couleur: BLANC };
    // ajouteOrigine(BLANC, 6, c);
    state.board[7][c].piece = { type: layout[c], couleur: BLANC };
    // ajouteOrigine(BLANC, 7, c);
    for (let r = 2; r < 6; r++) {
      state.board[r][c].piece = undefined;
    }
  }
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
  </div>
  <div>
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

table {
  background-color: darkgray;
}

td {
  padding: 0;
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
