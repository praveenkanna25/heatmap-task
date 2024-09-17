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
        <div><strong >Attendance</strong></div>
        <div><strong>Name:</strong> {{ tooltipData.name }}</div>
         <div><strong>Date:</strong> {{ tooltipData.date }}</div>
        <div><strong>Check-In:</strong> {{ tooltipData.checkIn }}</div>
        <div><strong>Check-Out:</strong> {{ tooltipData.checkOut }}</div>
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
      jsonData: [],
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
     // console.log('Fetching data for:', date);
      // Fetch data for a single date
      return fetch(`http://192.168.11.153:8082/Attendance/attendancedata-${date}.json`)
        .then(response => {
          if (!response.ok) {
            throw new Error(`Data for ${date} not found`);
          }
          return response.json();
        })
        .then(data => data.data) // Return the 'data' array from the JSON
        .catch(error => {
          console.error('Error fetching data for date:', date, error);
          return [] // Return an empty array if fetching fails
        });
    },

    processData() {
      this.processedData = this.jsonData.map(item => {
        const hoursPresent = this.calculateHoursPresent(item.value);
        return {
          name: item.name,
          date:item.date,
          checkIn: item.value.CheckIn || 'Absent',
          checkOut: item.value.CheckOut || 'Absent',
          hoursPresent: hoursPresent || 0, // Handle absent case
          value: hoursPresent //  hoursPresent directly as the score
        };
      });
    },
    calculateHoursPresent(value) {
      if (!value.CheckIn || !value.CheckOut) {
        return 0; // Absent
      }

      const checkIn = new Date(`2024-01-01T${value.CheckIn}:00`);
      const checkOut = new Date(`2024-01-01T${value.CheckOut}:00`);
    
      const diff = (checkOut - checkIn) / (1000 * 60 * 60); // Difference in hours
      return  Math.round(Math.min(Math.max(diff, 0), 10)); // Score between 0 and 10
    },
    updateTooltipContent(data) {
      this.tooltipData = data;
      this.tooltipStyle.opacity = 1; // Show the tooltip
    },
    updateTooltipPosition(mouseX, mouseY) {
      const tooltip = this.$refs.tooltip;
      if (!tooltip) return;

      const tooltipRect = tooltip.getBoundingClientRect();
      const screenWidth = window.innerWidth;
      const screenHeight = window.innerHeight;

      let top = mouseY + 10; 
      let left = mouseX + 10;

     
      if (left + tooltipRect.width > screenWidth) {
        left = mouseX - tooltipRect.width - 10; 
      }
      if (top + tooltipRect.height > screenHeight) {
        top = mouseY - tooltipRect.height - 10; 
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
 
}
</style>
