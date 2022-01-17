<script>
import { ref, computed, onMounted, reactive } from "vue";
import data from "../assets/store.json";
export default {
  props: {
    msg: String,
    resize: Boolean,
  },
  setup() {
    const rosterRef = ref(null);
    const state = reactive({
      ...data,
      dragging: false,
      draggingX: 0,
    });
    onMounted(() => {
      state.rosterWidth = rosterRef.value.offsetWidth;
      console.log(state.rosterWidth);
    });
    const areaRow = (areaId) => {
      return state.shifts.filter((x) => x.area == areaId).length;
    };
    const areaHeight = (r) => {
      return state.rowHeight * (areaRow(r) + state.leaveSpace) + "px";
    };
    const shiftTop = (i) => {
      return state.rowHeight * i + "px";
    };
    const areaShiftSort = (areaId) => {
      return state.shifts
        .filter((x) => x.area == areaId)
        .sort((a, b) => {
          return a.x - b.x;
        });
    };
    const empDragStart = (event, emp) => {
      // console.log("empDragStart", emp);
      event.dataTransfer.dropEffect = "copy";
      event.dataTransfer.setData("emp", emp.name);
    };
    const shiftDragStart = (event, shift) => {
      // console.log("shiftDragStart", shift);
      event.dataTransfer.dropEffect = "move";
      event.dataTransfer.setData("shift", shift.id);
    };
    const onDrop = (event, areaId) => {
      event.preventDefault();
      if (state.dragging) {
        state.dragging = false;
        return false;
      }
      if (event.dataTransfer.getData("shift")) {
        // console.log("shift", event.dataTransfer);
        const shiftId = event.dataTransfer.getData("shift");
        let target = state.shifts.find((x) => x.id == shiftId);
        target.area = areaId;
        if (state.draggingX) {
          let c = state.rosterWidth / 12;
          target.x = parseInt(state.draggingX / c);
          state.draggingX = 0;
        }
      } else if (event.dataTransfer.getData("emp")) {
        var newShift = state.blankShift;
        newShift.emp = event.dataTransfer.getData("emp");
        newShift.area = areaId;
        newShift.id = state.shifts.length;
        if (state.draggingX) {
          let c = state.rosterWidth / 12;
          newShift.x = parseInt(state.draggingX / c);
          state.draggingX = 0;
        }
        state.shifts.push({ ...newShift });
        state.areas.find((x) => x.id == areaId).row++;
      } else return;
    };
    const onDropShift = (event, item) => {
      const shiftId = event.dataTransfer.getData("shift");
      if (shiftId != item.id) {
        state.dragging = true;
        confirm(`${shiftId} is onDropShift swap to ${item.id}`);
      }
    };
    const onDragEnter = (event, area) => {
      event.preventDefault();
      // console.log(event.x, event.y, event.offsetX, event.offsetY);
    };
    const onDragOver = (event) => {
      event.preventDefault();
      state.draggingX = event.offsetX;
    };
    return {
      state,
      rosterRef,
      areaShiftSort,
      onDrop,
      onDragOver,
      onDragEnter,
      empDragStart,
      shiftDragStart,
      onDropShift,
      areaRow,
      areaHeight,
      shiftTop,
    };
  },
};
</script>

<template>
  <h1>{{ msg }}</h1>
  <div class="flex">
    <div class="emp">
      <div
        v-for="item in state.emps"
        :key="item.id"
        class="emp-item"
        draggable="true"
        @dragstart="empDragStart($event, item)"
      >
        {{ item.name }}
      </div>
    </div>
    <div class="roster-board" ref="rosterRef">
      <div v-for="area in state.areas" :key="area.id">
        <div class="area-title">{{ area.name }}</div>
        <div
          class="roster co-dragger"
          :cod-row="area.row"
          @dragenter="onDragEnter($event, area.id)"
          @dragover="onDragOver($event)"
          @drop="onDrop($event, area.id)"
          :style="{ height: areaHeight(area.id) }"
        >
          <div
            v-for="(item, index) in areaShiftSort(area.id)"
            :key="item.id"
            class="shift-item co-dragger-item"
            :cod-x="item.x"
            :cod-y="item.y"
            :cod-h="item.h"
            :cod-w="item.w"
            draggable="true"
            @dragstart="shiftDragStart($event, item)"
            @drop="onDropShift($event, item)"
            @dragenter.prevent
            @dragover.prevent
            :style="{ top: shiftTop(index) }"
          >
            <!-- <div v-if="resize" class="resize-handle-left"></div> -->
            <!-- <div v-if="resize" class="resize-handle-right"></div> -->
            shift {{ item.id }} - {{ item.emp }} - start at {{ item.x }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.emp {
  width: 200px;
}
.roster-board {
  flex-grow: 1;
}
.shift-item {
  background-color: moccasin;
  position: relative;
  height: 70px;
  resize: horizontal;
  transition: left 200ms, top 400ms;
}
.shift-item:focus {
  background: #42b983;
}
.emp-item {
  background: paleturquoise;
}
.emp,
.roster,
.area-title {
  border: 1px solid #ccc;
}
a {
  color: #42b983;
}
.flex {
  display: flex;
  height: 500px;
}
.resize-handle-left {
  left: 5px;
}
.resize-handle-right {
  right: 5px;
}
.resize-handle-left,
.resize-handle-right {
  position: absolute;
  background-color: #42b983;
  height: 50px;
  width: 5px;
  top: 5px;
}
</style>
