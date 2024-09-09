<template>
  <div>
    
    <HeatmapChart
      :processedData="processedData"
      @update-tooltip="updateTooltipContent"
      @move-tooltip="moveTooltip"
      @hide-tooltip="hideTooltip"
    />
    <div ref="tooltip" class="tooltip" v-if="tooltipData" :style="tooltipStyle">
      <div>
        <div><strong>Name:</strong> {{ tooltipData.name }}</div>
        <div><strong>Date:</strong> {{ tooltipData.date }}</div>
        <div><strong>Morning:</strong> {{ tooltipData.morning }}</div>
        <div><strong>Evening:</strong> {{ tooltipData.evening }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import HeatmapChart from './HeatmapChart.vue';

export default {
  components: {
    HeatmapChart
  },
  data() {
    return {
      jsonData: [], // Replace with your JSON data source
      processedData: [],
      tooltipData: null,
      tooltipStyle: {
        position: 'absolute',
        opacity: 0,
        pointerEvents: 'none',
        transition: 'opacity 0.1s ease-in-out'
      }
    };
  },
  mounted() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      // Replace with your data fetching logic
      fetch('/attendance.json')
        .then(response => response.json())
        .then(data => {
          this.jsonData = data;
          this.processData();
        })
        .catch(error => console.error('Error fetching data:', error));
    },
    processData() {
      // Process the data and prepare it for the heatmap
      this.processedData = this.jsonData.map(data => {
        const cellData = data.cells[0].value;
        return {
          name: data.name,
          date: data.date,
          morning: cellData.Morning || 'Absent',
          evening: cellData.Evening || 'Absent',
          value: this.calculateScore(cellData),
        };
      });
    },
    calculateScore(cellData) {
      // Replace with your score calculation logic
      return (cellData.Morning === 'Present' ? 5 : 0) +
             (cellData.Evening === 'Present' ? 5 : 0);
    },
    updateTooltipContent(data) {
      this.tooltipData = data;
      this.tooltipStyle.opacity = 1; // Show the tooltip
    },
    moveTooltip(event) {
      this.tooltipStyle.left = `${event.pageX + 10}px`;
      this.tooltipStyle.top = `${event.pageY + 10}px`;
    },
    hideTooltip() {
      this.tooltipData = null;
      this.tooltipStyle.opacity = 0; // Hide the tooltip
    }
  }
};
</script>

<style scoped>
.tooltip {
  text-align: left;
  width: 350px;
  height: auto;
  font-size: 40px;
  line-height: 1.5;
  background-color: white;
  border: solid 2px;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0px 0px 15px rgba(0,0,0,0.5);
}
</style>
