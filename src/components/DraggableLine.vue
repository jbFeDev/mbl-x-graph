<template>
  <svg ref="chart" width="400" height="400"></svg>

  <pre>{{ info }}</pre>
</template>

<script setup>
import { reactive, computed, onMounted } from 'vue';
import * as d3 from 'd3';

const info = reactive({
  bass: 5,
  treble: -5,
  channel: 1,
});

const data = computed(() => [
  [0, info.bass],
  [50, info.bass],
  [100, info.treble],
  [150, info.treble],
]);

function renderChart() {
  const svg = d3.select('svg');

  const margin = {
    top: 20,
    right: 20,
    bottom: 20,
    left: 20,
  };

  const width = svg.attr('width') - margin.left - margin.right;
  const height = svg.attr('height') - margin.top - margin.bottom;

  const xScale = d3.scaleLinear().rangeRound([0, width]);
  const yScale = d3.scaleLinear().rangeRound([height, 0]);

  const line = d3
    .line()
    .x((d) => xScale(d[0]))
    .y((d) => yScale(d[1]));

  let drag = d3
    .drag()
    .on('start', dragstarted)
    .on('drag', dragged)
    .on('end', dragended);

  svg
    .append('rect')
    .attr('class', 'zoom')
    .attr('fill', 'none')
    .attr('pointer-events', 'all')
    .attr('width', width)
    .attr('height', height)
    .attr('transform', 'translate(' + margin.left + ',' + margin.top + ')');

  const focus = svg
    .append('g')
    .attr('transform', 'translate(' + margin.left + ',' + margin.top + ')');

  xScale.domain(d3.extent(data.value, (d) => d[0]));
  yScale.domain(d3.extent(data.value, (d) => d[1]));

  const path = focus
    .append('path')
    .datum(data.value)
    .attr('class', `channel-${info.channel}`)
    .attr('fill', 'none')
    .attr('stroke', '#009CDE')
    .attr('stroke-linejoin', 'round')
    .attr('stroke-linecap', 'round')
    .attr('stroke-width', 3)
    .attr('d', line);

  focus
    .append('g')
    .selectAll('circle')
    .data(data.value)
    .enter()
    .append('circle')
    .attr('class', `channel-${info.channel}`)
    .attr('data-position', function (d, i) {
      return `${d[0]}`;
    })
    .attr('r', 5.0)
    .attr('cx', function (d) {
      return xScale(d[0]);
    })
    .attr('cy', function (d) {
      return yScale(d[1]);
    })
    .style('cursor', 'pointer')
    .style('fill', 'steelblue');

  focus.selectAll('circle').call(drag);

  function dragstarted(d) {
    d3.select(this).raise().classed('active', true);
  }

  function dragged(event, d) {
    d[1] = yScale.invert(event.y);

    console.log(d[1]);

    if (d[1] >= -5 && d[1] <= 5) {
      if (d[0] === 0 || d[0] === 50) {
        d3.select(`circle.channel-${info.channel}[data-position="0"]`).attr(
          'cy',
          yScale(d[1])
        );
        d3.select(`circle.channel-${info.channel}[data-position="50"]`).attr(
          'cy',
          yScale(d[1])
        );

        info.bass = d[1];
      } else {
        d3.select(`circle.channel-${info.channel}[data-position="100"]`).attr(
          'cy',
          yScale(d[1])
        );
        d3.select(`circle.channel-${info.channel}[data-position="150"]`).attr(
          'cy',
          yScale(d[1])
        );

        info.treble = d[1];
      }

      path.datum(data.value).attr('d', line);
    }
  }

  function dragended(d) {
    d3.select(this).classed('active', false);
  }
}

onMounted(() => {
  renderChart();
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
