<script setup lang="ts">
import {ref} from "vue";
import {useClipboard} from "@vueuse/core";
import {useCookies} from "@vueuse/integrations/useCookies";
import {v4 as uuidv4} from "uuid";
import axios from "axios";
import {ELoadingStatus} from "@/interfaces/ELoadingStatus";
import {API_URL} from "../../enviroments";
import "primeicons/primeicons.css";

const props = defineProps<{
  image: File;
}>();

const cookies = useCookies(["sessionId"]);
const sessionId = ref<string | undefined>();
const imageId = ref<string | undefined>();
const loading = ref<ELoadingStatus>(ELoadingStatus.LOADING);
const {copy} = useClipboard();
const isClicked = ref(false);

function loadSessionId() {
  const cookieValue = cookies.get("sessionId");
  sessionId.value = cookieValue || uuidv4();
  if (!cookieValue) {
    cookies.set("sessionId", sessionId.value);
  }
}

function uploadImage() {
  if (!props.image) return;

  const formData = new FormData();
  formData.append("sessionId", sessionId.value as string);
  formData.append("file", props.image);

  axios.post(`${API_URL}/image`, formData)
      .then((response) => {
        imageId.value = response.data.id;
        loading.value = ELoadingStatus.SUCCESS;
      })
      .catch(() => {
        loading.value = ELoadingStatus.ERROR;
      });
}

function shareImage() {
  if (!imageId.value || !sessionId.value) return;

  axios.get(`${API_URL}/image/share`, {
    params: {sessionId: sessionId.value, imageId: imageId.value}
  })
      .then((response) => {
        copy(response.data);
        triggerClickAnimation();
      })
      .catch(() => {
      });
}

function triggerClickAnimation() {
  isClicked.value = true;
  setTimeout(() => {
    isClicked.value = false;
  }, 500);
}

loadSessionId();
uploadImage();
</script>


<template>
  <div
      class="flex justify-between items-center gap-10 w-full p-6 border-2 border-gray-300 rounded-2xl shadow-md transition-all hover:shadow-lg"
  >
    <i class="pi pi-image text-3xl text-green-700"></i>
    <div class="font-semibold text-lg text-gray-800">{{ props.image.name }}</div>
    <div class="flex gap-6 items-center">
      <div v-if="loading === ELoadingStatus.LOADING" class="loading-spinner text-blue-500"></div>
      <div v-if="loading === ELoadingStatus.SUCCESS" class="pi pi-check text-white bg-green-500 rounded-full p-2"></div>
      <div v-if="loading === ELoadingStatus.ERROR" class="pi pi-ban text-white bg-red-500 rounded-full p-2"></div>

      <button
          v-if="loading === ELoadingStatus.SUCCESS"
          class="flex items-center bg-blue-600 text-white rounded-xl p-2 px-6 hover:bg-blue-700 hover:scale-105 transition-all duration-300 ease-in-out"
          :class="{ 'click-animation': isClicked }"
          @click="shareImage"
      >
        <div class="pi pi-share-alt mr-2">
          <div v-if="!isClicked">
            Compartilhar
          </div>
          <div v-if="isClicked">
            Copiado!
          </div>
        </div>
      </button>
      <button
          v-if="loading === ELoadingStatus.LOADING"
          class="pi pi-share-alt bg-gray-400 text-white rounded-xl p-2 px-6 cursor-not-allowed"
          disabled
      >
        Compartilhar
      </button>
    </div>
  </div>
</template>


<style scoped>
@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}

.loading-spinner {
  animation: pulse 1s infinite;
  border-top: 3px solid #4f86e0;
  border-right: 3px solid transparent;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  border-width: 3px;
  border-style: solid;
  display: inline-block;
  box-sizing: border-box;
}

/* Animation for button click effect */
.click-animation {
  animation: clickEffect 0.5s ease-in-out;
}

@keyframes clickEffect {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(0.95);
  }
  100% {
    transform: scale(1);
  }
}
</style>
