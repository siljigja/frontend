<script setup>
import Navbar from "@/components/Navbar.vue";
import { RouterLink } from "vue-router";
import { inject } from "vue";
const isSignedIn = inject("is-signed-in");
</script>

<template>
  <div class="flex flex-col h-screen justify-between items-center gap-24">
    <Navbar />
    <main
      class="flex flex-col items-center justify-between w-full h-full max-w-6xl px-4 md:px-8"
    >
      <div class="flex items-start justify-between w-full h-fit gap-6">
        <div class="flex flex-col w-auto h-fit gap-4 sm:gap-8">
          <svg
            class="size-10 sm:size-16 fill-stone-500"
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
                d="M20.9098 11.1203V6.73031C20.9098 5.91031 20.2898 4.98031 19.5198 4.67031L13.9498 2.39031C12.6998 1.88031 11.2898 1.88031 10.0398 2.39031L4.46984 4.67031C3.70984 4.98031 3.08984 5.91031 3.08984 6.73031V11.1203C3.08984 16.0103 6.63984 20.5903 11.4898 21.9303C11.8198 22.0203 12.1798 22.0203 12.5098 21.9303C17.3598 20.5903 20.9098 16.0103 20.9098 11.1203ZM12.7498 12.8703V15.5003C12.7498 15.9103 12.4098 16.2503 11.9998 16.2503C11.5898 16.2503 11.2498 15.9103 11.2498 15.5003V12.8703C10.2398 12.5503 9.49984 11.6103 9.49984 10.5003C9.49984 9.12031 10.6198 8.00031 11.9998 8.00031C13.3798 8.00031 14.4998 9.12031 14.4998 10.5003C14.4998 11.6203 13.7598 12.5503 12.7498 12.8703Z"
              ></path>
            </g>
          </svg>
          <p class="text-3xl sm:text-4xl lg:text-5xl uppercase">
            Keep your code
            <br />
            <span
              class="text-stone-400 text-4xl sm:text-5xl lg:text-6xl uppercase"
              >Secure.</span
            >
          </p>
        </div>

        <p class="w-1/3 h-fit max-w-lg text-sm md:text-base">
          프로젝트 기능 설명 Lorem ipsum dolor sit amet consectetur. Faucibus
          volutpat elit sed lacinia. Justo imperdiet ullamcorper est at
          malesuada. Aliquam sagittis at mauris sit ut ultrices lacus blandit
          risus. Ultrices suscipit diam orci tempus ac.
        </p>
      </div>

      <div
        v-if="isSignedIn"
        class="flex flex-col items-center justify-center w-fit h-fit gap-2.5 mt-12 mb-16"
      >
        <span class="text-xl">Start the Analysis</span>
        <svg
          class="size-8 fill-stone-500"
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
              d="M17.9188 8.17969H11.6888H6.07877C5.11877 8.17969 4.63877 9.33969 5.31877 10.0197L10.4988 15.1997C11.3288 16.0297 12.6788 16.0297 13.5088 15.1997L15.4788 13.2297L18.6888 10.0197C19.3588 9.33969 18.8788 8.17969 17.9188 8.17969Z"
            ></path>
          </g>
        </svg>
      </div>
      <RouterLink
        v-else
        class="group flex items-center justify-center w-fit h-fit gap-2.5 cursor-pointer mt-12 mb-16"
        to="/sign-in"
      >
        <svg
          class="size-5 fill-stone-500"
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
              d="M17.8796 6.12156L3.9196 20.0816C3.4896 20.5116 2.7696 20.4716 2.4196 19.9816C1.9196 19.2916 1.8196 18.3316 2.3396 17.5016L5.0996 13.0716C5.4696 12.4816 5.4696 11.5216 5.0996 10.9316L2.3396 6.50156C1.4096 5.02156 2.4796 3.10156 4.2196 3.10156H15.6696C16.3496 3.10156 17.1896 3.57156 17.5496 4.14156L18.0196 4.88156C18.2596 5.28156 18.2096 5.79156 17.8796 6.12156Z"
            ></path>
            <path
              d="M21.6299 13.1119L16.4499 20.0119C16.0899 20.5019 15.2899 20.9019 14.6699 20.9019H7.50987C6.61987 20.9019 6.16987 19.8219 6.79987 19.1919L18.3199 7.68195C18.7699 7.23195 19.5399 7.31195 19.8799 7.86195L21.7299 10.8319C22.1299 11.4719 22.0899 12.5019 21.6299 13.1119Z"
            ></path>
          </g>
        </svg>
        <span class="text-stone-400 text-xl group-hover:underline"
          >지금 시작하기</span
        >
      </RouterLink>

      <RouterLink
        v-if="isSignedIn"
        class="w-full max-w-5xl h-fit cursor-pointer rounded-t-3xl transition duration-200 ease-in-out shadow-[0_0_15px_5px]/40 shadow-stone-400/50 hover:shadow-[0_0_20px_6px]/70 hover:shadow-pink-500/50"
        to="/analysis"
        title="취약점 분석 시작"
      >
        <section
          class="flex flex-col items-center justify-end w-full h-fit gap-10 md:gap-16 rounded-t-3xl border-2 border-b-0 border-stone-500 py-12 sm:py-16 md:py-20 px-36"
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

          <div
            class="flex items-center justify-between w-full h-14 sm:h-16 min-w-96 rounded-full border-2 border-stone-500 px-5 sm:px-7"
          >
            <input
              type="text"
              placeholder="코드 파일을 업로드하거나 직접 입력하세요"
              class="full-width-input w-full h-fit bg-transparent text-base placeholder:text-stone-500 focus:outline-none cursor-pointer"
              disabled
            />

            <div class="flex shrink-0 w-fit h-fit gap-3">
              <svg
                class="size-6 fill-stone-500 transition duration-150"
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
              <svg
                class="size-6 fill-stone-500 transition duration-150"
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
            </div>
          </div>
        </section>
      </RouterLink>
      <div v-else class="w-full max-w-5xl h-fit cursor-not-allowed">
        <section
          class="flex flex-col items-center justify-end w-full h-fit gap-10 md:gap-16 rounded-t-3xl border-2 border-b-0 border-stone-400 bg-stone-800 py-12 sm:py-16 md:py-20 px-36"
        >
          <svg
            class="size-16 sm:size-24 fill-stone-400"
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

          <div
            class="flex items-center justify-between w-full h-14 sm:h-16 min-w-80 rounded-full border-2 border-stone-400 px-5 sm:px-7"
          >
            <input
              type="text"
              placeholder="코드 파일을 업로드하거나 직접 입력하세요"
              class="full-width-input w-full h-fit bg-transparent text-base placeholder:text-stone-400 focus:outline-none cursor-not-allowed"
              disabled
            />

            <div class="flex shrink-0 w-fit h-fit gap-3">
              <svg
                class="size-6 fill-stone-400 transition duration-150"
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
              <svg
                class="size-6 fill-stone-400 transition duration-150"
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
            </div>
          </div>
        </section>
      </div>
    </main>
  </div>
</template>
