<template>
  <v-chart
    class="echart-container"
    autoresize
    :option="barChartOptions"
    @legendselectchanged="handleLegendSelectChanged"
  />
</template>

<script setup>
import { computed, ref } from 'vue';
import { use } from 'echarts/core';
import { CanvasRenderer } from 'echarts/renderers';
import { BarChart } from 'echarts/charts';
import { LegendComponent, GridComponent } from 'echarts/components';
import VChart from 'vue-echarts';

use([GridComponent, LegendComponent, BarChart, CanvasRenderer]);

// Create some initial fake data
const initData = ref([
  {
    date: new Date('2023-11-22T17:00:00.000Z'),
    appearances: 1,
    missedOpportunities: 2,
  },
  {
    date: new Date('2023-11-23T17:00:00.000Z'),
    appearances: 2,
    missedOpportunities: 1,
  },
]);

// Track series visibility
const seriesVisibility = ref({
  'Missed Opportunities': true,
  Appearances: true,
});

// Update series visibility when legend is toggled
const handleLegendSelectChanged = (legend) => {
  Object.entries(legend.selected).forEach(([selectedKey, selectedValue]) => {
    seriesVisibility.value[selectedKey] = selectedValue;
  });
};

// Create computed options for the chart
const barChartOptions = computed(() => {
  // Create base series (data for this is added later)
  const baseSeries = [
    {
      name: 'Appearances',
      type: 'bar',
      color: '#FF0000',
      stack: 'ranks',
    },
    {
      name: 'Missed Opportunities',
      type: 'bar',
      color: '#333333',
      stack: 'ranks',
    },
  ];

  // Function to get the top stacked series for each date
  const getTopSeriesForEachDate = () => {
    // Object to store top series name for each date
    const topSeriesForEachDate = {};

    initData.value.forEach((dataPoint) => {
      let topSeriesName = '';
      // Check which series is on top for this data point
      if (
        seriesVisibility.value['Missed Opportunities'] &&
        dataPoint.missedOpportunities > 0
      ) {
        topSeriesName = 'Missed Opportunities';
      } else if (
        seriesVisibility.value['Appearances'] &&
        dataPoint.appearances > 0
      ) {
        topSeriesName = 'Appearances';
      }
      // Store the top series name for this date
      if (topSeriesName) {
        topSeriesForEachDate[dataPoint.date.toDateString()] = topSeriesName;
      }
    });

    return topSeriesForEachDate;
  };

  // Function to add border radius to the top stacked series
  const getSeriesDataWithTopStackBorderRadius = (stackInfo) => {
    // Iterate over base series and create a new series
    const series = baseSeries.map((seriesItem) => {
      // Iterate over initData and create a new array of series data
      const seriesData = initData.value.map((dataPoint) => {
        const dataPointDateString = dataPoint.date.toDateString();
        const dataPointTopStackName = stackInfo[dataPointDateString];
        const isTopStack = dataPointTopStackName === seriesItem.name;

        // Return the data item with the border radius applied
        return {
          value: [
            dataPoint.date,
            seriesItem.name === 'Appearances'
              ? dataPoint.appearances
              : dataPoint.missedOpportunities,
          ],
          itemStyle: {
            borderRadius: isTopStack ? [20, 20, 0, 0] : [0, 0, 0, 0],
          },
        };
      });

      const seriesOption = {
        ...seriesItem,
        data: seriesData,
      };
      return seriesOption;
    });

    return series;
  };

  // Get the new series data with the top stack border radius applied
  const seriesWithTopStackBorderRadius = getSeriesDataWithTopStackBorderRadius(
    getTopSeriesForEachDate()
  );

  // Return the options object
  const options = {
    xAxis: {
      type: 'time',
      axisLabel: {
        formatter: '{d} {MMM} {yy}',
      },
      minInterval: 3600 * 1000 * 24, // 1 day in milliseconds
    },
    yAxis: {
      type: 'value',
      show: true,
      minInterval: 1,
    },
    series: seriesWithTopStackBorderRadius,
    legend: {
      show: true,
    },
  };
  return options;
});
</script>

<style scoped>
.echart-container {
  height: 500px;
  width: 500px;
}
</style>
