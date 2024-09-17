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
        <div><strong>FPS</strong></div>
        <div><strong>Name:</strong> {{ tooltipData.name }}</div>
        <div><strong>Date:</strong> {{ tooltipData.date }}</div> 
        <div><strong>Creator:</strong> {{ tooltipData.creator }}</div>
        <div><strong>Paper:</strong> {{ tooltipData.paper }}</div>
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
  props: {
    selectedDateRange: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      jsonData: [], // Will contain combined data from all dates
      processedData: [], // Processed and ready for heatmap
      tooltipData: null,
      tooltipStyle: {
        position: 'absolute',
        opacity: 0,
        pointerEvents: 'none',
        transition: 'opacity 0.1s ease-in-out'
      }
    };
  },
  watch: {
    selectedDateRange: {
      handler(newDates) {
        if (newDates && newDates.length > 0) {
          this.fetchAndCombineData(newDates);
        }
      },
      immediate: true // Run immediately when the component is created
    }
  },
  methods: {
    fetchAndCombineData(dates) {
      // Clear old data to prevent overlap
      this.jsonData = [];
      this.processedData = [];
      
      const fetchPromises = dates.map(date => this.fetchData(date));
      
      // Wait for all data to be fetched
      Promise.all(fetchPromises)
        .then(results => {
          // Combine the data from all the fetched results
          this.jsonData = results.flat(); // Flatten array of arrays
          this.processData();
        })
        .catch(error => console.error('Error fetching data:', error));
    },

    fetchData(date) {
      // Fetch data for a single date
      return fetch(`http://192.168.11.153:8082/FPS/fpsdata-${date}.json`)
        .then(response => {
          if (!response.ok) {
            throw new Error(`Data for ${date} not found`);
          }
          return response.json();
        })
        .then(data => data.data) // Return the 'data' array from the JSON
        .catch(error => {
          console.error('Error fetching data for date:', date, error);
          return []; // Return an empty array if fetching fails
        });
    },

    processData() {
    
      this.processedData = this.jsonData.map(item => {
        return {
          name: item.name,
          date:item.date,
          creator: item.value.Creator,
          paper: item.value.Paper,
          value: this.calculateScore(item.value),
        };
      });
    },

    calculateScore(value) {
      // Example score calculation logic
      return (value.Creator.toLowerCase() === 'yes' ? 5 : 0) +
             (value.Paper.toLowerCase() === 'yes' ? 5 : 0);
    },

    updateTooltipContent(data) {
      // console.log('Tooltip Data:', data);
      this.tooltipData = data;
      this.tooltipStyle.opacity = 1; // Show the tooltip
    },
    updateTooltipPosition(mouseX, mouseY) {
      const tooltip = this.$refs.tooltip;
      if (!tooltip) return;

      const tooltipRect = tooltip.getBoundingClientRect();
      const screenWidth = window.innerWidth;
      const screenHeight = window.innerHeight;

      let top = mouseY + 10; // Add offset for better visibility
      let left = mouseX + 10;

      // Adjust for right edge overflow
      if (left + tooltipRect.width > screenWidth) {
        left = mouseX - tooltipRect.width - 10; // Move to the left of the mouse
      }

      // Adjust for bottom edge overflow
      if (top + tooltipRect.height > screenHeight) {
        top = mouseY - tooltipRect.height - 10; // Move tooltip above the mouse
      }

      // Adjust for top edge overflow
      if (top < 0) {
        top = 10; // Prevent overflow above the screen
      }

      // Adjust for left edge overflow
      if (left < 0) {
        left = 10; // Prevent overflow on the left side
      }

      // Update the tooltip style position
      this.tooltipStyle.left = `${left}px`;
      this.tooltipStyle.top = `${top}px`;
    },


    moveTooltip(event) {
      const mouseX = event.pageX;
      const mouseY = event.pageY;

      // Update tooltip position based on mouse movement
      this.updateTooltipPosition(mouseX, mouseY);
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
  position: absolute;
  pointer-events: none;
  background: #fff;
  border: 1px solid black;
  padding: 15px;
  border-radius: 8px;
  font-size: 24px;
  line-height: 1.5;
  max-width: 400px;
  width: auto;
  height: auto;
  word-wrap: break-word;
  white-space: normal;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1); /* Add some shadow for better visibility */
  z-index: 1000;
  

}
</style>
