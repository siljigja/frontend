<script setup>
import { onBeforeRouteLeave } from "vue-router";
import { ref, onMounted, onBeforeUnmount, watch, computed } from "vue";
import Navbar from "@/components/Navbar.vue";
import UploadedFile from "@/components/UploadedFile.vue";
import UploadedFileModal from "@/components/UploadedFileModal.vue";

// JSON 응답 예시, 백엔드 연동시 제거
import mockResponse from "@/mock_response.json";

// PrismJS
import Prism from "prismjs";
import "prismjs/themes/prism-tomorrow.css";
// 언어
import "prismjs/themes/prism-tomorrow.css";
import "prismjs/components/prism-javascript";
import "prismjs/components/prism-typescript"; // tsx 포함
import "prismjs/components/prism-python";
import "prismjs/components/prism-markup"; // html, xml, svg
import "prismjs/components/prism-css";
import "prismjs/components/prism-json";
import "prismjs/components/prism-c";
import "prismjs/components/prism-cpp";
import "prismjs/components/prism-go";
import "prismjs/components/prism-bash";
import "prismjs/components/prism-yaml";

// JSON 응답 예시 변수
const MOCK_RESPONSE = mockResponse;

// 상태
const isFirstInput = ref(true);
const awaitingResponse = ref(false);
const isCopied = ref(false);

const loadingTexts = [
  "보안 약점 분석 중.",
  "보안 약점 분석 중..",
  "보안 약점 분석 중...",
];
const loadingMsgIndex = ref(0);
const loadingText = ref(loadingTexts[0]);
let loadingInterval = null;

const analysisResult = ref(null);
const currentIssueIndex = ref(0);

const files = ref([]);
const message = ref("");
const fileInputRef = ref(null);
const submittedSource = ref(null);
const isSourceModalOpen = ref(false);

// 로딩 애니메이션
const startLoadingAnimation = () => {
  if (loadingInterval) clearInterval(loadingInterval);
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

// 모달
const openSourceModal = () => {
  isSourceModalOpen.value = true;
};
const closeSourceModal = () => {
  isSourceModalOpen.value = false;
};

// 입력 처리
const placeholderText = computed(() => {
  if (files.value.length > 0)
    return "파일이 업로드되었습니다. 분석을 시작하세요.";
  return "코드 파일을 업로드하거나 직접 입력하세요.";
});

watch(message, (newMessage) => {
  if (newMessage.trim() !== "") files.value = [];
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
  if (!newFile) return;
  const newFileObject = {
    id: Date.now() + Math.random(),
    file: newFile,
    name: newFile.name,
    size: newFile.size,
    type: newFile.type,
  };
  files.value = [newFileObject];
  message.value = "";
  if (e.target) e.target.value = null;
};

const removeFile = (idToRemove) => {
  files.value = files.value.filter((file) => file.id !== idToRemove);
};

// 제출 처리
// ==================================================
// 백엔드 연동시 여기 코드 수정 필요
// ==================================================
const handleSubmit = () => {
  if (files.value.length > 0 || message.value.trim() !== "") {
    // 제출 처리
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
        size: new Blob([message.value]).size,
      };
    }

    // setTimeout은 백엔드 연동시 제거
    // 결과 나왔을 때 awaitingResponse.value = false; stopLoadingAnimation(); 실행 필요
    setTimeout(() => {
      // Mock response for analysis result
      analysisResult.value = MOCK_RESPONSE;
      currentIssueIndex.value = 0;

      awaitingResponse.value = false;
      stopLoadingAnimation();
    }, 3000);
  }
};
// ==================================================
// 수정 필요 코드 끝
// ==================================================

// 결과 처리
const getLanguage = (fileName) => {
  if (!fileName) return "plaintext";
  const parts = fileName.split(".");
  if (parts.length > 1) {
    const ext = parts.pop().toLowerCase();
    const map = {
      js: "javascript",
      ts: "typescript",
      jsx: "javascript",
      tsx: "typescript",
      py: "python",
      java: "java",
      c: "c",
      cpp: "cpp",
      cs: "csharp",
      go: "go",
      html: "markup",
      css: "css",
      json: "json",
      yaml: "yaml",
      vue: "markup",
      sh: "bash",
      md: "markdown",
    };
    return map[ext] || "plaintext";
  }
  return "plaintext";
};

const currentIssue = computed(() => {
  if (!analysisResult.value || !analysisResult.value.issues) return null;
  if (analysisResult.value.issues.length === 0) return null;
  return analysisResult.value.issues[currentIssueIndex.value];
});

const issueStyle = computed(() => {
  if (currentIssue.value && currentIssue.value.display_meta) {
    const color = currentIssue.value.display_meta.severity_color || "#666";
    return {
      borderColor: color,
      color: color,
    };
  }
  return {};
});

