<script setup lang="ts">
import { computed } from "vue";

import viseur_noir from "../assets/viseur_noir.svg";
import viseur_blanc from "../assets/viseur_blanc.svg";
import Chess_hdt45 from "../assets/Chess_hdt45.svg";
import Chess_plt45 from "../assets/Chess_plt45.svg";
import Chess_mdt45 from "../assets/Chess_mdt45.svg";
import Chess_rlt45 from "../assets/Chess_rlt45.svg";
import Chess_fdt45 from "../assets/Chess_fdt45.svg";
import Chess_klt45 from "../assets/Chess_klt45.svg";
import Chess_Bdt45 from "../assets/Chess_Bdt45.svg";
import Chess_blt45 from "../assets/Chess_blt45.svg";
import Chess_Ndt45 from "../assets/Chess_Ndt45.svg";
import Chess_nlt45 from "../assets/Chess_nlt45.svg";
import Chess_gdt45 from "../assets/Chess_gdt45.svg";
import Chess_qlt45 from "../assets/Chess_qlt45.svg";
const images = [
  viseur_noir,
  viseur_blanc,
  Chess_hdt45,
  Chess_plt45,
  Chess_mdt45,
  Chess_rlt45,
  Chess_fdt45,
  Chess_klt45,
  Chess_Bdt45,
  Chess_blt45,
  Chess_Ndt45,
  Chess_nlt45,
  Chess_gdt45,
  Chess_qlt45,
];

interface Piece {
  type: number;
  couleur: string;
}

const props = defineProps<{
  black: boolean;
  c: number;
  piece?: Piece;
  r: number;
  selected?: boolean;
}>();
const imgSrc = computed(() => {
  if (props.piece === undefined) return undefined;
  const i = 2 * props.piece.type;
  const j = props.piece.couleur === "noir" ? 0 : 1;
  const k = i + j;
  return images[k];
});
const emit = defineEmits(["click"]);

function onClick() {
  emit("click", { r: props.r, c: props.c });
}
</script>

<template>
  <div :class="{ black, selected }" @click="onClick">
    <img v-if="piece" :src="imgSrc" />
  </div>
</template>

<style scoped>
div {
  width: 60px;
  height: 60px;
  cursor: pointer;
  background-color: #ffce9e;
}

div.black {
  background-color: #d18b47;
}

div.selected {
  background-color: red;
}

img {
  width: 60px;
  height: 60px;
}
</style>
