<script setup>
import { RouterLink, onBeforeRouteLeave } from "vue-router";
import { ref, onMounted, onBeforeUnmount, watch, computed } from "vue";
import Navbar from "@/components/Navbar.vue";
import UploadedFile from "@/components/UploadedFile.vue";
import UploadedFileModal from "@/components/UploadedFileModal.vue";

const isFirstInput = ref(true);
const awaitingResponse = ref(false);

const loadingTexts = [
  "보안 약점 분석 중.",
  "보안 약점 분석 중..",
  "보안 약점 분석 중...",
];
const loadingMsgIndex = ref(0);
const loadingText = ref(loadingTexts[0]);
let loadingInterval = null;

const startLoadingAnimation = () => {
  if (loadingInterval) {
    clearInterval(loadingInterval);
  }

  loadingInterval = setInterval(() => {
    loadingMsgIndex.value = (loadingMsgIndex.value + 1) % loadingTexts.length;
    loadingText.value = loadingTexts[loadingMsgIndex.value];
  }, 1200);
};

const stopLoadingAnimation = () => {
  if (loadingInterval) {
    clearInterval(loadingInterval);
    loadingInterval = null;
  }
};

const files = ref([]);
const message = ref("");
const fileInputRef = ref(null);

const submittedSource = ref(null);
const isSourceModalOpen = ref(false);

const openSourceModal = () => {
  isSourceModalOpen.value = true;
};

const closeSourceModal = () => {
  isSourceModalOpen.value = false;
};

const placeholderText = computed(() => {
  if (files.value.length > 0) {
    return "파일이 업로드되었습니다. 분석을 시작하세요.";
  }
  return "코드 파일을 업로드하거나 여기에 입력하세요";
});

watch(message, (newMessage) => {
  if (newMessage.trim() !== "") {
    files.value = [];
  }
});

const formatFileSize = (bytes) => {
  if (bytes === 0) return "0 Byte";
  const k = 1024;
  const sizes = ["Bytes", "KB", "MB", "GB"];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + " " + sizes[i];
};

const handleFileChange = (e) => {
  const newFile = e.target.files[0];

  if (!newFile) {
    return;
  }

  const newFileObject = {
    id: Date.now() + Math.random(),
    file: newFile,
    name: newFile.name,
    size: newFile.size,
    type: newFile.type,
  };

  files.value = [newFileObject];
  message.value = "";

  if (e.target) {
    e.target.value = null;
  }
};

const removeFile = (idToRemove) => {
  files.value = files.value.filter((file) => file.id !== idToRemove);
};

const handleSubmit = () => {
  if (files.value.length > 0 || message.value.trim() !== "") {
    isFirstInput.value = false;
    awaitingResponse.value = true;
    startLoadingAnimation();

    if (files.value.length > 0) {
      submittedSource.value = files.value[0];
    } else {
      submittedSource.value = {
        name: "code.txt",
        file: new File([message.value], "TypedInput.txt", {
          type: "text/plain",
        }),
        size: new Blob([message.value]).size, // 텍스트 크기
      };
    }

    setTimeout(() => {
      awaitingResponse.value = false;
      stopLoadingAnimation();
    }, 3000);
  }
};

const hasUnsavedData = computed(() => {
  return files.value.length > 0 || message.value.trim() !== "";
});

onBeforeRouteLeave((to, from, next) => {
  if (hasUnsavedData.value) {
    const confirmed = window.confirm(
      "정말 페이지를 떠나시겠습니까? 데이터는 저장되지 않습니다."
    );
    if (confirmed) {
      next();
    } else {
      next(false);
    }
  } else {
    next();
  }
});

const handleBeforeUnload = (e) => {
  if (hasUnsavedData.value) {
    e.preventDefault();
    return "정말 페이지를 새로고침하시겠습니까? 데이터는 저장되지 않습니다.";
  }
};

onMounted(() => {
  window.addEventListener("beforeunload", handleBeforeUnload);
});

onBeforeUnmount(() => {
  window.removeEventListener("beforeunload", handleBeforeUnload);
  stopLoadingAnimation();
});
</script>

