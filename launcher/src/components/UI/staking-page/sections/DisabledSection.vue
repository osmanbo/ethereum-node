<template>
  <div class="full-body w-full h-full absolute grid grid-cols-24 grid-rows-12 z-10">
    <div
      class="disable-layer grid grid-cols-12 grid-rows-12 col-start-2 col-end-20 row-start-1 row-end-12 bg-[#252525] z-10 rounded-md"
      @mousedown.prevent
    >
      <div class="col-start-2 col-end-12 row-start-2 col-span-full row-end-8 flex justify-center items-center">
        <img class="w-2/3" src="/img/icon/staking-page-icons/staking-disabled.png" alt="icon" />
      </div>
      <div
        class="col-start-2 col-end-12 col-span-full row-start-10 row-end-11 flex justify-center items-center text-center mt-2"
      >
        <span class="text-cyan-400 text-2xl font-semibold font-sans">{{ t("disableStake.message") }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import i18n from "@/includes/i18n";

const t = i18n.global.t;
</script>
