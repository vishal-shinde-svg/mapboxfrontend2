<template>
  <!-- container contain to map and list -->
  <div>
    <div id="fromDiv" class="flex justify-center items-center">
      <div
        id="innerFrom"
        class="block p-6 rounded-lg shadow-lg bg-white max-w-sm border-2 border-red-600 ml-96 mt-40"
      >
        <div id="innerFrom" class="border-2">
          <div class="form-group mb-6">
            <label
              for="exampleInputEmail1"
              class="form-label inline-block mb-2 text-gray-700"
              >Name</label
            >
            <input
              type="text"
              class="form-control block w-full px-3 py-1.5 text-base font-normal text-gray-700 bg-white bg-clip-padding border border-solid border-gray-300 rounded transition ease-in-out m-0 focus:text-gray-700 focus:bg-white focus:border-blue-600 focus:outline-none"
              id="exampleInputEmail1"
              aria-describedby="emailHelp"
              placeholder="Enter The Name"
              v-model="data.name"
            />
          </div>
          <div class="form-group mb-6">
            <label
              for="exampleInputPassword1"
              class="form-label inline-block mb-2 text-gray-700"
              >Set Color</label
            >
            <input
              type="color"
              class="form-control block w-full px-3 py-1.5 text-base font-normal text-gray-700 bg-white bg-clip-padding border border-solid border-gray-300 rounded transition ease-in-out m-0 focus:text-gray-700 focus:bg-white focus:border-blue-600 focus:outline-none"
              id="exampleInputPassword1"
              v-model="data.colorName"
              placeholder="Password"
            />
          </div>

          <button
            type="submit"
            class="px-6 py-2.5 bg-blue-600 text-white font-medium text-xs leading-tight uppercase rounded shadow-md hover:bg-blue-700 hover:shadow-lg focus:bg-blue-700 focus:shadow-lg focus:outline-none focus:ring-0 active:bg-blue-800 active:shadow-lg transition duration-150 ease-in-out"
            @click="savename"
          >
            Submit
          </button>
        </div>
      </div>
    </div>
    <!-- list define div -->
    <div
      id="list"
      class="w-1/5 h-screen float-left bg-gradient-to-r from-indigo-500 via-purple-500 to-pink-500"
    >
      <h3>Layers</h3>
      <table class="">
        <tr>
          <th>name</th>
          <th>Hide/Show</th>
        </tr>
        <ul>
          <tr>
            <li v-for="(item, index) in data.backEndData">
              <td>
                <label for="checkBox"> {{ item.Name }}</label>
              </td>
              <td>
                <input
                  type="checkbox"
                  id="checkBox"
                  @change="toggle($event, index)"
                />
              </td>
              <td>
                <input
                  id="slider"
                  type="range"
                  min="0"
                  max="100"
                  step="0"
                  value="100"
                />
              </td>
            </li>
          </tr>
        </ul>
      </table>
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
import MapboxGeocoder from "@mapbox/mapbox-gl-geocoder";
import "@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css";

let ID;
let Geom;

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
  backEndData: [],
  data1: [],
  name: "",
  
  colorName: "",

  drawTool: null,
});

getAllData();
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
  data.drawTool = new MapboxDraw({
    displayControlsDefault: false,
    controls: {
      line_string: true,
      polygon: true,
      point: true,
      trash: true,
    },
  });

  const geocoder = new MapboxGeocoder({
    accessToken: mapboxgl.accessToken,
    flyTo: {
      bearing: 0,
      // Control the flight curve, making it move slowly and
      // zoom out almost completely before starting to pan.
      speed: 1, // Make the flying slow.
      curve: 1, // Change the speed at which it zooms out.
      // This can be any easing function: it takes a number between
      // 0 and 1 and returns another number between 0 and 1.
      easing: function (t) {
        return t;
      },
    },
    mapboxgl: mapboxgl,
  });

  // Add the geocoder to the map/
  data.map.addControl(geocoder);

  data.map.addControl(data.drawTool, "top-left");
  data.map.on("draw.create", updateArea);
  // data.map.on("draw.delete", updateArea);

  // data.map.on("draw.update", updateArea);

  async function updateArea(e) {
    console.log(e.features[0].id);
    document.getElementById("fromDiv").style.display = "block";
    console.log("end");

    data.drawTool.deleteAll().getAll();

    // data.map.on("draw.delete", () => {
    //   setTimeout(() => {
    //     draw.deleteAll();
    //   }, 0);
    // });

    // create object
    // let name = prompt("Enter the name ");

    data.data1 = e.features[0];
    ID = e.features[0].id;
    Geom = e.features[0].geometry;

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

  console.log(data.backEndData);
}
//////////////////////////////////////// Togale function  this function is used to show and hide layears

function toggle(e, index) {
  console.log(data.backEndData[index]);
  console.log(data.backEndData[index].Id);
  console.log(e.target.checked);
  if (e.target.checked == true) {
    console.log(data.layears[index]);
    data.map.addSource(data.backEndData[index].Id, {
      type: "geojson",
      data: data.backEndData[index].Geom,
    });
    data.map.addLayer({
      id: data.backEndData[index].Id,
      source: data.backEndData[index].Id,
      type: "fill",
      layout: {},
      paint: {
        "fill-color": data.backEndData[index].Property,
        "fill-opacity": 0.5,
      },
    });

    // let flylocation = data.layears[index].data.coordinates[0];
    console.log(data.layears[index].data.geometry.coordinates[0][0]);
    console.log(data.layears[index].data.geometry.type);
    let type = data.layears[index].data.geometry.type;
    if (type == "Point") {
      let fly = data.layears[index].data.geometry.coordinates;
      randomFly(fly, type);
    } else if (type == "Polygon") {
      let fly = data.layears[index].data.geometry.coordinates[0][0];
      randomFly(fly, type);
    } else {
      let fly = data.layears[index].data.geometry.coordinates[0];
      randomFly(fly, type);
    }
    // randomFly(flylocation);
  } else {
    data.map.removeLayer(data.backEndData[index].Id);
    data.map.removeSource(data.backEndData[index].Id);
  }
  // console.log(data.layears[index].geometry.coordinates);
}
async function savename() {
  let name = data.name;
  let colorName = data.colorName;
  let data1: any = { data: data.data1, name: name, colorName: colorName };
  data.layears.push(data1);
  document.getElementById("fromDiv").style.display = "none";

  let formData = {
    Name: name,
    Property: colorName,
    Geom: Geom,
  };
  await $fetch("http://localhost:3001/mapg", {
    method: "POST",
    body: formData,
  })
    .then((res) => console.log("Data has been save successfully"))
    .catch((err) => alert(err));
  data.name = "";
  getAllData();
}
function randomFly(a, type) {
  data.map.flyTo({
    center: a,
    zoom: 9,
    speed: 1,
    curve: 1,
    easing(t) {
      return t;
    },
  });
  // // random name
  //   data.map.flyTo({
  //     center: [(Math.random() - 0.5) * 360, (Math.random() - 0.5) * 100],
  //     essential: true, // this animation is considered essential with respect to prefers-reduced-motion
  //   });
}
async function getAllData() {
  const res: any = await $fetch("http://localhost:3001/mapg");
  data.backEndData = res;
}
</script>
<style scoped>
#fromDiv {
  height: 100%;
  z-index: 5;
  background-color: rgb(49, 50, 50);
  opacity: 0.9;
  width: 100%;
  position: absolute;
  top: auto;
  left: auto;
  border: 2px solid black;
  border-radius: 15px 5px 15px 5px;
  display: none;
}
</style>
