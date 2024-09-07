<script setup lang="ts">
import BaseButton from '@/components/atoms/BaseButton.vue';
import { computed } from 'vue';
const props = defineProps<{
  parkName: string | null;
  remainingSpace: number | null;
  price: string | null;
  distance: number | null;
  timePassed: number | null;
  maxTime: number | null;
  leaveEarly: boolean | null;
  billingTime: string | undefined;
  display:
    | 'browsing_map'
    | 'navigator_park'
    | 'navigator_yellow_line'
    | 'navigating'
    | 'park_confirm'
    | 'park_timer';
}>();
const emit = defineEmits([
  'button-go',
  'button-back',
  'button-mark-parked',
  'button-cancel-park',
  'button-cancel-navigating',
  'button-confirm-park',
  'button-leave'
]);
const isNavigatorPark = computed(() => props.display === 'navigator_park');
const isNavigatorYellowLine = computed(() => props.display === 'navigator_yellow_line');
const isNavigating = computed(() => props.display === 'navigating');
const isParkConfirm = computed(() => props.display === 'park_confirm');
const isParkTimer = computed(() => props.display === 'park_timer');
const warningThreshold = computed(() => (props.leaveEarly ? 30 : 150));
const pricePerHour = computed(() => parseFloat(props.price.replace('元/小時', '')));

const timerTextColor = computed(() => {
  if (props.timePassed === null || props.maxTime === null) {
    return 'text-grey-500';
  }
  const timeLeft = props.maxTime - props.timePassed;
  if (timeLeft < 0) {
    return 'text-warn-100';
  } else if (timeLeft < warningThreshold.value) {
    return 'text-orange-500';
  } else {
    return 'text-primary-500';
  }
});

const formattedTime = (time: number) => {
  const hours = Math.floor(time / 3600);
  const minutes = Math.floor(time / 60);
  const remainingSeconds = time % 60;
  return `${hours < 10 ? '0' + hours : hours}:${minutes < 10 ? '0' + minutes : minutes}:${remainingSeconds < 10 ? '0' + remainingSeconds : remainingSeconds}`;
};
</script>
<template>
  <div
    class="goto-card flex flex-col absolute bottom-0 bg-white rounded-lg p-4 mb-8"
    :class="{ show: props.display !== 'browsing_map' }"
  >
    <!-- 停車場 -->
    <div v-if="isNavigatorPark" class="container">
      <div class="text-2xl w-full text-center p-2">
        <span>{{ props.parkName }}</span>
        <span class="text-base text-gray-500"> (尚有{{ props.remainingSpace }}格)</span>
      </div>
      <div class="align-center text-center w-2x pt-2 pb-4">
        <p>收費時段：{{ props.billingTime || '--' }}</p>
        <p>收費：{{ props.price }}</p>
        <p>距離：{{ props.distance }} 公尺</p>
      </div>
      <div class="button-set">
        <BaseButton class="button button-go" @click="$emit('button-go')">前往</BaseButton>
        <BaseButton outline class="button button-back" @click="$emit('button-back')">
          取消
        </BaseButton>
      </div>
    </div>
    <!-- 黃線停車 -->
    <div v-else-if="isNavigatorYellowLine" class="container">
      <h2 class="text-2xl w-full text-center p-2">黃線</h2>
      <div class="align-center text-center w-2x pt-2 pb-4">
        <p>距離：{{ props.distance }}公尺</p>
      </div>
      <div class="button-set">
        <BaseButton class="button button-go" @click="$emit('button-go')">前往</BaseButton>
        <BaseButton outline class="button button-back" @click="$emit('button-back')">
          返回
        </BaseButton>
      </div>
    </div>
    <!-- 導航中 -->
    <div v-else-if="isNavigating" class="container">
      <h2 class="text-2xl w-full text-center p-2">導航中</h2>
      <div class="button-set">
        <BaseButton outline class="button" @click="$emit('button-cancel-navigating')"
          >取消導航</BaseButton
        >
      </div>
    </div>
    <!-- 停車確認 -->
    <div v-else-if="isParkConfirm" class="container">
      <h2 class="text-2xl w-full pb-6 pt-2 text-center">是否停好車了？</h2>
      <div class="button-set">
        <BaseButton class="button button-go" @click="$emit('button-confirm-park')">是</BaseButton>
        <BaseButton class="button button-back" @click="$emit('button-mark-parked')"
          >車位已有車</BaseButton
        >
        <BaseButton outline class="button button-back" @click="$emit('button-cancel-park')"
          >取消</BaseButton
        >
      </div>
    </div>
    <!-- 停車計時 -->
    <div v-else-if="isParkTimer" class="container">
      <h2 class="text-2xl w-full p-2 text-center">停車計時</h2>
      <div class="flex justify-center">
        <span :class="timerTextColor" class="text-xl font-bold pb-1">
          {{ formattedTime(props.timePassed) }}
        </span>
        <span class="text-grey-500 text-xl font-bold">/{{ formattedTime(props.maxTime) }}</span>
      </div>
      <div class="text-center pb-2">
        <span class="text-lg text-center"
          >已累計金額：{{ pricePerHour * (props.timePassed / 3600) }}元</span
        >
      </div>
      <div class="button-set">
        <BaseButton class="button button-go" @click="$emit('button-leave')">離開</BaseButton>
        <!-- <BaseButton class="button button-back" @click="$emit('button-back')">車位已停車</BaseButton> -->
        <!-- <BaseButton outline class="button button-back" @click="$emit('button-go')">取消</BaseButton> -->
      </div>
    </div>
  </div>
</template>
<style lang="postcss">
.goto-card {
  @apply flex;
  @apply flex-col;
  width: calc(100% - 32px);
  margin-left: 16px;
  margin-right: 16px;
  left: 100%;
  transition: left 0.5s;
}

.goto-card.show {
  left: 0;
}

.container {
  @apply flex;
  @apply flex-col;
}

.button-set {
  @apply flex;
  @apply justify-center;
  @apply w-full;
}

.button-set .button {
  @apply mx-2;
  @apply flex-1;
  max-width: 200px;
}
</style>