const highlightedCode = computed(() => {
  if (!currentIssue.value?.fix_code) return "";

  const fileName = submittedSource.value?.name || "code.js";
  const langAlias = getLanguage(fileName);
  const grammar = Prism.languages[langAlias];

  return grammar
    ? Prism.highlight(currentIssue.value.fix_code, grammar, langAlias)
    : Prism.util.encode(currentIssue.value.fix_code);
});

const handleCopyCode = async () => {
  if (!currentIssue.value?.fix_code) return;
  try {
    const textarea = document.createElement("textarea");
    textarea.value = currentIssue.value.fix_code;
    textarea.style.position = "fixed";
    document.body.appendChild(textarea);
    textarea.focus();
    textarea.select();
    document.execCommand("copy");
    document.body.removeChild(textarea);

    isCopied.value = true;
    setTimeout(() => {
      isCopied.value = false;
    }, 2000);
  } catch (err) {
    console.error("코드 복사 실패", err);
  }
};

const nextIssue = () => {
  if (
    analysisResult.value &&
    currentIssueIndex.value < analysisResult.value.issues.length - 1
  ) {
    currentIssueIndex.value++;
  }
};

const prevIssue = () => {
  if (currentIssueIndex.value > 0) {
    currentIssueIndex.value--;
  }
};

const hasUnsavedData = computed(() => {
  return files.value.length > 0 || message.value.trim() !== "";
});

// 페이지 이탈 방지
onBeforeRouteLeave((to, from, next) => {
  if (hasUnsavedData.value) {
    // 추후 window.confirm 대신 커스텀 모달로 변경 가능
    const confirmed = window.confirm(
      "정말 페이지를 떠나시겠습니까?\n현재 데이터는 저장되지 않습니다."
    );
    if (confirmed) next();
    else next(false);
  } else {
    next();
  }
});

