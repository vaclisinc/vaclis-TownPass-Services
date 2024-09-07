<template>
  <div ref="mapContainer" class="map-container"></div>
</template>

<script>
import mapboxgl from 'mapbox-gl';
import axios from 'axios';

mapboxgl.accessToken =
  'pk.eyJ1IjoiZjc0MTE0NzYwIiwiYSI6ImNtMHJyenV3eTBjOGQyaXNicDFsbXU2YzIifQ.fhoguJDc6TfWAGwn471Hog';
export default {
  mounted() {
    const map = (this.map = new mapboxgl.Map({
      container: this.$refs.mapContainer,
      style: 'mapbox://styles/mapbox/light-v11', // Replace with your preferred map style
      center: [121.5624999, 25.0325917],
      zoom: 15
    }));

    map.on('moveend', (e) => {
      console.log('move end:', e.type);
      getParkingData();
    });

    let updating = 0;
    const markers = [];

    getParkingData();

    function getParkingData() {
      if (updating > 0) return;
      updating = 2;
      markers.forEach(i=>i.remove());

      let center = map.getCenter();
      let lon = center.lng, lat = center.lat;
      axios
        .get(
          // `https://127.0.0.1:25569?lon=${lon}&lat=${lat}`
          `https://api.wavjaby.nckuctf.org:25569?lon=${lon}&lat=${lat}`
        )
        .then((i) => i.data)
        .then((i) => {
          console.log(i);
          addPoints(
            'parkingLot',
            i.parkingLot.map((i) => ({
              type: 'Feature',
              properties: {
                color: '#ff0000'
              },
              geometry: {
                type: 'Point',
                coordinates: [i.lon, i.lat]
              }
            })),
            1,
            3
          );
          addPolygon(
            'parkingGrid',
            i.parkingGrid.map((i) => ({
              type: 'Feature',
              properties: {
                color: i.available ? '#81b29a' : '#e07a5f'
              },
              geometry: {
                type: 'Polygon',
                coordinates: [i.wkt]
              }
            })),
            1
          );
          // add markers to map
          i.parkingGrid.forEach(
            (i) => i.available && makeMarker('1', '#26a7ac', 25, [i.lon, i.lat])
          );
          i.parkingLot
            .sort((a, b) => a.carRemainderNum - b.carRemainderNum)
            .forEach((i) =>
              makeMarker(
                i.carRemainderNum,
                i.carRemainderNum > 0 ? '#693' : '#f20',
                i.carRemainderNum === 0 ? 20 : Math.min(50 * (i.carRemainderNum / 1000) + 25, 60),
                [i.lon, i.lat]
              )
            );
          updating--;
        });
      axios
        .get(`https://api.wavjaby.nckuctf.org:25569/get_line?lon=${lon}&lat=${lat}`)
        .then((i) => i.data)
        .then((i) => {
          console.log(i);
          let featuresPath = i.map((i) => ({
            type: 'Feature',
            properties: {
              color: '#ffff00'
            },
            geometry: {
              type: 'LineString',
              coordinates: i.coordinates
            }
          }));
          addSourceAndLayer('yellowLine', featuresPath, {
            type: 'line',
            layout: {
              'line-join': 'round',
              'line-cap': 'round'
            },
            paint: {
              'line-color': ['get', 'color'],
              'line-width': 4
            }
          });
          updating--;
        });
    }

    function makeMarker(text, color, size, cord) {
      let el = document.createElement('div');
      el.className = 'marker';
      // Set text and size
      let sp = document.createElement('span');
      sp.innerHTML = '<b>' + text + '</b>';
      sp.style.background = color;
      sp.style.width = sp.style.height = size + 'px';
      el.append(sp);

      // make a marker for each feature and add it to the map
      markers.push(new mapboxgl.Marker(el).setLngLat(cord).addTo(map));
    }

    function addSourceAndLayer(id, features, layerConfig) {
      const srcId = id + '_src';
      if (map.getLayer(id)) map.removeLayer(id);
      if (map.getSource(srcId)) map.removeSource(srcId);
      map.addSource(srcId, {
        type: 'geojson',
        data: { type: 'FeatureCollection', features: features }
      });
      layerConfig.id = id;
      layerConfig.source = srcId;
      map.addLayer(layerConfig);
    }

    function addPoints(id, features, opacity, radius) {
      addSourceAndLayer(id, features, {
        type: 'circle',
        layout: {},
        paint: {
          'circle-opacity': opacity,
          'circle-color': ['get', 'color'],
          'circle-radius': radius
        }
      });
    }

    function addPolygon(id, features, opacity) {
      addSourceAndLayer(id, features, {
        type: 'fill',
        layout: {},
        paint: {
          'fill-opacity': opacity,
          'fill-color': ['get', 'color']
        }
      });
    }
  },
  unmounted() {
    this.map.remove();
    this.map = null;
  }
};
</script>

<style>
.map-container {
  flex: 1;
}

.marker {
  width: 0;
  height: 0;
  font-size: 16px;
}

.marker > span {
  display: flex;
  justify-content: center;
  align-items: center;
  box-sizing: border-box;
  color: #fff;
  border: solid 1px;
  border-radius: 0 70% 70%;
  box-shadow: 0 0 2px #000;
  cursor: pointer;
  transform-origin: 0 0;
  transform: rotateZ(-135deg);
}

.marker b {
  transform: rotateZ(135deg);
}
</style>
