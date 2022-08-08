<template>
  <div class="container">
    <div class="control">
      <el-button @click="getLocation()">Locate Where am I</el-button>
      <div>
        <el-input
          @change="searchLocation"
          type="text"
          v-model="search"
          placeholder="Search place.."
        />
      </div>
    </div>
    <div class="table">
      <div class="controlPanel">
        <el-button type="warning" @click="deleteSelectRecord">DELETE</el-button>
        <div v-if="timeZone && currentTime">
          Current Time(GMT: {{ timeZone }}):{{ currentTime }}
        </div>
      </div>
      <el-table :data="pagedResult" @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="55"> </el-table-column>
        <el-table-column prop="display_name" label="Name"></el-table-column>
        <el-table-column prop="lat" label="Lat"></el-table-column>
        <el-table-column prop="lon" label="Long"></el-table-column>
      </el-table>
    </div>
    <el-pagination
      layout="prev, pager, next"
      :total="searchResult.length"
      :size="pageSize"
      :current-page.sync="currentPage"
    >
    </el-pagination>
  </div>
</template>
<script>
import EventBus from "../tools/eventbus.js";
import axios from "axios";

export default {
  name: "controlPanel",
  data: function () {
    return {
      currentLocation: null,
      search: "",
      searchResult: [],
      pagedResult: [],
      currentPage: 1,
      pageSize: 10,
      selectedRecord: [],
      timeZone: "",
      currentTime: "",
    };
  },
  components: {},
  methods: {
    deleteSelectRecord() {
      var temp = [];
      console.log(this.selectedRecord);
      for (var i = 0; i < this.searchResult.length; i++) {
        if (!this.selectedRecord.includes(this.searchResult[i].place_id)) {
          temp.push(this.searchResult[i]);
        }
      }
      this.searchResult = temp;
      this.refreshPage();
    },
    getLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(this.showPosition);
      }
    },
    showPosition(position) {
      EventBus.$emit("currentLocation", position);
    },
    searchLocation() {
      axios
        .get(
          "https://nominatim.openstreetmap.org/search/" +
            this.search +
            "?format=json&limit=50"
        )
        .then((response) => {
          this.searchResult = response.data;
          this.currentPage = 1;
          this.refreshPage();
          this.getTimeZone();
        });
    },
    getTimeZone() {
      if (this.searchResult[0]) {
        var lat = this.searchResult[0].lat;
        var long = this.searchResult[0].lon;
        axios
          .get(
            "http://api.geonames.org/timezoneJSON?formatted=true&lat=" +
              lat +
              "&lng=" +
              long +
              "&username=ho_tom2333"
          )
          .then((resp) => {
            console.log(resp.data);
            this.timeZone =
              resp.data.gmtOffset > 0
                ? "+" + resp.data.gmtOffset
                : resp.data.gmtOffset;
            this.currentTime = resp.data.time;
          });
      }
    },
    handleSelectionChange(val) {
      this.selectedRecord = [];
      for (var i = 0; i < val.length; i++) {
        this.selectedRecord.push(val[i].place_id);
      }
    },
    refreshPage() {
      console.log("-------");
      console.log(this.searchResult.slice(0,10))
      this.pagedResult = this.searchResult.slice(
        (this.currentPage - 1) * this.pageSize + 1,
        (this.currentPage - 1) * this.pageSize + 1 +this.pageSize
      );
      EventBus.$emit("addMarkers", this.pagedResult);
    },
  },
  mounted() {},
  watch: {
    currentPage() {
      this.refreshPage();
    },
    searchResult() {
      this.refreshPage();
    },
  },
};
</script>

<style scoped>
.container {
  height: calc(100% - 40px);
  width: 100%;
}
.control {
  display: flex;
  justify-content: space-around;
  margin-top: 20px;
}
.controlPanel {
  display: flex;
  justify-content: space-evenly;
  align-items: center;
  margin-top: 20px;
}
</style>