const handleBeforeUnload = (e) => {
  if (hasUnsavedData.value) {
    e.preventDefault();
    return "";
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

    <!-- 1. 입력 -->
    <main
      v-if="isFirstInput"
      class="flex flex-col grow gap-16 justify-center items-center w-full h-full px-16"
    >
      <svg
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
              accept=".js,.ts,.jsx,.tsx,.py,.java,.c,.cpp,.cs,.rb,.go,.php,.html,.css,.json,.xml,.yaml,.yml, .vue,.sh,.md"
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
                    d="M20.5 10.19H17.61C15.24 10.19 13.31 8.26 13.31 5.89V3C13.31 2.45 12.86 2 12.31 2H8.07C4.99 2 2.5 4 2.5 7.57V16.43C2.5 20 4.99 22 8.07 22H15.93C19.01 22 21.5 20 21.5 16.43V11.19C21.5 10.64 21.05 10.19 20.5 10.19ZM11.53 13.53C11.38 13.68 11.19 13.75 11 13.75C10.81 13.75 10.62 13.68 10.47 13.53L9.75 12.81V17C9.75 17.41 9.41 17.75 9.00 17.75C8.59 17.75 8.25 17.41 8.25 17V12.81L7.53 13.53C7.24 13.82 6.76 13.82 6.47 13.53C6.18 13.24 6.18 12.76 6.47 12.47L8.47 10.47C8.54 10.41 8.61 10.36 8.69 10.32C8.71 10.31 8.74 10.3 8.76 10.29C8.82 10.27 8.88 10.26 8.95 10.25C8.98 10.25 9 10.25 9.03 10.25C9.11 10.25 9.19 10.27 9.27 10.3C9.28 10.3 9.28 10.3 9.29 10.3C9.37 10.33 9.45 10.39 9.51 10.45C9.52 10.46 9.53 10.46 9.53 10.47L11.53 12.47C11.82 12.76 11.82 13.24 11.53 13.53Z"
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

    <!-- 2. 로딩 -->
    <main
      v-else-if="awaitingResponse"
      class="flex flex-col justify-center items-center gap-6 h-full"
    >
      <svg
        class="size-16 sm:size-24 fill-stone-500 animate-pulse"
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
      <span class="text-lg animate-pulse">{{ loadingText }}</span>
    </main>

    <!-- 3. 결과 -->
    <main
      v-else-if="!awaitingResponse && !isFirstInput"
      class="flex flex-col justify-start items-center size-full gap-2 pt-4 pb-8 overflow-hidden"
    >
      <button
        @click="openSourceModal"
        class="group flex justify-center items-center px-4 py-2 rounded-xl border border-stone-500 cursor-pointer mb-4"
      >
        <svg
          class="size-5 fill-stone-500 shrink-0"
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
        <span
          class="text-sm pl-2 text-stone-400 group-hover:underline"
          title="원본 코드 보기"
          >Original Code</span
        >
      </button>

      <section
        class="flex justify-center items-stretch size-full gap-8 h-full px-8 min-h-0"
      >
        <!-- 왼쪽: 수정된 코드 -->
        <div
          class="group flex flex-col rounded-xl border border-stone-500 w-5/12 h-full overflow-hidden relative"
        >
          <!-- 헤더 -->
          <div
            class="flex items-center justify-between bg-stone-900 px-4 py-2 border-b border-stone-500 text-sm h-11"
          >
            <span class="text-stone-400 font-medium">수정된 코드 (Fixed)</span>
            <span
              v-if="isCopied"
              class="flex items-center gap-2 text-xs text-stone-400 uppercase cursor-default"
              title="복사 완료"
              >Copied to Clipboard</span
            >
            <span
              v-else-if="currentIssue"
              @click="handleCopyCode"
              class="text-xs text-stone-500 uppercase hover:underline group-hover:text-stone-400 cursor-pointer"
              title="클릭해서 복사"
              >Click to Copy</span
            >
          </div>

          <pre
            v-if="currentIssue"
            class="scrollbar-thin scrollbar-thumb-stone-600 scrollbar-track-stone-800 overflow-auto h-full p-4 m-0 text-sm"
          ><code v-html="highlightedCode" :class="`language-${getLanguage(submittedSource?.name)}`"></code></pre>
          <div
            v-else
            class="flex flex-col justify-center items-center gap-6 h-full text-stone-500"
          >
            <svg
              class="size-12 fill-stone-500"
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
                  d="M5.97 1H3.03C1.76 1 1 1.76 1 3.03V5.97C1 7.24 1.76 8 3.03 8H5.97C7.24 8 8 7.24 8 5.97V3.03C8 1.76 7.24 1 5.97 1ZM6.47 5.56C6.72 5.81 6.72 6.22 6.47 6.47C6.34 6.59 6.17 6.65 6.01 6.65C5.85 6.65 5.69 6.59 5.56 6.47L4.49 5.41L3.45 6.47C3.32 6.59 3.16 6.65 2.98 6.65C2.82 6.65 2.66 6.59 2.53 6.47C2.28 6.22 2.28 5.81 2.53 5.56L3.6 4.5L2.54 3.45C2.29 3.2 2.29 2.79 2.54 2.54C2.79 2.29 3.2 2.29 3.45 2.54L4.49 3.6L5.55 2.54C5.8 2.29 6.21 2.29 6.46 2.54C6.71 2.79 6.71 3.2 6.46 3.45L5.41 4.5L6.47 5.56Z"
                ></path>
                <path
                  d="M21.4995 15.8197C21.4995 15.9697 21.4495 16.1197 21.3195 16.2497C19.8695 17.7097 17.2895 20.3097 15.8095 21.7997C15.6795 21.9397 15.5095 21.9997 15.3395 21.9997C15.0095 21.9997 14.6895 21.7397 14.6895 21.3597V17.8597C14.6895 16.3997 15.9295 15.1897 17.4495 15.1897C18.3995 15.1797 19.7195 15.1797 20.8495 15.1797C21.2395 15.1797 21.4995 15.4897 21.4995 15.8197Z"
                ></path>
                <path
                  d="M21.4995 15.8197C21.4995 15.9697 21.4495 16.1197 21.3195 16.2497C19.8695 17.7097 17.2895 20.3097 15.8095 21.7997C15.6795 21.9397 15.5095 21.9997 15.3395 21.9997C15.0095 21.9997 14.6895 21.7397 14.6895 21.3597V17.8597C14.6895 16.3997 15.9295 15.1897 17.4495 15.1897C18.3995 15.1797 19.7195 15.1797 20.8495 15.1797C21.2395 15.1797 21.4995 15.4897 21.4995 15.8197Z"
                ></path>
                <path
                  d="M16.63 2H10.5C9.95 2 9.5 2.45 9.5 3V6.5C9.5 8.16 8.16 9.5 6.5 9.5H3.5C2.95 9.5 2.5 9.95 2.5 10.5V17.13C2.5 19.82 4.68 22 7.37 22H12.19C12.74 22 13.19 21.55 13.19 21V17.86C13.19 15.56 15.1 13.69 17.45 13.69C17.98 13.68 19.27 13.68 20.5 13.68C21.05 13.68 21.5 13.24 21.5 12.68V6.87C21.5 4.18 19.32 2 16.63 2ZM8.72 17.01H6.08C5.67 17.01 5.33 16.67 5.33 16.26C5.33 15.84 5.67 15.5 6.08 15.5H8.72C9.15 15.5 9.47 15.84 9.47 16.26C9.47 16.67 9.15 17.01 8.72 17.01ZM11.51 13.3H6.08C5.67 13.3 5.33 12.96 5.33 12.55C5.33 12.13 5.67 11.79 6.08 11.79H11.51C11.92 11.79 12.27 12.13 12.27 12.55C12.27 12.96 11.92 13.3 11.51 13.3Z"
                ></path>
              </g>
            </svg>
            수정된 코드가 없습니다.
          </div>
        </div>

        <!-- 오른쪽: 설명 -->
        <div
          class="flex flex-col rounded-xl border border-stone-500 w-5/12 h-full overflow-hidden"
        >
          <!-- 헤더 & 내비게이션 -->
          <div
            class="flex justify-between items-center bg-stone-900 px-4 py-2 border-b border-stone-500 h-11"
          >
            <div v-if="currentIssue" class="text-sm text-stone-400">
              발견된 취약점 {{ currentIssueIndex + 1 }} /
              {{ analysisResult.issues.length }}
            </div>
            <div v-else class="text-sm text-stone-400">발견된 취약점 0 / 0</div>
            <div v-if="currentIssue" class="flex gap-2">
              <button
                @click="prevIssue"
                :disabled="currentIssueIndex === 0"
                class="text-xs px-2 py-1 rounded border border-stone-600 hover:bg-stone-700 cursor-pointer disabled:opacity-30 disabled:cursor-not-allowed text-stone-300"
              >
                이전
              </button>
              <button
                @click="nextIssue"
                :disabled="
                  currentIssueIndex === analysisResult.issues.length - 1
                "
                class="text-xs px-2 py-1 rounded border border-stone-600 hover:bg-stone-700 cursor-pointer disabled:opacity-30 disabled:cursor-not-allowed text-stone-300"
              >
                다음
              </button>
            </div>
          </div>

          <!-- 내용 -->
          <div
            class="flex flex-col p-6 gap-6 overflow-y-auto scrollbar-thin scrollbar-thumb-stone-600 h-full"
          >
            <div>
              <span
                class="text-stone-500 text-xs font-bold uppercase tracking-wider"
                >Vulnerability</span
              >
              <h2
                v-if="currentIssue"
                class="text-xl font-bold text-stone-200 mt-1"
              >
                {{ currentIssue.name }}
              </h2>
              <h2 v-else class="text-xl font-bold text-stone-200 mt-1">
                취약점 미발견
              </h2>
            </div>

            <div class="flex gap-4 items-center mt-4">
              <span
                v-if="currentIssue"
                class="px-3 py-1 rounded-full text-sm font-semibold uppercase border-2 text-white"
                :style="issueStyle"
              >
                {{ currentIssue.severity }}
              </span>
              <span
                v-else
                class="px-3 py-1 rounded-full text-sm font-semibold uppercase border-2 text-green-500 border-green-500"
              >
                Safe
              </span>
              <span v-if="currentIssue" class="text-stone-400 text-sm"
                >Confidence:
                {{ (currentIssue.confidence * 100).toFixed(0) }}%</span
              >
            </div>

            <div class="border-t border-stone-700 pt-4 mt-4">
              <span
                class="text-stone-500 text-xs font-bold uppercase tracking-wider"
                >Description & Analysis</span
              >
              <p
                v-if="currentIssue"
                class="text-stone-300 mt-2 text-sm leading-relaxed whitespace-pre-wrap"
              >
                {{ currentIssue.description }}
              </p>
              <p
                v-else
                class="text-stone-300 mt-2 text-sm leading-relaxed whitespace-pre-wrap"
              >
                발견된 취약점이 적습니다. 기본 보안 점검을 계속 유지하세요.
              </p>
            </div>

            <div
              v-if="currentIssue && currentIssue.exploit_example"
              class="bg-stone-800/50 rounded-lg p-4 border border-stone-700 mt-4"
            >
              <span
                class="text-xs font-bold uppercase tracking-wider"
                :style="{
                  color:
                    currentIssue.display_meta.severity_color ||
                    'oklch(59.2% 0.249 0.584)',
                }"
                >Potential Exploit Scenario</span
              >
              <p class="text-stone-400 mt-1 text-xs leading-relaxed">
                {{ currentIssue.exploit_example }}
              </p>
            </div>
          </div>
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

<style>
code[class*="language-"],
pre[class*="language-"] {
  background-color: oklch(21.6% 0.006 56.043) !important; /* stone-900 */
  text-shadow: none !important;
  font-family: Consolas, Monaco, "Andale Mono", "Ubuntu Mono", monospace;
  font-size: 12px;
}

.scrollbar-thin::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}
.scrollbar-thin::-webkit-scrollbar-track {
  background: oklch(26.8% 0.007 34.298); /* stone-800 */
}
.scrollbar-thin::-webkit-scrollbar-thumb {
  background: oklch(55.3% 0.013 58.071); /* stone-600 */
  border-radius: 4px;
}
</style>
