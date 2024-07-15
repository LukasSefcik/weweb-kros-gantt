<template>
  <DxGantt
      :task-list-width="500"
      scale-type="weeks"
  >
    <DxTasks :data-source="dataSource"/>

    <DxColumn
        :width="300"
        data-field="title"
        caption="Názov"
    />
    <DxColumn
        data-field="started_at"
        caption="Start Date"
    />
    <DxColumn
        data-field="end"
        caption="End Date"
    />
  </DxGantt>
</template>

<script>
import "./dx.fluent.dx-light-theme.css";
import "./dx.light.css";
import "./dx-gantt.min.css";

import {DxColumn, DxGantt, DxTasks} from 'devextreme-vue/gantt';

export default {
  components: {
    DxColumn,
    DxGantt,
    DxTasks
  },
  data() {
    return {
      minDate: new Date("1.1.2024"),
      dates: [],
      dayWidth: 1,
      dataSource: [...this.content.data]
    };
  },
  props: {
    content: {type: Object, required: true},
  },
  emits: ["trigger-event"],
  computed: {},
  methods: {
    onDragStart(e) {
      console.log("onDragStart", e);
      e.itemData = this.dataSource[e.fromIndex];
    },
    onAdd(e) {
      console.log("onAdd", e);
      const newData = [...this.dataSource];
      newData.splice(e.toIndex, 0, e.itemData);
      this.dataSource = newData;
    },
    onRemove(e) {
      console.log("onRemove", e);
      const newData = [...this.dataSource];
      newData.splice(e.fromIndex, 1);
      this.dataSource = newData;
    },
    onReorder(e) {
      console.log("onReorder", e);
      this.onRemove(e);
      this.onAdd(e);
      this.$emit("trigger-event", {
        name: "onReorder",
        event: e,
      });
    },
    onItemClick(e) {
      console.log("onItemClick", e);
      this.$emit("trigger-event", {
        name: "onItemClick",
        event: e,
      });
    },
    getDayDiff(start, end) {
      const oneDay = 24 * 60 * 60 * 1000;
      return Math.round((end.getTime() - start.getTime()) / oneDay);
    },
    getDaysSinceStart(date, start) {
      const oneDay = 24 * 60 * 60 * 1000;
      return Math.round((date.getTime() - start.getTime()) / oneDay);
    },
    getItemMaxDate(item) {
      if (item && item._indicators_of_sheets?.length > 0) {
        const dates = item._indicators_of_sheets.flatMap(indicator => new Date(indicator.checkpoint_at));
        return new Date(Math.max(...dates));
      } else {
        return new Date();
      }
    },
    getMinDate() {
      // potential performance issue
      if (!this.dates || this.dates.length === 0) {
        this.dates = this.content.data.flatMap(event => [
          new Date(event.started_at),
          ...event._indicators_of_sheets
              .filter(indicator => indicator.checkpoint_at !== null)
              .map(indicator => new Date(indicator.checkpoint_at))
        ]);
      }
      return new Date(Math.min(...this.dates));
    },
  }
};
</script>

<style lang="scss" scoped>
.gantt-events {
  position: relative;
  display: flex;
  flex-direction: column;
}

.gantt-event {
  position: absolute;
  background-color: #3498db;
  color: white;
  padding: 0 8px;
  border-radius: 4px;
}

.gantt-checkpoint {
  position: absolute;
  width: 10px;
  height: 10px;
  background-color: red;
  border-radius: 50%;
  top: 50%;
  transform: translateY(-50%);
}
</style>
