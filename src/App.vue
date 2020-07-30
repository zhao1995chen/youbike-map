<template>
  <div id="app">
    <div class="row no-gutters">
      <!-- 選擇地區 -->
      <div class="toolbox col-sm-3 p-2 bg-white">
        <div class="form-group d-flex">
          <label for="city" class="col-form-label mr-2 text-right">縣市</label>
          <div class="flex-fill">
            <select id="city" class="form-control" v-model="select.city">
              <!-- 製作下拉選單 -->
              <option v-bind:value="city.name" v-bind:key="city.name" v-for="city in cityName">
                {{ city.name }}
              </option>
            </select>
          </div>
        </div>
        <div class="form-group d-flex">
          <label for="dist" class="col-form-label mr-2 text-right">地區</label>
          <div class="flex-fill">
            <select id="dist" class="form-control" v-model="select.district">
              <!-- 製作下拉選單 -->
              <option :value="district.name" :key="district.name"
                v-for="district in cityName.find((city) => city.name === select.city).districts">
                {{ district.name }}
              </option>
            </select>
          </div>
        </div>
      </div>

      <!-- 顯示地圖和 UBike 站點 -->
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
import L from 'leaflet';
import cityName from './assets/cityName.json';

export default {
  name: 'App',
  data: () => ({
    cityName,
    select: {
      city: '台北市',
      district: '中正區',
    },
    ubikes: [],
    OSMap: {},
  }),
  computed: {
    youbikes() {
      return this.ubikes.filter((bike) => bike.sarea === this.select.district);
    },
  },
  watch: { // 監聽事件發生並觸發之後事件
    youbikes() {
      this.updateMap();
    },
  },
  methods: {
    updateMap() {
      // remove markers
      this.OSMap.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          this.OSMap.removeLayer(layer);
        }
      });

      // add markers
      this.youbikes.forEach((bike) => {
        L.marker([bike.lat, bike.lng])
          .bindPopup(`<p><strong style="font-size: 20px;">${bike.sna}</strong></p>
          <strong style="font-size: 16px; color: #d45345;">可租借車輛剩餘：${bike.sbi} 台</strong><br>
          可停空位剩餘: ${bike.bemp}<br>
          <small>最後更新時間: ${bike.mday}</small>
        `).addTo(this.OSMap);
      });

      // move new center
      this.cityName[0].districts.find((district) => {
        if (district.name === this.select.district) {
          // this.OSMap.panTo(new L.LatLng(district.latitude, district.longitude)); // 直接移過去
          this.OSMap.flyTo(new L.LatLng(district.latitude, district.longitude)); // 飛過去的感覺
        }
        return district.name === this.select.district;
      });
    },
  },
  created() {
    const api = 'https://tcgbusfs.blob.core.windows.net/blobyoubike/YouBikeTP.json';
    this.$http.get(api).then((response) => {
      // eslint-disable-next-line
      console.log(response.data);
      this.ubikes = Object.keys(response.data.retVal).map((key) => response.data.retVal[key]);
    });
  },
  mounted() {
    this.OSMap = L.map('map', {
      center: [25.041956, 121.508791],
      zoom: 18,
    });

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 18,
    }).addTo(this.OSMap);
  },
};
</script>

<style lang="scss">
@import 'bootstrap/scss/bootstrap';

#map {
  height: 100vh;
  position: relative;
}
</style>
