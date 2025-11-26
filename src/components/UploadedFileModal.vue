<script setup>
import { ref, watch, nextTick, onMounted } from "vue";

const props = defineProps({
  file: {
    type: Object,
    required: true,
  },
  isOpen: {
    type: Boolean,
    required: true,
  },
});

const emit = defineEmits(["close"]);

const fileContent = ref("파일 내용 읽는 중...");
const fileReadError = ref(null);
const isLoading = ref(false);
const codeRef = ref(null);

const PRISM_CSS =
  "https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/themes/prism-tomorrow.min.css";
const PRISM_JS =
  "https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/prism.min.js";
const PRISM_JS_LANG_JS =
  "https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-javascript.min.js";
const PRISM_JS_LANG_PY =
  "https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-python.min.js";
const PRISM_JS_LANG_HTML =
  "https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-markup.min.js"; // For HTML/XML

const loadAsset = (type, url) => {
  if (document.querySelector(`${type}[href="${url}"], ${type}[src="${url}"]`)) {
    return Promise.resolve();
  }

  return new Promise((resolve, reject) => {
    if (type === "link") {
      const link = document.createElement("link");
      link.rel = "stylesheet";
      link.href = url;
      link.onload = resolve;
      link.onerror = reject;
      document.head.appendChild(link);
    } else if (type === "script") {
      const script = document.createElement("script");
      script.src = url;
      script.async = true;
      script.onload = resolve;
      script.onerror = reject;
      document.head.appendChild(script);
    }
  });
};

const highlightCode = () => {
  if (codeRef.value && window.Prism) {
    window.Prism.highlightElement(codeRef.value);
  }
};

const readFileContent = () => {
  if (!props.file || !props.file.file) {
    fileReadError.value = "파일 데이터가 유효하지 않습니다.";
    return;
  }

  isLoading.value = true;
  const reader = new FileReader();

  reader.onload = (e) => {
    fileContent.value = e.target.result;
    fileReadError.value = null;
    isLoading.value = false;

    nextTick(() => {
      highlightCode();
    });
  };

  reader.onerror = () => {
    fileReadError.value = `파일을 읽는 중 오류가 발생했습니다: ${reader.error.name}`;
    fileContent.value = "";
    isLoading.value = false;
    console.error(fileReadError.value, reader.error);
  };

  try {
    reader.readAsText(props.file.file);
  } catch (e) {
    fileReadError.value = `파일 읽기 시작 중 오류: ${e.message}`;
    fileContent.value = "";
    isLoading.value = false;
  }
};

watch(
  () => props.isOpen,
  (newVal) => {
    if (newVal) {
      fileContent.value = "파일 내용을 읽는 중...";
      fileReadError.value = null;
      readFileContent();
    }
  }
);

const closeModal = () => {
  emit("close");
};

const getLanguage = (fileName) => {
  const parts = fileName.split(".");
  if (parts.length > 1) {
    const ext = parts.pop().toLowerCase();
    const map = {
      js: "javascript",
      ts: "typescript",
      jsx: "jsx",
      tsx: "tsx",
      py: "python",
      java: "java",
      c: "c",
      cpp: "cpp",
      cs: "csharp",
      rb: "ruby",
      go: "go",
      php: "php",
      html: "html",
      css: "css",
      json: "json",
      xml: "xml",
      yaml: "yaml",
      yml: "yaml",
      vue: "html",
      sh: "bash",
      md: "markdown",
    };
    return map[ext] || "plaintext";
  }
  return "plaintext";
};

onMounted(async () => {
  loadAsset("link", PRISM_CSS);

  try {
    await loadAsset("script", PRISM_JS);
    await loadAsset("script", PRISM_JS_LANG_JS);
    await loadAsset("script", PRISM_JS_LANG_PY);
    await loadAsset("script", PRISM_JS_LANG_HTML);

    if (props.isOpen && fileContent.value !== "파일 내용 읽는 중...") {
      nextTick(highlightCode);
    }
  } catch (e) {
    console.error("Prism.js 로드 실패:", e);
  }
});
</script>

<template>
  <div
    v-if="props.isOpen"
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-stone-900/10 backdrop-blur-sm"
    @click.self="closeModal"
  >
    <div
      class="relative w-full max-w-4xl max-h-[90vh] bg-stone-900 border border-stone-700 rounded-xl flex flex-col"
    >
      <!-- Modal Header -->
      <div class="flex items-center justify-between p-4">
        <h3 class="text-xl font-medium text-stone-400 truncate pr-8">
          {{ props.file.name }}
        </h3>
        <button
          @click="closeModal"
          class="p-2 rounded-full text-stone-400 hover:bg-stone-700 transition duration-200 ease-in-out cursor-pointer"
          aria-label="Close modal"
        >
          <svg
            class="size-6"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M6 18L18 6M6 6l12 12"
            />
          </svg>
        </button>
      </div>

      <!-- Modal Body (Code Content) -->
      <div class="p-4 overflow-y-auto grow custom-scrollbar">
        <div v-if="isLoading" class="text-center p-8 text-stone-400">
          <div
            class="animate-spin inline-block size-6 border-2 border-current border-t-transparent text-pink-500 rounded-full"
            role="status"
          >
            <span class="sr-only">Loading...</span>
          </div>
          <p class="mt-2">파일 내용 읽는 중</p>
        </div>
        <pre
          v-else-if="fileReadError"
          class="text-pink-500 whitespace-pre-wrap p-4 bg-stone-900/10 rounded-lg"
        >
          {{ fileReadError }}
        </pre>
        <pre
          v-else
          class="text-xs sm:text-sm font-mono p-2 rounded-lg overflow-x-auto text-white custom-scrollbar-pre"
        ><code ref="codeRef" :class="`language-${getLanguage(props.file.name)}`">{{ fileContent }}</code></pre>
      </div>
    </div>
  </div>
</template>

<style scoped>
.custom-scrollbar::-webkit-scrollbar,
.custom-scrollbar-pre::-webkit-scrollbar {
  width: 8px;
  height: 8px;
  transition: ease-in-out 0.2s;
}

.custom-scrollbar::-webkit-scrollbar-track,
.custom-scrollbar-pre::-webkit-scrollbar-track {
  background: oklch(26.8% 0.007 34.298); /* stone-800 */
  border-radius: 10px;
}

.custom-scrollbar::-webkit-scrollbar-thumb,
.custom-scrollbar-pre::-webkit-scrollbar-thumb {
  background: oklch(55.3% 0.013 58.071); /* stone-400 */
  border-radius: 10px;
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover,
.custom-scrollbar-pre::-webkit-scrollbar-thumb:hover {
  background: oklch(70.9% 0.01 56.259); /* pink-500 */
}

pre[class*="language-"] {
  background-color: oklch(21.6% 0.006 56.043) !important; /* stone-900 */
}
</style>
