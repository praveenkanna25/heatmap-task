<template>
  <div id="heatmap-container">
    <svg ref="heatmap"></svg>
  </div>
</template>

<script>
import * as d3 from 'd3';

export default {
  name: 'HeatmapChart',
  props: {
    processedData: {
      type: Array,
      required: true
    }
  },
  watch: {
    processedData: 'createHeatmap'
  },
  mounted() {                                                                                                          
    this.createHeatmap();
  },
  methods: {
    createHeatmap() {  
      const margin = { top: 5, right: 5, bottom: 5, left: 5 };
      const cellSize = 150;
      const numMembers = this.processedData.length;
      const cellsPerRow = Math.ceil(Math.sqrt(numMembers));
      const containerSize = cellSize * cellsPerRow;

      const svg = d3.select(this.$refs.heatmap)
        .attr('viewBox', `0 0 ${containerSize + margin.left + margin.right} ${containerSize + margin.top + margin.bottom}`)
        .attr('preserveAspectRatio', 'xMidYMid meet')
        .attr('width', containerSize + margin.left + margin.right)
        .attr('height', containerSize + margin.top + margin.bottom)
        .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);

      const colorScale = d3.scaleLinear()
        .domain([0, 5, 10])
        .range(["#F9210C", "#839B89", "#66CB7F"]);

      svg.selectAll('rect')
        .data(this.processedData.map((d, index) => ({
          ...d,
          x: (index % cellsPerRow) * cellSize,
          y: Math.floor(index / cellsPerRow) * cellSize
        })))
        .enter()
        .append('rect')
        .attr('x', d => d.x)
        .attr('y', d => d.y)
        .attr('width', cellSize)
        .attr('height', cellSize)
        .style('fill', d => colorScale(d.value))
        .style('stroke', 'white')
        .style('stroke-width', '0.1px')
        .on('mouseover', (event, d) => {
          this.$emit('update-tooltip', d);
          this.$emit('move-tooltip', event);
        })
        .on('mousemove', (event) => {
          this.$emit('move-tooltip', event);
        })
        .on('mouseleave', () => {
          this.$emit('hide-tooltip');
        });
    }
  }
};
</script>

<style scoped>
#heatmap-container {
  display: flex;
  justify-content: top;
  align-items: left;
  margin: auto;
  width: 100%;
  height: 100%;
  max-width: 100%;
  max-height: 100%;
  box-sizing: border-box;

}
</style>
