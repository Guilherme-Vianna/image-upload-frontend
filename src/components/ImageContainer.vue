<script setup lang="ts">
const props = defineProps<{
  image: any;
}>();

import "primeicons/primeicons.css";
import {v4} from "uuid";
import {ref} from "vue";
import {useCookies} from "@vueuse/integrations/useCookies";
import {ELoadingStatus} from "@/interfaces/ELoadingStatus";

import axios from "axios";
import {API_URL} from "../../enviroments";

//#region Sessao
const cookies = useCookies(["sessionId"]);
const sessionId = ref<string>();

function loadSessionId() {
  const cookieValue = cookies.get("sessionId");
  if (cookieValue != null) {
    sessionId.value = cookieValue;
  } else {
    sessionId.value = v4();
    cookies.set("sessionId", sessionId.value);
  }
}

loadSessionId();

//#endregion

//#region Upload

let loading = ref<ELoadingStatus>(ELoadingStatus.LOADING);

function uploadImage() {
}

let bodyFormData = new FormData();
bodyFormData.append("sessionId", sessionId.value);
console.log(props.image);
bodyFormData.append("file", props.image);
axios.post(API_URL + "/image", bodyFormData).then((x) => {
  loading.value = ELoadingStatus.SUCESS;
}).catch(x => {
  loading.value = ELoadingStatus.ERROR;
});
uploadImage();
//#endregion
</script>

<template>
  <div
      class="flex justify-between items-center gap-20 min-w-150 w-full p-5 border-2 border-green-900 rounded-2xl"
  >
    <i class="pi pi-image"></i>
    <div class="font-light">{{ props.image.name }}</div>
    <div v-if="loading == ELoadingStatus.LOADING" class="loading-spinner"></div>
    <div v-if="loading == ELoadingStatus.SUCESS" class="pi pi-check  bg-green-700 rounded-full p-1"></div>
    <div v-if="loading == ELoadingStatus.ERROR" class="pi pi-ban  bg-green-700 rounded-full p-1"></div>
  </div>
</template>

<style>
.loading-spinner {
  border: 4px solid rgba(89, 255, 0, 0.1);
  border-left-color: #005eff;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
