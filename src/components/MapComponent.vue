<template>
  <div id="map" style="height: 500px;"></div>
</template>

<script>
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import * as turf from '@turf/turf';

export default {
  data() {
    return {
      map: null,
      pois: [],
    };
  },
  mounted() {
    // 지도 초기화
    this.map = L.map('map').setView([37.5708141, 126.9789911], 13);

    // 이 부분을 vworld로 바꾸는 작업
    // openlayers 이용할 방법도 고려 필요 (vworld가 openlayers로 만들어져있음)
    // OpenStreetMap 타일 레이어 추가
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '© OpenStreetMap contributors'
    }).addTo(this.map);

    // 사용자의 현재 위치를 가져온다
    navigator.geolocation.getCurrentPosition(this.onLocationFound, this.onLocationError);

    // 지도 클릭 이벤트 처리
    this.map.on('click', this.onMapClick);
  },
  methods: {
    onLocationFound(e) {
      const radius = 100; // 반경 100미터
      const center = [e.coords.longitude, e.coords.latitude];
      // const center = [e.coords.latitude, e.coords.longitude];

      // 사용자 위치에 마커를 추가한다
      L.marker([e.coords.latitude, e.coords.longitude]).addTo(this.map);
          // .bindPopup("당신의 위치입니다.")
          // .openPopup();

      // 랜덤 POI 생성
      this.createRandomPois(center, radius);

      // 지도를 사용자의 위치로 이동한다
      this.map.setView([e.coords.latitude, e.coords.longitude], 13);
    },
    onLocationError(e) {
      alert(e.message);
    },
    createRandomPois(center, radius) {
      // 반경 내에서 랜덤한 위치에 POI 생성
      for (let i = 0; i < 10; i++) {
        const randomPoint = turf.randomPoint(1, { bbox: [center[0] - 0.01, center[1] - 0.01, center[0] + 0.01, center[1] + 0.01] });
        const poi = randomPoint.features[0];
        this.pois.push(poi);

        // 생성된 POI에 마커를 추가한다
        L.marker([poi.geometry.coordinates[1], poi.geometry.coordinates[0]])
            .addTo(this.map)
            .bindPopup(`POI ${i + 1}`);
      }
    },
    // onMapClick(e) {
    //   // 클릭한 지점의 POI 목록을 콘솔에 출력한다
    //   console.log('클릭한 지점 근처의 POI:', this.pois);
    // }
    onMapClick(e) {
      // 이전에 만든 원이 있다면 지도에서 제거
      if (this.radiusCircle) {
        this.radiusCircle.remove();
      }

      // 클릭한 지점의 좌표
      const clickedPoint = [e.latlng.lng, e.latlng.lat];

      // 반경을 설정 (예: 500미터)
      const radius = 0.5; // 킬로미터 단위

      // Turf.js를 사용하여 원 생성
      const circle = turf.circle(clickedPoint, radius, { steps: 80, units: 'kilometers' });

      // Leaflet 지도에 원을 추가
      this.radiusCircle = L.geoJSON(circle).addTo(this.map);
    }
  }
};
</script>

<style>
/* Leaflet CSS */
#map { width: 100%; height: 100%; }
.leaflet-container { background-color: #fff; }
</style>
