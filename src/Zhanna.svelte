<script>
 //data imports 
  import {hours_by_country} from './hours_by_country.js'
  import {country_info, categories_description} from './country_info.js'
  import {global_human_day} from './global_day.js'
  export let myVariable;

  console.log(myVariable)

  // main pie_chart that includes hours spend across activities in different countries + Global

  const global_day = new DataContainer(global_human_day)
  const hours_country = new DataContainer(hours_by_country)
  const categories_desc = new DataContainer(categories_description)
  console.log(categories_desc)
  categories_desc.setKey('Category')
  // Merging Datasets
  hours_country.setKey('Subcategory') //set subcategory as keys
  hours_country.addColumn('Category', ['Recreation & sports', "Allocation & Transport", "Raw materials & Manufacturing", "Raw materials & Manufacturing", "Raw materials & Manufacturing", 'Food growth & processing', 'Meals', 'Food growth & processing', "Hygiene & care", "Allocation & Transport", "Hygiene & care", "Raw materials & Manufacturing", 'Cleaning & Waste', 'Recreation & sports', "Allocation & Transport", "Raw materials & Manufacturing", 'Meals', 'Recreation & sports', "Hygiene & care", 'Education & religion', 'Education & religion', 'Sleep & bedrest', 'Recreation & sports', 'Cleaning & Waste'])
  hours_country.addColumn('Global', global_day.column('hoursPerDay'))
  
  //countries for the first select option
  const country_names = hours_country.columnNames().slice(202).concat(hours_country.columnNames().slice(1, 200))

  //extra info about the country: population/datastatus/region
  const countries = new DataContainer(country_info)
  countries.setKey('country')
  
  
  //component imports 
  import { scaleLinear, scaleOrdinal, scaleBand } from 'd3-scale'
  import { quantize} from 'd3-interpolate'
  import { interpolateSpectral} from 'd3-scale-chromatic'
  import { Graphic, PointLayer, RectangleLayer, LabelLayer, polar, Section, LineLayer, Label, Rectangle } from '@snlab/florence'
  import DataContainer from '@snlab/florence-datacontainer'

  let scaleColor, categoryClicked, explanatory_graph, pie_chart_country, add_column, pie_chart, explanatory_graph_country, first_country, first_bar_chart, second_country, second_bar_chart, clickedDataset, countrySelection, categoryDescription
  
  let ready = true
  let ready2 = false
  let nrow = 9 //for scaling
 
  let infoSelection = ["No country chosen", '...', '...', '...'] //for info-window
  let lastValidSelectionIndex = 0;

  // Handles the case where user tries to select a country which is already selected.
  let selectedIndex = null;

  //data for the pie chart
  $: if (myVariable !== "Greenland") {
      pie_chart_country = hours_country.select([myVariable, 'Category', 'Subcategory'])
      pie_chart_country.addColumn('hoursPerDay', pie_chart_country.column(myVariable))
      pie_chart = pie_chart_country.groupBy("Category")
        .summarise({ total_hours: {hoursPerDay: 'sum' }})
        .cumsum(
          { activities: 'total_hours' },
          { asInterval: true }
        ).mutate({
          lower: row => row.activities[0],
          upper: row => row.activities[1]
        }).mutate({
          mid: ({ lower, upper }) => lower + (upper - lower) / 2})
      clickedDataset=pie_chart_country //updates the stacked chart
      }

  //data for the info window
  $: if (myVariable !== 'Global') {
      countrySelection = countries.row({ key: myVariable })
      infoSelection = [countrySelection.country, countrySelection.region_code, countrySelection.population, countrySelection.dataStatus]
    }

//color scaling
  const colors = quantize(
      t => interpolateSpectral(t * 0.9 + 0.1),
      nrow
    )
  
  scaleColor = scaleOrdinal()
      .domain(hours_country.domain('Category'))
      .range(colors)

//function to display time in minutes
  function convert_time(d) {
    if (d*60 > 90) {
      return Math.floor(d*60/60)+'h '+Math.round(d*60%60)+"min"
    } else {
      return Math.round(d*60)+"min"
    }
  }
//simple clock hand imitation
  function return_random() {
    return Math.floor(Math.random()*20)
  }

  function handleClick(category, data) {
    // handle click of the bar or sector
    categoryClicked = data.column("Category")[category.index]
    categoryDescription = categories_desc.row({key: categoryClicked}).Description
    clickedDataset = pie_chart_country
    ready2=true
  }

    //data for a stacked barchart
  $: if (ready2) {
        explanatory_graph = clickedDataset.filter(row => row.Category == categoryClicked).cumsum(
          { activities: 'hoursPerDay'},
          { asInterval: true }
          ).mutate({
            lower: row => row.activities[0],
            upper: row => row.activities[1]
          }).mutate({
            mid: ({ lower, upper }) => lower + (upper - lower) / 2})
  
      }

  function reset (){
    //registers a mouseout and resets everything
    lastValidSelectionIndex = ''
  }
</script>

