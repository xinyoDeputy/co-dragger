<script>
import { computed, reactive } from "vue";
import data from "../assets/store.json";
export default {
  props: {
    msg: String,
    resize: Boolean,
  },
  setup(props) {
    const state = reactive({
      ...data,
      dragging: false,
    });
    const areaRow = (areaId) => {
      return state.shifts.filter((x) => x.area == areaId).length;
    };
    const areaHeight = (r) => {
      return state.rowHeight * (areaRow(r) + 1) + "px";
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
      if (state.dragging) {
        state.dragging = false;
        return false;
      }
      if (event.dataTransfer.getData("shift")) {
        // console.log("shift", event.dataTransfer);
        const shiftId = event.dataTransfer.getData("shift");
        state.shifts.find((x) => x.id == shiftId).area = areaId;
      } else if (event.dataTransfer.getData("emp")) {
        var newShift = state.blankShift;
        newShift.emp = event.dataTransfer.getData("emp");
        newShift.area = areaId;
        newShift.id = state.shifts.length;
        state.shifts.push({ ...newShift });
        state.areas.find((x) => x.id == areaId).row++;
      } else return;
    };
    const onDropShift = (event, item) => {
      state.dragging = true;
      const shiftId = event.dataTransfer.getData("shift");
      console.log(shiftId, "onDropShift swap to", item.id);
    };
    const onDragOver = (event) => {
      // console.log("onDragOver", event);
      // event.dataTransfer.dropEffect = 'copy';
      // event.dataTransfer.setData('itemID', item.id);
    };
    return {
      state,
      onDrop,
      onDragOver,
      empDragStart,
      shiftDragStart,
      onDropShift,
      areaRow,
      areaHeight,
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
    <div class="roster-board">
      <div
        v-for="area in state.areas"
        :key="area.id"
        class="roster co-dragger"
        :cod-row="area.row"
        @dragenter.prevent
        @dragover.prevent
        @drop="onDrop($event, area.id)"
        :style="{ height: areaHeight(area.id) }"
      >
        <div
          v-for="item in state.shifts.filter((x) => x.area == area.id)"
          :key="item.id"
          class="shift-item"
          :cod-x="item.x"
          :cod-y="item.y"
          :cod-h="item.h"
          :cod-w="item.w"
          draggable="true"
          @dragstart="shiftDragStart($event, item)"
          @drop="onDropShift($event, item)"
          @dragenter.prevent
          @dragover.prevent
        >
          <div v-if="resize" class="resize-handle-left"></div>
          <div v-if="resize" class="resize-handle-right"></div>
          {{ item.id }} - {{ item.emp }}
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
  background: moccasin;
  position: relative;
  height: 70px;
  resize: horizontal;
}
.emp-item {
  background: paleturquoise;
}
.emp,
.roster {
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
