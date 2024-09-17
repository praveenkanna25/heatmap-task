<template>
  <div class="navbar">
    <div class="navbar-container">
      <div class="navbar-title">Zoho</div>
      <el-date-picker
        v-model="selectedDateRange"
        type="daterange"
        range-separator="to"
        start-placeholder="Start date"
        end-placeholder="End date"
        @change="onDateRangeChange"
        format="YYYY-MM-DD"
        style="width: 300px;"
      />
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue';
import { ElDatePicker } from 'element-plus';
import 'element-plus/dist/index.css';

export default defineComponent({
  components: {
    ElDatePicker,
  },
  data() {
    return {
      selectedDateRange: null, 
    };
  },
  methods: {
    onDateRangeChange(dates) {
      if (dates && dates.length === 2) {
        let [startDate, endDate] = dates;

        
        if (new Date(startDate) > new Date(endDate)) {
          [startDate, endDate] = [endDate, startDate];
        }

        
        const dateArray = this.generateDateArray(startDate, endDate);


        
        this.$emit('date-range-selected', dateArray);
      }
    },
    generateDateArray(startDate, endDate) {
  const dateArray = [];
  
  //  Date objects for proper handling
  let currentDate = new Date(startDate);
  const end = new Date(endDate);

  //  when `startDate` is before `endDate`
  while (currentDate <= end) {
    const year = currentDate.getFullYear();
    const month = String(currentDate.getMonth() + 1).padStart(2, '0'); // Get month, +1 since it's 0-indexed
    const day = String(currentDate.getDate()).padStart(2, '0');
    
    //  formatted date (YYYY-MM-DD) 
    dateArray.push(`${year}-${month}-${day}`);

    // Move to the next day
    currentDate.setDate(currentDate.getDate() + 1);
  }

  return dateArray;
}
  }
});
</script>

<style scoped>
.navbar {
  background-color: #f5f5f5;
  padding: 10px;
  border-bottom: 1px solid #ddd;
}

.navbar-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.navbar-title {
  font-size: 20px;
  font-weight: bold;
  padding: 10px;
}
</style>
