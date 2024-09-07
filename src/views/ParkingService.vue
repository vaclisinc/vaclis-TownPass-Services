<script setup lang="ts">
import Map from '@/components/organisms/Map.vue';
import 'mapbox-gl/dist/mapbox-gl.css';
import NavigatorCard from '@/components/organisms/NavigatorCard.vue';
import { useHandleConnectionData } from '../composables/useHandleConnectionData';
import { useConnectionMessage } from '../composables/useConnectionMessage';
import { onMounted } from 'vue';

onMounted(() => {
  console.log('Get location');
  useConnectionMessage('location', null);

  useHandleConnectionData((i) => {
    console.log(i);
  });
});

</script>
<template>
  <div class="h-screen flex flex-col overflow-hidden position-relative">
    <div class="flex flex-1">
      <Map />
    </div>
    <NavigatorCard
      parkName="foo"
      remainingSpace="10"
      price="40圓/小時"
      distance="10"
      display="navigator_yellow_line"
      show
    />
  </div>
</template>