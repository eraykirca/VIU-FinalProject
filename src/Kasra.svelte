<script>
  import TreeMap from './treemap.svelte';
  import { global_human_day } from './global_day2.js';
  import { Graphic, Polygon } from '@snlab/florence'
  import * as animateScroll from "svelte-scrollto";
  export let myVariable;
  let data = global_human_day;
  let footerText = "Graphical Representation of Average Daily Human Activities Worldwide";

  function handleMouseOver(event) {
    // Access the description from event detail
    const { description } = event.detail;
    footerText = description;
  }

  function handleMouseOut() {
    footerText = "Graphical Representation of Average Daily Human Activities Worldwide";
  }
</script>

<main>
  <TreeMap {data} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} />
  <footer>
    <p>{footerText}</p>
  </footer>
</main>

<style>
  main {
    text-align: left;
    padding: 1rem;
  }

  footer {
    color:'black';
  text-align: center;
  font-family: "Teko";
  font-size: 20px;
  font-weight: 400;
  font-style: normal;
  }

  .tree-map-block {
    transition: transform 0.3s ease, font-weight 0.3s ease;
    cursor: pointer;
  }
</style>
