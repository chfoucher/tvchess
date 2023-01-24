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
const mvtsPossibles: {
  noir: MovementsPossibles[];
  blanc: MovementsPossibles[];
} = {
  noir: [],
  blanc: [],
};
let indexOrigine: number;

type Couleur = "blanc" | "noir";

interface Coordinates {
  r: number;
  c: number;
}

interface Piece {
  type: number;
  couleur: Couleur;
}

interface Case {
  r: number;
  c: number;
  black: boolean;
  piece?: Piece;
  selected?: boolean;
}

interface Movement {
  origine: { r: number; c: number; piece: Piece };
  destination: { r: number; c: number; piece?: Piece };
}

interface MovementsPossibles {
  origine: Coordinates;
  destinations: Coordinates[];
}

interface State {
  historique: Movement[];
  board: Case[][];
  joueurActif: "blanc" | "noir";
  selection: Coordinates | null;
}
const state: State = reactive({
  historique: [],
  board: initBoard(),
  joueurActif: "blanc",
  selection: null,
});
const annulationDesactivee = computed(() => {
  return state.historique.length === 0;
});
const message = computed(() => {
  if (state.selection) {
    return "Sélectionne la destination";
  } else {
    return `Aux ${state.joueurActif}s de jouer`;
  }
});

function initPartie() {
  state.historique = [];
  state.joueurActif = "blanc";
  initBoardLayout();
  calculeMouvements();
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
  mvtsPossibles.noir = [];
  mvtsPossibles.blanc = [];
  const layout = [TOUR, CHEVAL, FOU, REINE, ROI, FOU, CHEVAL, TOUR];
  for (var c = 0; c < 8; c++) {
    state.board[0][c].piece = { type: layout[c], couleur: NOIR };
    ajouteOrigine(NOIR, 0, c);
    state.board[1][c].piece = { type: PION, couleur: NOIR };
    ajouteOrigine(NOIR, 1, c);
    state.board[6][c].piece = { type: PION, couleur: BLANC };
    ajouteOrigine(BLANC, 6, c);
    state.board[7][c].piece = { type: layout[c], couleur: BLANC };
    ajouteOrigine(BLANC, 7, c);
    for (let r = 2; r < 6; r++) {
      state.board[r][c].piece = undefined;
    }
  }
}

function onClick(caseChoisie: Coordinates) {
  if (state.selection) {
    if (destinationAutorisee(caseChoisie.r, caseChoisie.c)) {
      state.historique.push({
        origine: {
          r: state.selection.r,
          c: state.selection.c,
          piece: state.board[state.selection.r][state.selection.c]
            .piece as Piece,
        },
        destination: {
          r: caseChoisie.r,
          c: caseChoisie.c,
          piece: state.board[caseChoisie.r][caseChoisie.c].piece,
        },
      });
      const adversaire = state.joueurActif === BLANC ? NOIR : BLANC;
      retireOrigine(state.joueurActif, state.selection.r, state.selection.c);
      if (state.board[caseChoisie.r][caseChoisie.c].piece)
        retireOrigine(adversaire, caseChoisie.r, caseChoisie.c);
      state.board[caseChoisie.r][caseChoisie.c].piece =
        state.board[state.selection.r][state.selection.c].piece;
      ajouteOrigine(state.joueurActif, caseChoisie.r, caseChoisie.c);
      promotion(caseChoisie);
      roque(state.selection.r, state.selection.c, caseChoisie.r, caseChoisie.c);
      state.board[state.selection.r][state.selection.c].piece = undefined;
      state.joueurActif = adversaire;
      calculeMouvements();
    }

    state.board[state.selection.r][state.selection.c].selected = false;
    state.selection = null;
  } else {
    if (origineAutorisee(caseChoisie)) {
      state.selection = caseChoisie;
      state.board[state.selection.r][state.selection.c].selected = true;
    }
  }
}

function onAnnule() {
  const mouvement = state.historique.pop();
  if (mouvement == undefined) return;
  const orig = mouvement.origine; // {row, col, piece}
  const dst = mouvement.destination;
  const couleur = orig.piece.couleur;
  const roque = orig.piece.type === ROI && Math.abs(dst.c - orig.c) == 2;
  if (roque) {
    if (dst.c - orig.c > 0) {
      retireOrigine(couleur, dst.r, 5);
      ajouteOrigine(couleur, dst.r, 7);
      state.board[dst.r][7].piece = state.board[dst.r][5].piece;
      state.board[dst.r][5].piece = undefined;
    } else {
      retireOrigine(couleur, dst.r, 3);
      ajouteOrigine(couleur, dst.r, 0);
      state.board[dst.r][0].piece = state.board[dst.r][3].piece;
      state.board[dst.r][3].piece = undefined;
    }
  }
  retireOrigine(couleur, dst.r, dst.c);
  if (dst.piece) ajouteOrigine(dst.piece.couleur, dst.r, dst.c);
  ajouteOrigine(couleur, orig.r, orig.c);
  state.board[orig.r][orig.c].piece = orig.piece;
  state.board[dst.r][dst.c].piece = dst.piece;
  state.joueurActif = state.joueurActif === BLANC ? NOIR : BLANC;
  calculeMouvements();
}

