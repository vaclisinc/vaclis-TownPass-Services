<script setup lang="ts">
import Map from '@/components/organisms/Map.vue';
import 'mapbox-gl/dist/mapbox-gl.css';
import NavigatorCard from "@/components/organisms/NavigatorCard.vue";
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
    display: "navigator_park" | "navigator_yellow_line" | "park_confirm" | "park_timer";
};

const currentSelectedPark = ref<ParkPoint | null>({
    name: 'sussy',
    address: 'safa',
    lat: 100,
    lng: 24,
    remainingSpace: 48763,
    price: '40圓/小時',
    distance: 20,
    display: 'navigator_park'
});

const handleClick = (point: ParkPoint) => {
    currentSelectedPark.value = point;
};


const pointClickHandler = ref(handleClick);
const isShowNavigatorCard = computed(() => currentSelectedPark.value !== null);

// Usage: <Map @point-click="pointClickHandler" />
</script>
<template>
    <div class="h-screen flex flex-col overflow-hidden position-relative">
        <div class="flex flex-1">
            <Map @point-click="pointClickHandler" />
        </div>
        <NavigatorCard v-show="isShowNavigatorCard" :parkName="currentSelectedPark?.name" :remainingSpace="currentSelectedPark?.remainingSpace" :price="currentSelectedPark?.price" :distance="currentSelectedPark?.distance" :display="currentSelectedPark?.display || 'navigator_park'" show />
    </div>
</template>
