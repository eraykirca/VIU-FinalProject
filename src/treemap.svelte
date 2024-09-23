<script>
  import { onMount, createEventDispatcher } from 'svelte';
  import * as d3 from 'd3';

  export let data = [];
  const dispatch = createEventDispatcher();

  const width = 1200;
  const height = 500;
  const margin = { top: 20, right: 40, bottom: 20, left: 0 };
  
  onMount(() => {
    const svg = d3.select('#treemap')
      .attr('width', width)
      .attr('height', height);

    const root = d3.hierarchy({ children: data })
      .sum(d => d.hoursPerDay)
      .sort((a, b) => b.hoursPerDay - a.hoursPerDay);

    const treemap = d3.treemap()
      .size([width - margin.left - margin.right, height - margin.top - margin.bottom])
      .padding(1);

    treemap(root);

    const colorScale = d3.scaleOrdinal(d3.schemeSpectral[11])
      .domain(root.leaves().map(d => d.data.Subcategory));

    svg.selectAll('rect')
      .data(root.leaves())
      .enter().append('rect')
      .attr('x', d => d.x0 + margin.left)
      .attr('y', d => d.y0 + margin.top)
      .attr('width', d => Math.max(d.x1 - d.x0, 5))
      .attr('height', d => Math.max(d.y1 - d.y0, 5))
      .attr('fill', d => colorScale(d.data.Subcategory))
      .attr('stroke', '#fff')
      .attr('data-activity', d => d.data.Subcategory)
      .attr('data-description', d => d.data.description) 
      .on('mouseover', (event, d) => {
        dispatch('mouseover', { description: d.data.description });
      })
      .on('mouseout', () => {
        dispatch('mouseout');
      });

    svg.selectAll('text')
      .data(root.leaves())
      .enter().append('text')
      .attr('x', d => (d.x0 + d.x1) / 2 + margin.left)
      .attr('y', d => (d.y0 + d.y1) / 2 + margin.top)
      .attr('text-anchor', 'middle')
      .attr('dy', '.35em')
      .text(d => d.data.Subcategory)
      .style('font-size', '18px')
      .style('fill', '#000')
      .style('weight', '400px')
      .style('font-family', 'Teko');
  });
</script>

<svg id="treemap" viewBox={`0 0 ${width} ${height}`} style="max-width: 100%; height: auto;"></svg>

<style>
  svg {
    font-size: 15px;
  }
</style>

