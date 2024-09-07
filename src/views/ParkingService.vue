<script setup lang="ts">
import Map from '@/components/organisms/Map.vue';
import 'mapbox-gl/dist/mapbox-gl.css';
import NavigatorCard from '@/components/organisms/NavigatorCard.vue';
import { ref } from 'vue';
import { computed } from 'vue';

export type ParkPoint = {
    name: string;
    address: string;
    lat: number;
    lng: number;
    remainingSpace: number;
    price: string;
    distance: number;
    type: 'park' | 'yellow_line';
};

enum NavigatorStep {
    BrowsingMap = 'browsing_map',   // 瀏覽地圖，沒有顯示任何卡片
    NavigatorPark = 'navigator_park',   // 查看停車場
    NavigatorYellowLine = 'navigator_yellow_line',  // 查看黃線
    Navigatoing = 'navigating', // 導航中
    ParkConfirm = 'park_confirm',   // 確認停車
    ParkTimer = 'park_timer'    // 停車計時
}

const currentSelectedPark = ref<ParkPoint | null>({
    name: 'sussy',
    address: 'safa',
    lat: 100,
    lng: 24,
    remainingSpace: 48763,
    price: '40圓/小時',
    distance: 20,
    type: 'park'
});
const currentStep = ref<NavigatorStep>(NavigatorStep.NavigatorPark);
const handleMapClick = (point: ParkPoint) => {
    currentSelectedPark.value = point;
};

// 前往目的地
const handleGoClick = () => {
    // TODO: pop up the routing card
    currentStep.value = NavigatorStep.Navigatoing;
};

// 返回地圖
const handleBackClick = () => {
    currentSelectedPark.value = null;
    currentStep.value = NavigatorStep.BrowsingMap;
};

// 取消導航，回到地標
const handleCancelNavigating = () => {
    currentStep.value = currentSelectedPark.value.type === "park" ? NavigatorStep.NavigatorPark : NavigatorStep.NavigatorYellowLine;
};

// 確認停車
const handleConfirmPark = () => {
    // TODO: pop up the parking timer card
    currentStep.value = NavigatorStep.ParkTimer;
};

// 標示為已停車
const handleMarkParked = () => {
    // TODO: back to the map, send the parking information to the server
};

// 取消停車
const handleCancelPark = () => {
    // back to the point
    currentStep.value = currentSelectedPark.value.type === "park" ? NavigatorStep.NavigatorPark : NavigatorStep.NavigatorYellowLine;
};

// 開車閃人
const handleLeave = () => {
    // back to the map
    currentSelectedPark.value = null;
    currentStep.value = NavigatorStep.BrowsingMap;
};

const backClickHandler = ref(handleBackClick);
const goClickHandler = ref(handleGoClick);
const cancelNavigatingHandler = ref(handleCancelNavigating);
const pointClickHandler = ref(handleMapClick);
const isShowNavigatorCard = computed(() => currentSelectedPark.value !== null);

// Usage: <Map @point-click="pointClickHandler" />
</script>
<template>
    <div class="h-screen flex flex-col overflow-hidden relative">
        <div class="flex flex-1">
            <Map @point-click="pointClickHandler" />
        </div>
        <NavigatorCard v-show="isShowNavigatorCard" :parkName="currentSelectedPark?.name ?? null"
            :remainingSpace="currentSelectedPark?.remainingSpace ?? null" :price="currentSelectedPark?.price ?? null"
            :distance="currentSelectedPark?.distance ?? null" :address="currentSelectedPark?.address ?? null"
            :display="currentStep" :timePassed="null" :maxTime="null" :leaveEarly="true"
            @button-back="backClickHandler" @button-go="goClickHandler"
            @button-cancel-navigating="cancelNavigatingHandler" />
    </div>
</template>