function promotion(destination: Coordinates) {
  const piece = state.board[destination.r][destination.c].piece;
  if (piece?.type === PION) {
    if (
      (piece.couleur === NOIR && destination.r === 7) ||
      (piece.couleur === BLANC && destination.r === 0)
    ) {
      const nouvPiece = { type: REINE, couleur: piece.couleur };
      state.board[destination.r][destination.c].piece = nouvPiece;
    }
  }
}

function roque(or: number, oc: number, dr: number, dc: number) {
  const couleur = state.board[or][oc].piece?.couleur;
  const row = or;
  if (oc == 4 && state.board[or][oc].piece?.type === ROI) {
    if (dc == 2 || dc == 6) {
      const origCol = dc == 2 ? 0 : 7;
      const destCol = dc == 2 ? 3 : 5;
      state.board[row][destCol].piece = state.board[row][origCol].piece;
      state.board[row][origCol].piece = undefined;
      if (couleur !== undefined) {
        retireOrigine(couleur, row, origCol);
        ajouteOrigine(couleur, row, destCol);
      }
    }
  }
}

function ajouteOrigine(couleur: Couleur, r: number, c: number) {
  mvtsPossibles[couleur].push({
    origine: { r, c },
    destinations: [],
  });
}

function retireOrigine(couleur: Couleur, r: number, c: number) {
  const mvts = mvtsPossibles[couleur];
  for (let i = 0; i < mvts.length; i++) {
    if (mvts[i].origine.r === r && mvts[i].origine.c === c) {
      mvts.splice(i, 1);
      break;
    }
  }
}

function calculeMouvementsPion(r: number, c: number): Coordinates[] {
  const resultat = [];
  let dr, dc;
  const direction = state.joueurActif === BLANC ? -1 : 1;
  // Avance de 1
  dr = r + direction;
  dc = c;
  if (dr <= 7 && dr >= 0 && !state.board[dr][dc].piece)
    resultat.push({ r: dr, c: dc });
  // Avance de 2
  const startingRow = state.joueurActif === BLANC ? 6 : 1;
  if (r === startingRow) {
    dr = r + direction * 2;
    if (dr <= 7 && dr >= 0 && !state.board[dr][dc].piece)
      resultat.push({ r: dr, c: dc });
  }
  // Mange à gauche
  dr = r + direction;
  dc = c - 1;
  if (
    dr <= 7 &&
    dr >= 0 &&
    dc >= 0 &&
    state.board[dr][dc].piece &&
    state.board[dr][dc].piece?.couleur != state.joueurActif
  )
    resultat.push({ r: dr, c: dc });
  // Mange à droite
  dc = c + 1;
  if (
    dr <= 7 &&
    dr >= 0 &&
    dc <= 7 &&
    state.board[dr][dc].piece &&
    state.board[dr][dc].piece?.couleur != state.joueurActif
  )
    resultat.push({ r: dr, c: dc });
  return resultat;
}

function calculeMouvementsCercle(mvts: number[][], r: number, c: number) {
  const resultat = [];
  let dr, dc;
  for (const mvt of mvts) {
    dr = r + mvt[0];
    dc = c + mvt[1];
    if (dr <= 7 && dr >= 0 && dc <= 7 && dc >= 0) {
      if (
        !state.board[dr][dc].piece ||
        state.board[dr][dc].piece?.couleur != state.joueurActif
      ) {
        resultat.push({ r: dr, c: dc });
      }
    }
  }
  return resultat;
}

function calculeMouvementsCavalier(r: number, c: number) {
  return calculeMouvementsCercle(
    [
      [1, 2],
      [2, 1],
      [-1, 2],
      [2, -1],
      [1, -2],
      [-2, 1],
      [-1, -2],
      [-2, -1],
    ],
    r,
    c
  );
}

