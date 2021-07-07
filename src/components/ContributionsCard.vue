<template>
  <div v-if="drawList.length > 0" class="title">
    {{ `Total Commits in last year: ${allCommitCount}` }}
  </div>
  <div v-if="drawList.length > 0" class="year">
    <div>
      <div class="chart-date"></div>
      <div class="chart-date">Mon</div>
      <div class="chart-date"></div>
      <div class="chart-date">Wed</div>
      <div class="chart-date"></div>
      <div class="chart-date">Fri</div>
      <div class="chart-date"></div>
    </div>
    <div v-for="(item, index) in drawList" :key="index" class="week">
      <div
        v-for="(cell, cellIndex) in item"
        :key="cellIndex"
        class="day"
        :class="getLevel(cell)"
        @mouseover="handleMouseOver(cell, $event)"
        @mouseout="handleMouseOut"
      ></div>
    </div>
  </div>
  <div v-show="isToolTipShow" class="tooltip" :style="getPosition">
    {{ `Date:${selectDay.date} Count:${selectDay.count}` }}
  </div>
</template>

<script>
import { defineComponent, ref, watch, computed } from "vue";

export default defineComponent({
  name: "ContributionsCard",
  props: ["contributions", "username"],
  setup(props) {
    const username = props.username;
    const allCommitCount = ref(0);
    const drawList = ref([]);
    const isToolTipShow = ref(false);
    const selectDay = ref({ date: "", count: "", x: "", y: "" });

    const handleMouseOver = (cell, e) => {
      isToolTipShow.value = true;
      if (cell) {
        selectDay.value = {
          date: cell.date,
          count: cell.count,
          x: e.x,
          y: e.y,
        };
      }
    };

    const handleMouseOut = () => {
      isToolTipShow.value = false;
      selectDay.value = {
        date: "",
        count: "",
        x: "",
        y: "",
      };
    };

    const getPosition = computed(() => {
      if (selectDay.value.x || selectDay.value.y) {
        return {
          top: `${selectDay.value.y - 25}px`,
          left: `${selectDay.value.x - 10}px`,
        };
      }
    });

    watch(
      () => props.contributions,
      () => {
        allCommitCount.value = 0;
        drawList.value = [];
        props.contributions.forEach((cell, dayIndex) => {
          //计算所有的提交次数
          allCommitCount.value += Number(cell.count);

          //所有提交 按周进行分组
          const weekIndex = Math.floor(dayIndex / 7);
          if (drawList.value.length <= weekIndex) {
            //没有这个数组 往里面塞一个
            drawList.value.push([]);
          }
          drawList.value[weekIndex].push(cell);
        });
      }
    );

    const getLevel = (cell) => {
      return `l${cell.level}`;
    };

    return {
      allCommitCount,
      username,
      drawList,
      getLevel,
      isToolTipShow,
      handleMouseOver,
      handleMouseOut,
      selectDay,
      getPosition,
    };
  },
});
</script>

<style>
.day {
  width: 12px;
  height: 12px;
  background-color: white;
  border: 0.5px solid rgba(27, 31, 35, 0.06);
  margin-bottom: 2px;
  margin-left: 2px;
  margin-right: 2px;
}

.week {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: flex-start;
}

.year {
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  align-items: flex-start;
  margin-top: 10px;
  margin-left: 10px;
}

.title {
  font-size: 16px;
  margin-top: 20px;
}

.l0 {
  background-color: #ebedf0;
}

.l1 {
  background-color: #9be9a8;
}

.l2 {
  background-color: #40c463;
}

.l3 {
  background-color: #30a14e;
}

.l4 {
  background-color: #216e39;
}

.chart-date {
  width: 30px;
  height: 12px;
  background-color: white;
  border: 0.5px solid transparent;
  margin-bottom: 2px;
  margin-left: 2px;
  margin-right: 10px;
  font-size: 10px;
  text-align: center;
  vertical-align: middle;
  line-height: 12px;
}

.day:hover {
  cursor: pointer;
}

.tooltip {
  background-color: rgba(0, 0, 0, 0.596);
  color: #fff;
  font-size: 12px;
  position: absolute;
  padding: 4px 2px;
}
</style>