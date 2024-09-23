<script>
 //data imports 
  import {hours_by_country} from './hours_by_country.js'
  import {country_info} from './country_info.js'
  import {global_human_day} from './global_day.js'
  import { world } from './world.js'
  import { hours_by_cat} from './hours_by_category_transposed.js'

  // Main datasets imports

  const global_day = new DataContainer(global_human_day)
  const hours_country = new DataContainer(hours_by_country)

  // Merging Datasets
  hours_country.setKey('Subcategory') //set subcategory as keys
  hours_country.addColumn('Category', ['Recreation & sports', "Allocation & Transport", "Raw materials & Manufacturing", "Raw materials & Manufacturing", "Raw materials & Manufacturing", 'Food growth & processing', 'Meals', 'Food growth & processing', "Hygiene & care", "Allocation & Transport", "Hygiene & care", "Raw materials & Manufacturing", 'Cleaning & Waste', 'Recreation & sports', "Allocation & Transport", "Raw materials & Manufacturing", 'Meals', 'Recreation & sports', "Hygiene & care", 'Education & religion', 'Education & religion', 'Sleep & bedrest', 'Recreation & sports', 'Cleaning & Waste'])
  hours_country.addColumn('Global', global_day.column('hoursPerDay'))
  
  const hours_category = new DataContainer(hours_by_cat)

  //countries names
  const country_names = hours_country.columnNames().slice(202).concat(hours_country.columnNames().slice(1, 200))

  //categories
  const categories = hours_country.domain('Category')

  //extra info about the country: population/datastatus/region
  const countries = new DataContainer(country_info)
  countries.setKey('country')
  
  //component imports 
  import { scaleLinear, scaleOrdinal, scaleBand, scaleSequential } from 'd3-scale'
  import { quantize} from 'd3-interpolate'
  import { interpolateSpectral,  interpolateBlues, interpolateGreens, interpolateOranges, interpolatePurples, interpolateReds, interpolateOrRd, interpolateYlGn, interpolateYlOrRd, interpolateGnBu } from 'd3-scale-chromatic'
  import { Graphic, PointLayer, RectangleLayer, LabelLayer, polar, Section, LineLayer, Label, Rectangle, fitScales, PolygonLayer, Polygon } from '@snlab/florence'
  import DataContainer from '@snlab/florence-datacontainer'
  import * as animateScroll from "svelte-scrollto";
  import Child from './Popup.svelte';
  import Child2 from './Zhanna.svelte';
  import Child3 from './Kasra.svelte'

  let scaleColor, categoryClicked, explanatory_graph, pie_chart_country, add_column, pie_chart, explanatory_graph_country, first_country, first_bar_chart, second_country, second_bar_chart, clickedDataset, countrySelection, myColorScale, minMaxCountry