function calculeRoque(r: number, c: number) {
  const resultat = [];
  const startingRow = state.joueurActif === BLANC ? 7 : 0;
  if (
    r == startingRow &&
    c == 4 &&
    !state.board[r][c + 1].piece &&
    !state.board[r][c + 2].piece
  ) {
    resultat.push({ r, c: c + 2 });
  }
  if (
    r == startingRow &&
    c == 4 &&
    !state.board[r][c - 1].piece &&
    !state.board[r][c - 2].piece &&
    !state.board[r][c - 3].piece
  ) {
    resultat.push({ r, c: c - 2 });
  }
  return resultat;
}

function calculeMouvementsRoi(r: number, c: number) {
  const mvtsNormaux = calculeMouvementsCercle(
    [
      [1, 0],
      [1, 1],
      [1, -1],
      [0, -1],
      [-1, 0],
      [-1, 1],
      [-1, -1],
      [0, 1],
    ],
    r,
    c
  );
  const mvtsRoque = calculeRoque(r, c);
  return mvtsNormaux.concat(mvtsRoque);
}

function calculeMouvementsLigne(mvts: number[][], r: number, c: number) {
  const resultat = [];
  var dr, dc;
  for (const mvt of mvts) {
    dr = r + mvt[0];
    dc = c + mvt[1];
    while (dr <= 7 && dr >= 0 && dc <= 7 && dc >= 0) {
      if (state.board[dr][dc].piece) {
        if (state.board[dr][dc].piece?.couleur != state.joueurActif) {
          // Prise
          resultat.push({ r: dr, c: dc });
        }
        break; // Fin de la trajectoire
      } else {
        // Simple mouvement
        resultat.push({ r: dr, c: dc });
      }
      dr += mvt[0];
      dc += mvt[1];
    }
  }
  return resultat;
}

function calculeMouvementsFou(r: number, c: number) {
  return calculeMouvementsLigne(
    [
      [1, 1],
      [1, -1],
      [-1, -1],
      [-1, 1],
    ],
    r,
    c
  );
}

function calculeMouvementsTour(r: number, c: number) {
  return calculeMouvementsLigne(
    [
      [0, 1],
      [0, -1],
      [-1, 0],
      [1, 0],
    ],
    r,
    c
  );
}

function calculeMouvementsReine(r: number, c: number) {
  const mvtsFou = calculeMouvementsFou(r, c);
  const mvtsTour = calculeMouvementsTour(r, c);
  return mvtsFou.concat(mvtsTour);
}

function calculeMouvementsOrigine(r: number, c: number): Coordinates[] {
  const piece = state.board[r][c].piece;
  var resultat: Coordinates[] = [];
  switch (piece?.type) {
    case PION:
      resultat = calculeMouvementsPion(r, c);
      break;
    case CHEVAL:
      resultat = calculeMouvementsCavalier(r, c);
      break;
    case FOU:
      resultat = calculeMouvementsFou(r, c);
      break;
    case TOUR:
      resultat = calculeMouvementsTour(r, c);
      break;
    case REINE:
      resultat = calculeMouvementsReine(r, c);
      break;
    case ROI:
      resultat = calculeMouvementsRoi(r, c);
      break;
  }
  return resultat;
}

function calculeMouvements() {
  const possibles = mvtsPossibles[state.joueurActif];
  for (const mouvement of possibles) {
    mouvement.destinations = calculeMouvementsOrigine(
      mouvement.origine.r,
      mouvement.origine.c
    );
  }
}

function origineAutorisee({ r, c }: Coordinates) {
  const cases = mvtsPossibles[state.joueurActif];
  for (let i = 0; i < cases.length; i++) {
    if (
      cases[i].origine.r === r &&
      cases[i].origine.c === c &&
      cases[i].destinations.length
    ) {
      indexOrigine = i;
      return true;
    }
  }
  return false;
}

function destinationAutorisee(dr: number, dc: number) {
  const destinations =
    mvtsPossibles[state.joueurActif][indexOrigine].destinations;
  for (let i = 0; i < destinations.length; i++) {
    if (destinations[i].r === dr && destinations[i].c === dc) {
      return true;
    }
  }
  return false;
}
</script>

<template>
  <div class="greetings">
    <h1 class="green">Vuejs/ts chess</h1>
    <table>
      <tr v-for="(row, i) in state.board" :key="i">
        <td v-for="(cell, j) in row" :key="`${i}-${j}`">
          <Cell v-bind="cell" @click="onClick" />
        </td>
      </tr>
    </table>
  </div>
  <div>
    <button id="btnNouveau" @click="initPartie">Nouvelle partie</button>
    <button id="btnAnnule" :disabled="annulationDesactivee" @click="onAnnule">
      Annule
    </button>
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
