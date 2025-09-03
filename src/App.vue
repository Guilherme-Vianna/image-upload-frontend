<script setup lang="ts">
import TopBar from "./components/TopBar.vue";
import ImageContainer from "./components/ImageContainer.vue";
import {useFileDialog, useMounted} from "@vueuse/core";
import {v4} from "uuid";

const {files, open, reset, onCancel, onChange} = useFileDialog({
  directory: false,
});

</script>

<template>
  <TopBar/>
  <div class="flex flex-col items-center px-6 sm:px-10 py-10">
    <div class="text-3xl sm:text-4xl font-extrabold text-gray-800 text-center">
      Suba qualquer imagem de qualquer tamanho!
    </div>
    <div class="text-lg sm:text-2xl text-gray-600 mt-2 text-center">
      Compartilhe com o link curto
    </div>

    <button
        type="button"
        @click="reset(); open()"
        class="mt-12 bg-green-700 hover:bg-green-800 transition-colors duration-300
             px-8 py-4 rounded-xl text-white font-semibold shadow-lg
             active:scale-95 focus:ring-4 focus:ring-green-400"
    >
      Adicionar Imagem
    </button>

    <div
        class="w-full max-w-3xl h-72 overflow-y-auto mt-10 p-5 rounded-2xl border border-gray-200
             shadow-md bg-white/80 backdrop-blur-sm"
    >
      <div class="flex flex-col gap-4">
        <ImageContainer v-for="image in files" :key="v4()" :image="image"/>
      </div>
    </div>
  </div>
</template>


<style></style>
