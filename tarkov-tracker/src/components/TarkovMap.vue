<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <template v-for="floor, floorIndex in props.map.svg.floors" :key="floorIndex">
          <v-btn variant="tonal" @click="setFloor(floor)">{{ floor }}</v-btn>
        </template>
      </v-col>
      <v-col cols="12">
        <div :id="randomMapId"></div>
      </v-col>
    </v-row>
  </v-container>
</template>
<script setup>
import { defineProps, computed, ref, onMounted } from "vue";
import { useUserStore } from "@/stores/user.js";
import { v4 as uuidv4 } from 'uuid';
import * as d3 from "d3";
const randomMapId = ref(uuidv4());
const emit = defineEmits(['gpsclick'])
const props = defineProps({
  map: {
    type: Object,
    required: true,
  },
});

// selectedFloor is a ref which defaults to the last item in the floors array
const selectedFloor = ref(props.map.svg.floors[props.map.svg.floors.length - 1])

const setFloor = (floor) => {
  selectedFloor.value = floor
  draw()
}

const draw = async () => {
  const svg = await d3.svg(`https://tarkovtracker.github.io/tarkovdata/maps/${props.map.svg.file}`)
  d3.select(document.getElementById(randomMapId.value)).selectAll('svg').remove()
  d3.select(document.getElementById(randomMapId.value)).node().appendChild(svg.documentElement)
  // Calculate the index of the selected floor
  const selectedFloorIndex = props.map.svg.floors.indexOf(selectedFloor.value)
  props.map.svg.floors.forEach((floor, index) => {
    if (index > selectedFloorIndex) {
      d3.select(document.getElementById(randomMapId.value)).select('svg').select(`#${floor}`).style('opacity', 0.02)
    }
  }, this)
}
onMounted(() => {
  console.log(`the component is now mounted.`)
  draw()

  // Event block for creating gps data
  document.getElementById(randomMapId.value).addEventListener("click", function (event) {
    let e = document.getElementById(randomMapId.value);
    let dim = e.getBoundingClientRect();
    let gps = {
      map: props.map.id,
      leftPercent: parseFloat(((event.clientX - dim.left) / e.clientWidth * 100).toFixed(2)),
      topPercent: parseFloat(((event.clientY - dim.top) / e.clientHeight * 100).toFixed(2)),
      floor: selectedFloor.value
    }
    emit('gpsclick', gps)
  }.bind(this));
})



const userStore = useUserStore();

</script>
<style lang="scss" scoped>

</style>