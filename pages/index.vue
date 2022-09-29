<template>
  <!-- container contain to map and list -->
  <div>
    <div id="fromDiv">
      <label for="name1"
        >Enter tha name<input
          type="text"
          id="name1"
          v-model="data.name"
          autocomplete="off"
      /></label>
      <button @click="savename">add name</button>
    </div>
    <!-- list define div -->
    <div id="list" class="w-1/5 h-full float-left bottom-1 border-red-600">
      <h3>Layers</h3>
      <ul>
        <li v-for="(item, index) in data.layears">
          <input type="checkbox" @change="toggle($event, index)" />
          {{ item.name }}
        </li>
      </ul>
    </div>

    <!-- map box div -->
    <!-- <div id="mapbox" class="h-full w-5/5"> -->
    <div class="w-4/5 h-screen border-2 border-slate-900 float-left">
      <v-map class="w-5/5 h-full" :options="data.options" @loaded="onMapLoaded">
      </v-map>
    </div>
  </div>
  <!-- </div> -->
</template>
<script setup lang="ts">
import "mapbox-gl/dist/mapbox-gl.css";
import "v-mapbox/dist/v-mapbox.css";
import "@mapbox/mapbox-gl-draw/dist/mapbox-gl-draw.css";
import Mapbox from "mapbox-gl";
import { VMap } from "v-mapbox";
import mapboxgl from "mapbox-gl";
import MapboxDraw from "@mapbox/mapbox-gl-draw";
import { anyTypeAnnotation } from "@babel/types";

const data = reactive({
  options: {
    accessToken:
      "pk.eyJ1IjoibWF5dXJ3YWtpa2FyIiwiYSI6ImNsNmdjdGxwbjBiNGMzY282bWh0dng2c2kifQ.y-m4-zQKOeOOnDG5I1u6ng",
    style: "mapbox://styles/mapbox/outdoors-v11",
    center: [-284.8618412017822, 20.253581321936355],
    zoom: 11,
    maxZoom: 22,
    crossSourceCollisions: false,
    failIfMajorPerformanceCaveat: false,
    attributionControl: false,
    preserveDrawingBuffer: true,
    hash: false,
    minPitch: 0,
    maxPitch: 60,
    container: "map",
  } as mapboxgl.MapboxOptions,
  map: {} as mapboxgl.Map,
  layears: [],
  data1: [],
  name: "",
});
async function onMapLoaded(tempMap: mapboxgl.Map) {
  data.map = tempMap;
  mapboxgl.accessToken =
    "pk.eyJ1IjoibWF5dXJ3YWtpa2FyIiwiYSI6ImNsNmdjdGxwbjBiNGMzY282bWh0dng2c2kifQ.y-m4-zQKOeOOnDG5I1u6ng";
  // map.addControl(
  //   new MapboxGeocoder({
  //     accessToken: mapboxgl.accessToken,
  //     mapboxgl: mapboxgl,
  //   })
  // );
  data.map.addControl(new mapboxgl.NavigationControl());
  var Draw = new MapboxDraw();
  data.map.addControl(Draw, "top-left");
  data.map.on("draw.create", updateArea);
  // data.map.on("draw.delete", updateArea);
  // data.map.on("draw.update", updateArea);

  function updateArea(e) {
    console.log(e);
    document.getElementById("fromDiv").style.display = "block";

    // create object
    // let name = prompt("Enter the name ");

    data.data1 = e.features[0];

    // data.map.on('draw.delete', () => {
    // setTimeout(() => {
    //     Draw.deleteAll()
    // }, 0)

    // data.layears.push(data1);

    /// addd source

    // data.map.addSource(e.features[0].id, {
    //   type: "geojson",
    //   // data: data.layears[data.layears.length - 1],
    //   data: data1,
    // });

    // console.log(coordinates);
    // console.log(type1);
  }
}
//////////////////////////////////////// Togale function  this function is used to show and hide layears

function toggle(e, index) {
  console.log(data.layears[index]);

  console.log(e.target.checked);
  if (e.target.checked == true) {
    console.log(data.layears[index]);
    data.map.addSource(data.layears[index].data.id, {
      type: "geojson",
      data: data.layears[index].data,
    });
    data.map.addLayer({
      id: data.layears[index].data.id,
      source: data.layears[index].data.id,
      type: "fill",
      layout: {},
      paint: { "fill-color": "pink", "fill-opacity": 0.5 },
    });
  } else {
    data.map.removeLayer(data.layears[index].data.id);
    data.map.removeSource(data.layears[index].data.id);
  }
  // console.log(data.layears[index].geometry.coordinates);
}
function savename() {
  let name = data.name;
  let data1: any = { data: data.data1, name: name };
  data.layears.push(data1);
  document.getElementById("fromDiv").style.display = "none";
}
</script>
<style scoped>
#fromDiv {
  height: 100px;
  z-index: 1;
  background-color: aqua;
  width: 20%;
  position: absolute;
  top: 300px;
  left: 500px;
  border: 2px solid black;
  border-radius: 15px 5px 15px 5px;
  display: none;
}
#fromDiv button {
  margin: 4px;
  padding: 3px;
  border: 2px solid black;
  margin-left: 150px;
}
#fromDiv input {
  margin-left: 10px;
}
</style>
