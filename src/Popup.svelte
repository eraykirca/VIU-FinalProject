<script>
  import { Graphic, Symbol_, Polygon, } from '@snlab/florence'
  import * as animateScroll from "svelte-scrollto";
  export let myVariable;
</script>
<div class="main">
  <div class="country">
    <h2>More data for {myVariable} </h2>
  </div>

<div class="prompt">
  <Graphic width={1400} height={30} scaleX={[0, 100]} scaleY={[0, 10]} padding={0}>
  onMousedown={() => animateScroll.scrollToTop()}
  <Polygon
    x={[89, 87, 85]}
    y={[10, 2.5, 10]}
    fill={'rgb(60, 60, 60)'}
    onMousedown={() => animateScroll.scrollToTop()}/>
  </Graphic>

</div>
</div>
<!-- <div class="prompt">
  <Graphic width={80} height={30} backgroundColor={"rgb(240, 219, 200)"}
  onMousedown={() => animateScroll.scrollToTop()}>
  </Graphic>
  <Graphic width={25.25} height={30} backgroundColor={'rgb(60, 60, 60)'}
  onMousedown={() => animateScroll.scrollToTop()}>>
  </Graphic>
</div> -->
  


<style>
.country {
  font-family: "Teko", serif;
  font-weight: 200;
  color: rgb(60, 60, 60);
  text-align: left;
  position: absolute;
  margin-top:0;

}

</style>