<template>
  <div class="flex flex-col justify-between items-center gap-6 h-screen">
    <Navbar />
    <!-- 코드 입력 -->
    <main
      v-if="isFirstInput"
      class="flex flex-col grow gap-16 justify-center items-center w-full h-full px-16"
    >
      <svg
        v-if="message === '' && files.length === 0"
        class="size-16 sm:size-24 fill-stone-500"
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <g id="SVGRepo_bgCarrier" stroke-width="0"></g>
        <g
          id="SVGRepo_tracerCarrier"
          stroke-linecap="round"
          stroke-linejoin="round"
        ></g>
        <g id="SVGRepo_iconCarrier">
          <path
            d="M18.3281 5.67L6.58813 17.41C6.14813 17.85 5.40813 17.79 5.04813 17.27C3.80813 15.46 3.07812 13.32 3.07812 11.12V6.73C3.07812 5.91 3.69813 4.98 4.45813 4.67L10.0281 2.39C11.2881 1.87 12.6881 1.87 13.9481 2.39L17.9981 4.04C18.6581 4.31 18.8281 5.17 18.3281 5.67Z"
          ></path>
          <path
            d="M19.27 7.04159C19.92 6.49159 20.91 6.96159 20.91 7.81159V11.1216C20.91 16.0116 17.36 20.5916 12.51 21.9316C12.18 22.0216 11.82 22.0216 11.48 21.9316C10.06 21.5316 8.74001 20.8616 7.61001 19.9816C7.13001 19.6116 7.08001 18.9116 7.50001 18.4816C9.68001 16.2516 16.06 9.75159 19.27 7.04159Z"
          ></path>
        </g>
      </svg>

      <div v-if="files.length > 0" class="w-full max-w-4xl min-w-96 -mb-16">
        <div class="py-3 max-h-48 overflow-y-auto">
          <h2 class="text-sm font-semibold text-stone-500 mb-2">
            첨부된 파일:
          </h2>
          <div class="flex flex-wrap gap-2">
            <div class="flex flex-wrap gap-2">
              <UploadedFile
                v-for="file in files"
                :key="file.id"
                :file="file"
                :formatFileSize="formatFileSize"
                @remove="removeFile"
              />
            </div>
          </div>
        </div>
      </div>

      <form
        @submit.prevent="handleSubmit"
        action=""
        class="flex items-center justify-between gap-4 w-full h-fit max-w-4xl min-w-96"
      >
        <div
          class="flex items-center justify-between gap-4 w-full h-fit max-w-4xl min-w-96 rounded-full border-2 border-stone-500 py-5 px-5 sm:px-7 transition ease-in-out duration-200 focus-within:border-stone-400"
        >
          <input
            v-model="message"
            :placeholder="placeholderText"
            type="text"
            class="full-width-input w-full h-fit bg-transparent text-base text-stone-400 placeholder:text-stone-500 focus:outline-none disabled:cursor-not-allowed"
            :disabled="files.length > 0"
          />

          <div class="flex shrink-0 w-fit h-fit gap-3">
            <input
              type="file"
              accept=".js,.ts,.jsx,.tsx,.py,.java,.c,.cpp,.cs,.rb,.go,.php,.html,.css,.json,.xml,.yaml,.yml"
              id="file-upload"
              class="hidden"
              ref="fileInputRef"
              @change="handleFileChange"
              :disabled="message.trim() !== ''"
            />
            <label
              for="file-upload"
              title="코드 파일 업로드"
              :class="{
                'cursor-pointer hover:fill-stone-400': message.trim() === '',
                'opacity-50 cursor-not-allowed': message.trim() !== '',
              }"
            >
              <svg
                class="size-6 fill-stone-500 transition duration-200 hover:fill-stone-400"
                viewBox="0 0 24 24"
                fill="none"
                xmlns="http://www.w3.org/2000/svg"
              >
                <g id="SVGRepo_bgCarrier" stroke-width="0"></g>
                <g
                  id="SVGRepo_tracerCarrier"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                ></g>
                <g id="SVGRepo_iconCarrier">
                  <path
                    d="M20.5 10.19H17.61C15.24 10.19 13.31 8.26 13.31 5.89V3C13.31 2.45 12.86 2 12.31 2H8.07C4.99 2 2.5 4 2.5 7.57V16.43C2.5 20 4.99 22 8.07 22H15.93C19.01 22 21.5 20 21.5 16.43V11.19C21.5 10.64 21.05 10.19 20.5 10.19ZM11.53 13.53C11.38 13.68 11.19 13.75 11 13.75C10.81 13.75 10.62 13.68 10.47 13.53L9.75 12.81V17C9.75 17.41 9.41 17.75 9 17.75C8.59 17.75 8.25 17.41 8.25 17V12.81L7.53 13.53C7.24 13.82 6.76 13.82 6.47 13.53C6.18 13.24 6.18 12.76 6.47 12.47L8.47 10.47C8.54 10.41 8.61 10.36 8.69 10.32C8.71 10.31 8.74 10.3 8.76 10.29C8.82 10.27 8.88 10.26 8.95 10.25C8.98 10.25 9 10.25 9.03 10.25C9.11 10.25 9.19 10.27 9.27 10.3C9.28 10.3 9.28 10.3 9.29 10.3C9.37 10.33 9.45 10.39 9.51 10.45C9.52 10.46 9.53 10.46 9.53 10.47L11.53 12.47C11.82 12.76 11.82 13.24 11.53 13.53Z"
                  ></path>
                  <path
                    d="M17.4297 8.81048C18.3797 8.82048 19.6997 8.82048 20.8297 8.82048C21.3997 8.82048 21.6997 8.15048 21.2997 7.75048C19.8597 6.30048 17.2797 3.69048 15.7997 2.21048C15.3897 1.80048 14.6797 2.08048 14.6797 2.65048V6.14048C14.6797 7.60048 15.9197 8.81048 17.4297 8.81048Z"
                  ></path>
                </g>
              </svg>
            </label>
            <button
              type="submit"
              :disabled="files.length === 0 && message.trim() === ''"
              class="disabled:opacity-50 cursor-pointer disabled:cursor-not-allowed"
              title="분석 시작"
            >
              <svg
                class="size-6 fill-stone-500 transition duration-150 hover:fill-stone-400"
                viewBox="0 0 24 24"
                fill="none"
                xmlns="http://www.w3.org/2000/svg"
              >
                <g id="SVGRepo_bgCarrier" stroke-width="0"></g>
                <g
                  id="SVGRepo_tracerCarrier"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                ></g>
                <g id="SVGRepo_iconCarrier">
                  <path
                    d="M16.1391 2.95907L7.10914 5.95907C1.03914 7.98907 1.03914 11.2991 7.10914 13.3191L9.78914 14.2091L10.6791 16.8891C12.6991 22.9591 16.0191 22.9591 18.0391 16.8891L21.0491 7.86907C22.3891 3.81907 20.1891 1.60907 16.1391 2.95907ZM16.4591 8.33907L12.6591 12.1591C12.5091 12.3091 12.3191 12.3791 12.1291 12.3791C11.9391 12.3791 11.7491 12.3091 11.5991 12.1591C11.3091 11.8691 11.3091 11.3891 11.5991 11.0991L15.3991 7.27907C15.6891 6.98907 16.1691 6.98907 16.4591 7.27907C16.7491 7.56907 16.7491 8.04907 16.4591 8.33907Z"
                  ></path>
                </g>
              </svg>
            </button>
          </div>
        </div>
      </form>
    </main>
    <!-- 로딩 -->
    <main
      v-else-if="awaitingResponse"
      class="flex flex-col justify-center items-center gap-6 h-full"
    >
      <svg
        class="size-24 fill-stone-500 animate-pulse"
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <g id="SVGRepo_bgCarrier" stroke-width="0"></g>
        <g
          id="SVGRepo_tracerCarrier"
          stroke-linecap="round"
          stroke-linejoin="round"
        ></g>
        <g id="SVGRepo_iconCarrier">
          <path
            d="M5 13.5C4.06 13.5 3.19 13.83 2.5 14.38C1.58 15.11 1 16.24 1 17.5C1 19.71 2.79 21.5 5 21.5C6.01 21.5 6.93 21.12 7.64 20.5C8.47 19.77 9 18.7 9 17.5C9 15.29 7.21 13.5 5 13.5ZM6 17.75C6 18.01 5.86 18.26 5.64 18.39L4.39 19.14C4.27 19.22 4.13 19.25 4 19.25C3.75 19.25 3.5 19.12 3.36 18.89C3.15 18.53 3.26 18.07 3.62 17.86L4.51 17.33V16.25C4.51 15.84 4.85 15.5 5.26 15.5C5.67 15.5 6 15.84 6 16.25V17.75Z"
          ></path>
          <path
            d="M17.25 2.42969H7.75C4.9 2.42969 3 4.32969 3 7.17969V11.6397C3 11.9897 3.36 12.2397 3.7 12.1497C4.12 12.0497 4.55 11.9997 5 11.9997C7.86 11.9997 10.22 14.3197 10.48 17.1297C10.5 17.4097 10.73 17.6297 11 17.6297H11.55L15.78 20.4497C16.4 20.8697 17.25 20.4097 17.25 19.6497V17.6297C18.67 17.6297 19.86 17.1497 20.69 16.3297C21.52 15.4897 22 14.2997 22 12.8797V7.17969C22 4.32969 20.1 2.42969 17.25 2.42969ZM15.83 10.8097H9.17C8.78 10.8097 8.46 10.4897 8.46 10.0997C8.46 9.69969 8.78 9.37969 9.17 9.37969H15.83C16.22 9.37969 16.54 9.69969 16.54 10.0997C16.54 10.4897 16.22 10.8097 15.83 10.8097Z"
          ></path>
        </g>
      </svg>
      <span class="text-lg animate-pulse">{{ loadingText }}</span>
    </main>
    <!-- 결과 -->
    <main
      v-if="!awaitingResponse && !isFirstInput"
      class="flex flex-col justify-between items-center size-full gap-2"
    >
      <button
        @click="openSourceModal"
        class="group flex justify-center items-center px-4 py-2 rounded-xl border-2 border-stone-500 cursor-pointer"
      >
        <svg
          class="size-6 fill-pink-500/70 shrink-0"
          viewBox="0 0 24 24"
          fill="none"
          xmlns="http://www.w3.org/2000/svg"
        >
          <g id="SVGRepo_bgCarrier" stroke-width="0"></g>
          <g
            id="SVGRepo_tracerCarrier"
            stroke-linecap="round"
            stroke-linejoin="round"
          ></g>
          <g id="SVGRepo_iconCarrier">
            <path
              d="M15.7997 2.21048C15.3897 1.80048 14.6797 2.08048 14.6797 2.65048V6.14048C14.6797 7.60048 15.9197 8.81048 17.4297 8.81048C18.3797 8.82048 19.6997 8.82048 20.8297 8.82048C21.3997 8.82048 21.6997 8.15048 21.2997 7.75048C19.8597 6.30048 17.2797 3.69048 15.7997 2.21048Z"
            ></path>
            <path
              d="M20.5 10.19H17.61C15.24 10.19 13.31 8.26 13.31 5.89V3C13.31 2.45 12.86 2 12.31 2H8.07C4.99 2 2.5 4 2.5 7.57V16.43C2.5 20 4.99 22 8.07 22H15.93C19.01 22 21.5 20 21.5 16.43V11.19C21.5 10.64 21.05 10.19 20.5 10.19ZM11.5 17.75H7.5C7.09 17.75 6.75 17.41 6.75 17C6.75 16.59 7.09 16.25 7.5 16.25H11.5C11.91 16.25 12.25 16.59 12.25 17C12.25 17.41 11.91 17.75 11.5 17.75ZM13.5 13.75H7.5C7.09 13.75 6.75 13.41 6.75 13C6.75 12.59 7.09 12.25 7.5 12.25H13.5C13.91 12.25 14.25 12.59 14.25 13C14.25 13.41 13.91 13.75 13.5 13.75Z"
            ></path>
          </g>
        </svg>
        <span class="text-sm pl-2 text-stone-400 group-hover:underline"
          >원본 코드 보기</span
        >
      </button>
      <section class="flex justify-center items-center size-full gap-8">
        <div
          class="flex justify-center items-center rounded-lg border-2 border-stone-500 w-5/12 h-11/12"
        >
          <code></code>
        </div>
        <div
          class="flex justify-center items-center rounded-lg border-2 border-stone-500 w-5/12 h-11/12"
        >
          <p></p>
        </div>
      </section>
    </main>
  </div>

  <UploadedFileModal
    v-if="submittedSource"
    :is-open="isSourceModalOpen"
    @close="closeSourceModal"
    :file="submittedSource"
  />
</template>