//color picker
  const colorCategory = new DataContainer({
    category: ["Recreation & sports", 'Allocation & Transport', 'Raw materials & Manufacturing', 'Food growth & processing', 'Meals', 'Hygiene & care', 'Cleaning & Waste', 'Education & religion', 'Sleep & bedrest'],
    lower: [ '#ffebee', '#fbe9e7','#fff3e0','#fff8e1', '#fff9c4','#f0f4c3','#e0f7fa','#e3f2fd','#e1bee7'], 
    upper: ['#e53935','#e64a19','#fb8c00','#ffa000','#fbc02d','#7cb342','#009688','#1976d2','#5e35b1']
  })
  colorCategory.setKey('category')

  //building a map
  const worldMap = new DataContainer(world)

   // Last valid country picked by the user 
  let lastValidSelectionIndex = 0;

  // Handles the case where user tries to select a country which is already selected.
  let selectedIndex = null;

  //geoscale
  const myGeoScale = fitScales(worldMap.domain('$geometry'))

  //variables for sleep slider
  let userInput = '';
  let dataToDisplay = "";
  let worldAvg = 9.1;

  let selectedCategory = "Sleep & bedrest";

  //Color interpolation for the map + legend
  let minMax = ['','']
   $: interpolatorPick = colorCategory.row({key: selectedCategory}) //choose a scheme

   $: myColorScale = scaleSequential()
    .domain(hours_category.domain(selectedCategory))
    .range([interpolatorPick.lower,interpolatorPick.upper]) //set up color scale
  
  $: minMax = hours_category.domain(selectedCategory) //domain for the legend

  $: minMaxCountry = hours_category.filter(row => row[selectedCategory] == minMax[0] || row[selectedCategory] == minMax[1]).arrange({[selectedCategory]: 'ascending'}) //countries for the legend

  //function to convert time
  function convert_time(d) {
    if (d*60 > 90) {
      return Math.floor(d*60/60)+' hours '+Math.round(d*60%60)+" minutes"
    } else {
      return Math.round(d*60)+" minutes"
    }
  }
  //picked country selection
  let myVariable = 'Afghanistan';
  $: myVariable = country_names[lastValidSelectionIndex]

  </script>

  <div class="prompt">
    <h3>
      {#if userInput == null || userInput == 0}
          Hi, how many hours do you usually sleep for?
      {:else if userInput < worldAvg}
          You sleep {convert_time((worldAvg - userInput).toFixed(2))} less than the world average!  Curious to see more? Explore how an average person lives their day!
      {:else if userInput == worldAvg}
          You sleep the same amout as the world average!  Curious to see more? Explore how an average person lives their day!
      {:else if userInput > worldAvg}
          You sleep {convert_time((userInput - worldAvg).toFixed(2))} more than the world average! Explore how an average person lives their day!
      {/if}
    </h3>
    <label> Enter here:  </label>
    <label>
        <input type="range" bind:value={userInput} min="0" max="24" />
        <input type="number" bind:value={userInput} min="0" max="24" />
    </label>
    <div class="source">
      <p>This visualization is based on the data from the article by Fajzel, William, et al. <a href="https://doi.org/10.1073/pnas.2219564120">"The Global Human Day."</a>(2023). The data was combined and interpolated from the data collected by various organisations from 2000 to 2019.</p>
    </div>   
  </div>
 
<!--append the treemap-->

  <Child3/>

<!--scroll-button-->
<div class='scroll-to-second'>
  <div class='arrow'>
    <Graphic width={1400} height={30} scaleX={[0, 100]} scaleY={[0, 10]} padding={0}>
    onMousedown={() => animateScroll.scrollToTop()}
      <Polygon
        x={[90, 91.5, 88]}
        y={[10, 1.5, 1.5]}
        fill={'rgb(60, 60, 60)'}
        onMousedown={() => animateScroll.scrollTo({element: '#secondPart'})}/>
    </Graphic>
  </div>
  <div class="credits" id="secondPart">
    <h3>
    made by Zhanna Osipova, Eray Kırca, Kasra Hasani at Venice International University
    </h3>
  </div>
</div>

<!--category select-->
<div>
  <label class="categoryDropdown" for="dropdown">Choose a category:</label>
    <select id="category-select" bind:value={selectedCategory}>
      {#each categories as item}
       <option value={item}>{item}</option>
      {/each}
    </select>
</div>

<div class="main">
  <div class="cat-map">
    <div class="map">
      <Graphic width={1400} height={400}>
        <Section 
            x1={0} x2={0.9} y1={0} y2={1}
            {...myGeoScale} flipY>
          <PolygonLayer
            
            fill={hours_category.map(selectedCategory, myColorScale)}
            onMousedown={() => animateScroll.scrollToBottom()}
            onMouseover={({ index }) => (selectedIndex, lastValidSelectionIndex = index)}
            onMouseup={() => (selectedIndex = null)}
            
            geometry={worldMap.column('$geometry')}
            stroke={({ index }) => (index === lastValidSelectionIndex ? "black" : 'white')}
            strokeWidth={0.5}
          />
        </Section>

        <!--Label layer for info about max and min-->
        <Section 
            x1={0} x2={0.125} y1={0} y2={1}>

            <!--Color-->
            <RectangleLayer 
              x1={[0.1, 0.1]}
              x2={[1, 1]}
              y1={[0.3, 0.5]} 
              y2={[0.25, 0.45]} 
              fill = {[interpolatorPick.lower, interpolatorPick.upper]} />
            <!--time-->
            <LabelLayer 
              x={[0.55, 0.55]} y={[0.35, 0.55]} 
              text={minMax.map(d => convert_time(d))} 
              anchorPoint = 'c' 
              fontSize={20}
              fontWeight={'500'}
              fontFamily={"Teko"} />
            <!---country-->
              <LabelLayer 
              x={[0.55, 0.55]} y={[0.275, 0.475]} 
              text= {minMaxCountry.column('Category')}
              anchorPoint = 'c' 
              fontSize={20}
              fontWeight={'500'}
              fontFamily={"Teko"} />

        </Section>
      </Graphic>
    </div>
    <div class='extra'> <!--extra things to consider-->
      {#if selectedCategory === 'Food growth & processing'}
        <p> These activities has been shown to vary grealty with the GDP per capita. Do you see the trend? </p>
      {:else if selectedCategory === 'Allocation & Transport'}
        <p> One of these activities has been shown to vary significantly with the GDP per capita. Can you figure out which? </p>
      {:else if selectedCategory === 'Recreation & sports'}
        <p> The pattern of time allocation for recreational activities "counteracts" another category. Which one and why? *hint: it has to do with the GDP per capita </p>
      {:else if selectedCategory === 'Meals'}
        <p> Can we make any observations about global food culture patterns? </p>
      {:else if selectedCategory === 'Cleaning & Waste'}
        <p> Is this pattern linked to the maintenance of living spaces or the management of waste? Can a small reallocation of time make a difference?</p>
      {/if}
    </div>
  </div>
</div>

<div>
  <h1 class="dataDisplay">
    {#if myVariable === 'Sudan'}
    Data not available for North Sudan. Global average is shown.
    {:else if dataToDisplay != null && myVariable != null && dataToDisplay != "N"}
    {convert_time(hours_category.column(selectedCategory)[lastValidSelectionIndex])} in {myVariable}
    {/if}
  </h1>
</div>

<!--append piechart-->
<Child {myVariable} />
<Child2 {myVariable} />



<style>
.categoryDropdown,
.prompt {
  font-family: "Teko", serif;
  font-weight: 200;
  color: rgb(60, 60, 60);
  text-align: left;
  position: relative
}
.source {position: absolute;
text-align: left;
margin-top:-80px;
width: 250px;
left:950px;
font-weight: 300;
font-size: 16px;
color: rgb(60, 60, 60);}

.dataDisplay {
  font-family: "Teko", serif;
  font-weight: 200;
  color: rgb(60, 60, 60);
  text-align: center;
}
.credits {
  font-family: "Teko", serif;
  font-weight: 200;
  color: rgb(60, 60, 60);
  text-align: left;
  
}
.scroll-to-second {position: relative;}
.arrow {position:absolute}
.cat-map {position: relative;
  }

.map {position: relative;
  }

.extra {position: absolute;
  width: 300px; 
  left: 15px;
  margin-top: -160px;
  font-family: "Teko";
  font-weight: 400;
  font-size: 18px;
  color: rgb(60, 60, 60);
  text-align: left}
</style>