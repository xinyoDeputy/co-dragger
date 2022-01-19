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
    const cell = computed(() => {
      return state.rosterWidth / 12;
    });
    const state = reactive({
      ...data,
      resizeStartX: null,
      resizeShiftX: null,
      resizeShiftW: null,
      resizingShift: null,
      resizeDirection: null,
      dragging: false,
      resizing: false,
      draggingX: 0,
      resizeHover: false,
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
          target.x = parseInt(state.draggingX / cell.value);
          state.draggingX = 0;
        }
      } else if (event.dataTransfer.getData("emp")) {
        var newShift = state.blankShift;
        newShift.emp = event.dataTransfer.getData("emp");
        newShift.area = areaId;
        newShift.id = state.shifts.length;
        if (state.draggingX) {
          newShift.x = parseInt(state.draggingX / cell.value);
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
    const resizeStart = (event, direction, shiftId) => {
      event.preventDefault();
      state.resizing = true;
      console.log("start");
      // let shift = state.shifts.find((x) => x.id == shiftId);
      // shift.w = 8;
      state.resizeDirection = direction;
      state.resizeStartX = event.clientX; // init start x
      state.resizingShift = state.shifts.find((x) => x.id == shiftId);
      state.resizeShiftX = state.resizingShift.x;
      state.resizeShiftW = state.resizingShift.w;
      window.addEventListener("mousemove", onResize);
      window.addEventListener("mouseup", resizeStop);
    };
    const onResize = (event) => {
      console.log("on resize", event, state.resizeDirection);
      if (state.resizing) {
        switch (state.resizeDirection) {
          case "w": {
            state.resizingShift.w =
              state.resizeShiftW -
              parseInt((event.clientX - state.resizeStartX) / cell.value);
            state.resizingShift.x =
              state.resizeShiftX +
              parseInt((event.clientX - state.resizeStartX) / cell.value);
            break;
          }
          case "e": {
            state.resizingShift.w =
              state.resizeShiftW +
              parseInt((event.clientX - state.resizeStartX) / cell.value);
            break;
          }
          case "n": {
          }
          case "s": {
          }
          default: {
          }
        }
      }
      console.log(event.clientX);
    };
    const resizeStop = (event) => {
      event.preventDefault();
      if (state.resizing) {
        state.resizing = false;
        // state.resizeStartX = null;
        // state.resizeShiftW = null;
        // state.resizingShift = null;
        window.removeEventListener("mousemove", onResize);
        console.log("stop");
      }
    };

    return {
      state,
      cell,
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
      resizeStart,
      onResize,
      resizeStop,
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
            v-for="(shift, index) in areaShiftSort(area.id)"
            :key="shift.id"
            class="shift-item co-dragger-item"
            :cod-x="shift.x"
            :cod-y="shift.y"
            :cod-h="shift.h"
            :cod-w="shift.w"
            draggable="true"
            @dragstart="shiftDragStart($event, shift)"
            @drop="onDropShift($event, shift)"
            @dragenter.prevent
            @dragover.prevent
            :style="{ top: shiftTop(index) }"
          >
            <div v-if="resize">
              <div
                class="resize-handle-left"
                @mousedown="resizeStart($event, 'w', shift.id)"
              ></div>
              <div
                class="resize-handle-right"
                @mousedown="resizeStart($event, 'e', shift.id)"
              ></div>
            </div>
            shift {{ shift.id }} - {{ shift.emp }} - start at {{ shift.x }}
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
  overflow: visible;
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
  left: -10px;
  cursor: ew-resize;
}
.resize-handle-right {
  right: -10px;
  cursor: ew-resize;
}
.resize-handle-left,
.resize-handle-right {
  position: absolute;
  background-color: #42b983;
  height: 70px;
  width: 15px;
  top: 0;
}
</style>