<div class="graph">
  <div>
  <div class='category'>
    {#if ready2}
      <p><span style="font-weight:600" >{categoryClicked} category </span> includes {categoryDescription}</p>
    {:else}
      <p> Click on a category to see more! </p>
    {/if}
    </div>
    <div class = "upper-chart">
      <Graphic width={1400} height={500}>
        <!--Pie chart section-->
        {#if ready}
          <!--Donut chart part-->
          <Section
          x1={0.15} x2={0.436} y1={0.1} y2={0.9} padding={3}
            scaleX={[0, pie_chart.max('activities')]}
            scaleY={[0, 30]}
            coordinates={polar({ startAngle: 0, endAngle: 2 * Math.PI })}
          >
          <!--Donut chart-->
            <RectangleLayer 
              x1={pie_chart.column('lower')}
              x2={pie_chart.column('upper')}
              y1={Array(nrow).fill(5)}
              y2={Array(nrow).fill(27)}
              fill={pie_chart.map('Category', scaleColor)}
              onMouseover={({ index }) => (selectedIndex, lastValidSelectionIndex = index)}
              onMouseout={() => reset()}
              stroke={({index}) => (index === lastValidSelectionIndex ? "black" : 'none')}
              onMousedown={({index}) => handleClick({index}, pie_chart)}
              fontFamily={"Teko"}
            />

            <!--clock hands-->
            <LineLayer 
            x={[[0, return_random()], [0, return_random()]]}
            y={[[0, 10], [0, 12]]}
            stroke={'black'}
            strokeWidth={[4, 2]}
            fontFamily={"Teko"}
          />
        <!-- Hour labels -->
          <LabelLayer
              x={pie_chart.column('mid').slice(0, nrow)}
              y={Array(nrow).fill(22)}
              text={pie_chart.column('total_hours').slice(0, nrow).map(d => convert_time(d))}
              fontSize={16}
              fontWeight={500}
              fontFamily={"Teko"} 
            />

          </Section>
         
        <!--Labels for donut chart-->
          <Section
            x1={0.05} x2={0.52} y1={0} y2={1} padding={3}
            scaleX={[0, pie_chart.max('activities')]}
            scaleY={[0, 30]}
            coordinates={polar({ startAngle: 0, endAngle: 2 * Math.PI })}
            fontFamily={"Teko"}>
            
            <!-- Category names -->
            <LabelLayer
              x={pie_chart.column('mid')}
              y={Array(nrow).fill(25)}
              text={pie_chart.column('Category').slice(0, nrow)}
              fontSize={16}
              anchorPoint = 'b'
              fontWeight={600}
              fontFamily={"Teko"}
            />

          </Section>
          {/if}


        <!--small info section-->
      <Section 
          x1={0.75} x2={1} y1={0} y2={1}>

          <LabelLayer 
            x={[0.2, 0.2, 0.2]} y={[0.3, 0.5, 0.7]} 
            text={['Region:','Population:', "Data status:"]} 
            anchorPoint = 'l' 
            fontSize={18}
            fontWeight={'500'}
            fontFamily={"Teko"} />

          <LabelLayer 
            x={[0.2, 0.2, 0.2, 0.2]} y={[0.2, 0.4, 0.6, 0.8]} 
            text={infoSelection} 
            anchorPoint = 'l' 
            fontSize={22}
            fontWeight={'600'}
            fontFamily={"Teko"}/>

        </Section>

        {#if ready2}
          <Section 
            x1={0.53} x2={0.63} y1={0} y2={0.9}
            scaleX={scaleLinear([0,1])}
            scaleY = {scaleLinear().domain([0,explanatory_graph.max('upper')+0.5]).nice()} 
            flipY
            padding ={10}
          >
        
            <!--extra subcategories stacked bar -->
            <RectangleLayer 
              x1={explanatory_graph.map('Subcategory', subcategory => 0)} x2={explanatory_graph.map('Subcategory', subcategory => 1)} 
              y1={explanatory_graph.column('lower')}
              y2={explanatory_graph.column('upper')}  
              fill={explanatory_graph.map('Subcategory', scaleColor)}/>
            <!--hours label-->
            <LabelLayer 
              x={explanatory_graph.map('Subcategory', subcategory => 0.25)} y={explanatory_graph.column('mid')} 
              text = {explanatory_graph.column('hoursPerDay').map(d => convert_time(d))} 
              anchorPoint = 'l' 
              fontSize={16} 
              fontFamily={"Teko"}
              fontWeight={500}/>
          </Section>
          <Section 
            x1={0.63} x2={1} y1={0} y2={0.9}
            scaleX={scaleLinear([0,1])}
            scaleY = {scaleLinear().domain([0,explanatory_graph.max('upper')+0.5]).nice()} 
            flipY
            padding={10}> 
        
            <!--extra subcategories labels-->
            <LabelLayer 
              x={explanatory_graph.map('Subcategory', subcategory => 0)} y={explanatory_graph.column('mid')} 
              text = {explanatory_graph.column('Subcategory')} 
              anchorPoint = 'l' 
              fontSize={18} 
              fontFamily={"Teko"}
              fontWeight={600}/>
          </Section>

        {/if}
      </Graphic>
      <div class='easter-egg'>
        {#if myVariable === "Russian Federation"}
          <p> Yay! You clicked on the country Zhanna is from! Zhanna says:&nbsp&nbsp<span style="font-family: Courier New; font-size: 18px" > Ты отлично выглядишь сегодня! Хорошего дня! </p>
        {:else if myVariable === "Islamic Republic of Iran"}
          <p>Yay! You clicked on the country Kasra is from! Kasra says:&nbsp&nbsp<span style="font-family: Courier New; font-size: 18px" > شما هم همینقدر میخوابید:)؟ </p>
        {:else if myVariable === "Türkiye"}
          <p>Yay! You clicked on the country Eray is from! Eray says:&nbsp&nbsp<span style="font-family: Courier New; font-size: 18px" > “Yurtta sulh, cihanda sulh.”</p>
        {:else}
          <p>&nbsp;</p>
        {/if}
      </div>
    </div> 
    
  </div>
  
</div>



<style>
.graph {width: 1400px;
padding: 1px
}

.category {color:'black';
text-align: left;
font-family: "Teko";
font-size: 18px;
font-weight: 400;
font-style: normal;
width: 1000px
}

.easter-egg {
font-family: "Teko";
font-size: 18px;
font-weight: 400;
}
</style>