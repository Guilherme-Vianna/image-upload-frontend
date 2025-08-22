<script setup lang="ts">
import axios from "axios";
import TopBar from "./components/TopBar.vue";
import {
  notNullish,
  useAsyncState,
  useClipboard,
  useFileDialog,
  useLocalStorage,
  useWindowSize,
} from "@vueuse/core";
import { API_URL } from "../enviroments";
import { v4 as uuidv4 } from "uuid";
import { ref } from "vue";

const { files, open, reset, onCancel, onChange } = useFileDialog({
  directory: false,
});

const images = ref([]);

async function loadImages() {
  const sessionId = localStorage.getItem("sessionId");
  if (sessionId == null) useLocalStorage("sessionId", uuidv4());

  const resultImages = await axios.get(
    API_URL + `/image?sessionId=${sessionId}`
  );
  images.value = resultImages.data;
}

const { state, execute, isLoading } = useAsyncState(loadImages, null, {
  immediate: false,
});

onChange(async (files) => {
  if (!files || files.length === 0) {
    return;
  }
  const sessionId = localStorage.getItem("sessionId");
  if (sessionId == null) {
    console.error("SessionId not found");
  }

  let formData = new FormData();
  formData.append("file", files?.[0]);
  formData.append("sessionId", sessionId);

  const result = await axios.post(API_URL + `/image`, formData);

  if (result.status == 200) {
    execute();
  }
});

execute();

onCancel(() => {});

const { copy } = useClipboard();
function copyLink(url) {
  copy(url);
  alert("Link Copiado");
}
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
      class="mt-20 bg-green-900 p-5 px-10 rounded-sm text-white shadow-2xl cursor-pointer"
    >
      Adicionar Imagem
    </button>
    <ol
      class="flex flex-col min-w-4/12 bg-green-800 rounded-xl p-5 mt-12 text-white font-bold gap-12"
    >
      <div v-if="isLoading">Carregando Imagens...</div>
      <li v-for="x in images">
        <div class="flex justify-between items-center gap-5 sm:px-5">
          <a class="max-w-5/12">
            {{
              x.url.slice(useWindowSize().width.value < 450 ? 75 : 62, Infinity)
            }}
          </a>
          <button
            class="p-2 bg-white rounded-sm text-green-800 cursor-pointer"
            @click="copyLink(x.url)"
          >
            Copiar Link
          </button>
        </div>
      </li>
    </ol>
  </div>
</template>

<style></style>
