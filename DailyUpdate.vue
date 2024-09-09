<template>
  
  <div class="container-wrapper">
    <div v-for="(data, category) in categorizedData" :key="category" class="category-container">
      
      <HeatmapChart
        :processedData="data"
        @update-tooltip="updateTooltipContent"
        @move-tooltip="moveTooltip"
        @hide-tooltip="hideTooltip"
      />
    </div>
    <AttendanceProcess/>
    
    
    <div ref="tooltip" class="tooltip" v-if="tooltipData" :style="tooltipStyle">
      <div>
        <div><strong>Name:</strong> {{ tooltipData.name }}</div>
        <div><strong>Category:</strong> {{ tooltipData.category }}</div>
        <div><strong>Content:</strong> {{ tooltipData.content }}</div>
      </div>
    </div>
  </div>
</template>
<script>
import HeatmapChart from './HeatmapChart.vue';
import AttendanceProcess from './AttendanceProcess.vue';


export default {
  components: {
    HeatmapChart,
    AttendanceProcess
  },
  data() {
    return {
      jsonData: [],
      categorizedData: {},
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
      fetch('/data.json') 
        .then(response => response.json())
        .then(data => {
          this.jsonData = data;
          this.processData();
        })
        .catch(error => console.error('Error fetching data:', error));
    },
    processData() {
      const categories = ["Emotion", "Learning", "Challenge", "Achievement", "Suggestions","Plan_For_The_Day","Plan_For_Next_Day"];
      const categorizedData = {};

      // Initialize each category
      categories.forEach(category => {
        categorizedData[category] = [];
      });

      // Process each data item and categorize it
      this.jsonData.forEach(dataItem => {
        Object.entries(dataItem).forEach(([name, details]) => {
          categories.forEach(category => {
            if (details[category]) {
              const content = Array.isArray(details[category]) ? details[category].join(', ') : details[category];
              categorizedData[category].push({
                name,
                category,
                content,
                value: this.calculateScore()
              });
            }
          });
        });
      });

      this.categorizedData = categorizedData; 
    },
    calculateScore() {
      return Math.floor(Math.random() * 10) + 1; 
    },
    updateTooltipContent(data) {
      this.tooltipData = data;
      this.tooltipStyle.opacity = 1;
    },
    moveTooltip(event) {
      this.tooltipStyle.left = `${event.pageX + 10}px`;
      this.tooltipStyle.top = `${event.pageY + 10}px`;
    },
    hideTooltip() {
      this.tooltipData = null;
      this.tooltipStyle.opacity = 0;
    }
  }
};
</script>
<style scoped>
.container-wrapper {
  display: flex;
  flex-wrap: wrap; 
  justify-content: flex-start; 
}

.category-container {
  margin-right: 0.1px; 
  margin-bottom: 0.1px; 
  display: inline-block;
}

.tooltip {
  text-align: left;
  width: 550px;
  font-size: 40px;
  background-color: white;
  border: solid 2px;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0px 0px 15px rgba(0, 0, 0, 0.5);
  position: absolute;
  pointer-events: none;
}
</style>
