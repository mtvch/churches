<template>
  <el-row :gutter="20">
    <el-col :offset="3" :span="2">
      <el-card class="box-card">
        <template #header>
          <div class="card-header">
            <span
              class="iconify-inline"
              data-icon="wi:cloud"
              style="color: #409eff"
            ></span>
          </div>
        </template>
        <div
          v-if="weather"
          class="vld-parent weather-data"
          ref="weatherContainer"
        >
          <div>
            {{ weather.temp }}
            <span
              class="iconify-inline"
              data-icon="wi:celsius"
              style="color: #409eff"
            ></span>
          </div>
          <div>
            {{ weather.humidity }}
            <span
              class="iconify-inline"
              data-icon="wi:humidity"
              style="color: #409eff"
            ></span>
          </div>
          <div>
            {{ weather.pressure
            }}<span
              class="iconify"
              data-icon="wi:barometer"
              style="color: #409eff"
            ></span>
          </div>
        </div>
        <el-empty v-else></el-empty>
      </el-card>
    </el-col>
    <el-col :span="8">
      <el-table
        ref="churchesTable"
        :data="churches"
        highlight-current-row
        style="width: 100%"
        @current-change="handleCurrentChange"
      >
        <el-table-column property="name" label="Name" />
        <el-table-column property="dist" label="Distance, m" />
      </el-table>
    </el-col>
    <el-col :span="8">
      <div ref="infoContainer" v-if="churchInfo">
        <el-scrollbar max-height="400px">
          <p>
            <el-icon color="#409EFF">
              <location-information />
            </el-icon>
            {{ buildAddrStr() }}
          </p>
          <!-- <el-avatar
            shape="square"
            :size="100"
            :src="'https://commons.wikimedia.org/wiki/File:Paris_7e_30_Rue_de_Lille_21.JPG'"
          ></el-avatar> -->
          <p>
            {{churchInfo.wikipedia_extracts.text}}
          </p>
        </el-scrollbar>
      </div>
    </el-col>
  </el-row>
</template>

<script>
import axios from "axios";
import { ElNotification } from "element-plus";
import { LocationInformation } from "@element-plus/icons";

export default {
  components: {
    LocationInformation,
  },
  data() {
    return {
      churches: [],
      weather: null,
      churchInfo: null,
    };
  },
  props: {
    placeGeo: {
      type: Object,
      required: true,
    },
  },
  methods: {
    buildAddrStr() {
      var str = this.churchInfo.address.city;
      if (this.churchInfo.address.road) {
        str = str.concat(", ", this.churchInfo.address.road);
      }
      if (this.churchInfo.address.house_number) {
        str = str.concat(", ", this.churchInfo.address.house_number);
      }
      return str;
    },
    async loadChurches(lon, lat) {
      try {
        const res = await axios.get(
          "http://api.opentripmap.com/0.1/en/places/radius",
          {
            params: {
              radius: 2000,
              kinds: "churches",
              lon,
              lat,
              apikey:
                "5ae2e3f221c38a28845f05b681ba291f27b88ddcb2fa871b279bfcb3", // should be hidden on server
              limit: 10,
              format: "json",
            },
          }
        );
        this.churches = res.data.map((c) => {
          c.dist = Math.round(c.dist);
          return c;
        });
      } catch (e) {
        ElNotification({
          title: "Loading error",
          message: e.message,
          position: "bottom-right",
        });
      }
    },
    async loadWeather(lon, lat) {
      let loader = this.$loading.show({
        container: this.$refs.weatherContainer,
      });
      try {
        const res = await axios.get(
          "http://api.openweathermap.org/data/2.5/weather",
          {
            params: {
              lon,
              lat,
              units: "metric",
              appid: "f9b571e68e7b4cf79e5ba32a6b2ed9a9",
            },
          }
        );
        this.weather = res.data.main;
        this.weather.temp = this.weather.temp.toFixed(1);
      } catch (e) {
        ElNotification({
          title: "Loading error",
          message: e.message,
          position: "bottom-right",
        });
      } finally {
        loader.hide();
      }
    },
    async loadChurchInfo(xid) {
      let loader = this.$loading.show({
        container: this.$refs.infoContainer,
      });
      try {
        const res = await axios.get(
          `http://api.opentripmap.com/0.1/en/places/xid/${xid}`,
          {
            params: {
              apikey:
                "5ae2e3f221c38a28845f05b681ba291f27b88ddcb2fa871b279bfcb3",
            },
          }
        );
        console.log(res);
        return res.data;
      } catch (e) {
        ElNotification({
          title: "Loading error",
          message: e.message,
          position: "bottom-right",
        });
      } finally {
        loader.hide();
      }
    },
    async handleCurrentChange(church) {
      this.churchInfo = await this.loadChurchInfo(church.xid);
    },
  },
  watch: {
    placeGeo(newValue) {
      if (newValue) {
        this.loadChurches(newValue.point.lng, newValue.point.lat);
        this.loadWeather(newValue.point.lng, newValue.point.lat);
      }
    },
  },
  mounted() {
    if (this.placeGeo) {
      this.loadChurches(this.placeGeo.point.lng, this.placeGeo.point.lat);
      this.loadWeather(this.placeGeo.point.lng, this.placeGeo.point.lat);
    }
  },
};
</script>

<style lang="scss">
.weather-data {
  text-align: right;
}
</style>