<script setup lang="ts">
import axios from "axios";
import TopBar from "./components/TopBar.vue";
import { useFileDialog } from "@vueuse/core";
import { API_URL } from "../enviroments";
import { v4 as uuidv4 } from "uuid";
import { ref } from "vue";

const { files, open, reset, onCancel, onChange } = useFileDialog({
  directory: false,
});

const images = ref([]);

onChange(async (files) => {
  if (!files || files.length === 0) {
    return;
  }

  let id = uuidv4();
  let formData = new FormData();
  formData.append("file", files?.[0]);
  formData.append("sessionId", id);

  const result = await axios.post(API_URL + `/image`, formData);

  if (result.status == 200) {
    debugger;
    const resultImages = await axios.get(API_URL + `/image?sessionId=${id}`);
    images.value = resultImages.data;
  }
});

onCancel(() => {});
</script>

<template>
  <TopBar />
  <div class="flex flex-col px-10 py-8 items-center">
    <div class="text-4xl font-bold">
      Suba qualquer imagem de qualquer tamanho!
    </div>
    <div class="text-2xl">Compartilhe com o link curto</div>
    <button
      type="button"
      @click="open()"
      class="mt-20 bg-green-900 p-5 px-10 rounded-sm text-white shadow-2xl"
    >
      Subir Imagem
    </button>
    <ol>
      <li>Teste</li>
      <li v-for="x in images">{{ x.url }}</li>
    </ol>
  </div>
</template>

<style></style>
