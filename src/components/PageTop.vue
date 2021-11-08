<template>
  <el-row id="top">
    <el-col :span="6">
      <h1>Chrchs</h1>
    </el-col>
    <el-col :span="6" :offset="3">
      <el-autocomplete
        class="search-input"
        v-model="searchInput"
        :trigger-on-focus="false"
        :prefix-icon="Search"
        :fetch-suggestions="queryPlaces"
        @select="handleSelect"
      />
    </el-col>
  </el-row>
</template>

<script>
import { Search } from "@element-plus/icons";
import axios from "axios";
import { ElNotification } from 'element-plus'

function buildPlaceString(place) {
      var str = place.name;
      if (place.countrycode) {
        str = str.concat(', ', place.countrycode);
      }
      if (place.city) {
        str = str.concat(', ', place.city);
      }
      return str;
    }

export default {
  data() {
    return {
      Search,
      searchInput: "",
      places: {},
      timeout: null,
    };
  },
  methods: {
    async loadPlaces(queryString) {
      try {
        const res = await axios.get("https://graphhopper.com/api/1/geocode", {
          params: {
            q: queryString,
            locale: "en",
            key: "83f5f2e3-017e-4202-9cf6-01727448deb7", // should be hidden on server
            limit: 5,
          },
        });
        this.places = res.data.hits.reduce(
          (acc, p) => {
            const str = buildPlaceString(p);
            acc[str] = p;
            return acc;
          },
          {}
        );
      } catch (e) {
        ElNotification({
          title: "Loading error",
          message: e.message,
          position: 'bottom-right',
        });
      }
    },
    queryPlaces(queryString, cb) {
      clearTimeout(this.timeout);
      setTimeout(async () => {
        await this.loadPlaces(queryString);
        const places = [];
        for (var key in this.places) {
          places.push({ value: key });
        }
        cb(places);
      }, 1000);
    },
    handleSelect(placeEvent) {
      const res = this.places[placeEvent.value];
      if (res) {
        this.$emit("select", res);
      }
    },
  },
};
</script>

<style lang="scss">
#top {
  margin-top: 5px;
}

h1 {
  text-transform: uppercase;
  font-size: 20px;
  font-family: "Inter", sans-serif;
  color: #409eff;
  font-weight: 700;
  margin-top: 10px;
}
</style>