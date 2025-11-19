<script setup>
import { inject } from "vue";
import { RouterLink } from "vue-router";

const isSignedIn = inject("is-signed-in");
</script>

<template>
  <nav
    class="flex sticky top-0 items-center justify-between w-full px-4 py-4 sm:px-8 lg:px-16 xl:px-20 2xl:px-40"
  >
    <RouterLink
      class="cursor-pointer text-2xl sm:text-3xl transition ease-in-out duration-200 hover:text-stone-400"
      to="/"
      >defAPI</RouterLink
    >
    <div v-if="isSignedIn" class="flex items-center gap-x-4 sm:gap-x-8">
      <RouterLink
        class="group flex cursor-pointer rounded-lg outline-1 outline-transparent px-3 py-2 sm:px-4 sm:py-2.5 transition duration-200 ease-in-out hover:outline-stone-400"
        to="/chats"
      >
        <span
          class="text-sm sm:text-base transition duration-200 ease-in-out group-hover:text-stone-400"
          >채팅 보기</span
        >
      </RouterLink>
      <RouterLink
        class="size-12 cursor-pointer rounded-full bg-stone-500 border-2 border-transparent transition duration-200 ease-in-out hover:border-stone-400"
        title="프로필 보기"
        to="/profile"
      >
        <img />
      </RouterLink>
    </div>
    <div v-else class="flex items-center gap-x-4 sm:gap-x-8">
      <RouterLink
        class="group flex cursor-pointer rounded-lg outline-1 outline-transparent px-3 py-2 sm:px-4 sm:py-2.5 transition duration-200 ease-in-out hover:outline-stone-400"
        to="/sign-in"
      >
        <span
          class="text-sm sm:text-base transition duration-200 ease-in-out group-hover:text-stone-400"
          >로그인</span
        >
      </RouterLink>
      <RouterLink
        class="group flex cursor-pointer rounded-lg bg-stone-500 px-3 py-2 sm:px-4 sm:py-2.5 transition duration-200 ease-in-out hover:bg-stone-400"
        to="/sign-up"
      >
        <span class="text-stone-900 text-sm sm:text-base">회원가입</span>
      </RouterLink>
    </div>
  </nav>
</template>
