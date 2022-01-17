<script>
import { reactive } from "vue";
export default {
  props: {
    msg: String,
  },
  setup(props) {
    const state = reactive({
      areas: [
        {
          id: 0,
          name: "area1",
        },
        {
          id: 1,
          name: "area2",
        },
      ],
      emps: [
        { id: 1, name: "emp1" },
        { id: 2, name: "emp2" },
      ],
      shifts: [
        {
          id: 0,
          name: "shift1",
          emp: "emp0",
          area: 0,
          content: "one shift",
          x: 0,
          y: 0,
          w: 5,
        },
        {
          id: 1,
          name: "shift2",
          emp: "emp0",
          area: 1,
          content: "another shift",
          x: 1,
          y: 1,
          w: 5,
        },
        {
          id: 2,
          name: "shift3",
          emp: "emp0",
          area: 0,
          content: "3rd shift",
          x: 1,
          y: 1,
          w: 4,
          locked: "yes",
        },
      ],
      blankShift: {
        id: 0,
        name: "new shift",
        emp: "",
        area: null,
        content: "",
        x: 0,
        y: 0,
        w: 5,
      },
    });
    const empDragStart = (event, emp) => {
      // console.log("empDragStart", emp);
      event.dataTransfer.dropEffect = "copy";
      event.dataTransfer.setData("emp", emp.id);
    };
    const shiftDragStart = (event, shift) => {
      // console.log("shiftDragStart", shift);
      event.dataTransfer.dropEffect = "move";
      event.dataTransfer.setData("shift", shift.id);
    };
    const onDrop = (event, areaId) => {
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
      } else return;
    };
    const onDropShift = (event, item) => {
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
        class="roster"
        @dragenter.prevent
        @dragover.prevent
        @drop="onDrop($event, area.id)"
      >
        <div
          v-for="item in state.shifts.filter((x) => x.area == area.id)"
          :key="item.id"
          class="shift-item"
          draggable="true"
          @dragstart="shiftDragStart($event, item)"
          @drop="onDropShift($event, item)"
        >
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
}
.emp-item {
  background: paleturquoise;
}
.emp,
.roster {
  border: 1px solid #ccc;
  min-height: 250px;
}
a {
  color: #42b983;
}
.flex {
  display: flex;
  height: 500px;
}
</style>
