<script setup lang="ts">
import TopBar from "./components/TopBar.vue";
import { useFileDialog } from "@vueuse/core";

const { files, open, reset, onCancel, onChange } = useFileDialog({
  directory: false,
});

onChange((files) => {
  if (!files || files.length === 0) {
    return;
  }

  let formData = new FormData();
  formData.append("file", files?.[0]);

  fetch("http://localhost:4040/image", {
    method: "POST",
    body: formData,
  })
    .then((response) => {
      if (response.ok && response.status === 200) {
        reset();
        alert("Upload realizado com sucesso");
      } else {
        throw new Error(`Falha no upload. Status: ${response.status}`);
      }
    })
    .catch((error) => {
      console.error("Error:", error);
      alert("Erro ao subir a imagem.");
    });
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
  </div>
</template>

<style></style>
