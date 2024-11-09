<script>
    import * as d3 from 'd3';
    export let data;
    const width = 1000;
    const height = 600;
    const marginTop = 20;
    const marginBottom = 100;
    const marginRight = 150;
    const marginLeft = 100;

    let svg;

    //for axis
    let gx;
    let gy;

    let dataFilter = data;
    let positionFilter = [];

    let circles;
    let currCircles = Array.from(d3.select(circles).selectAll("circle"));
    let selectButt;

    let closest = 0;

    let fullyLoaded = false;

    const color = {
        normal: "#9FA19F",
        fighting: "#FF8000",
        flying: "#FF8000",
        poison: "#9141CB",
        ground: "#9141CB",
        rock: "#AFA981",
        bug: "#91A119",
        ghost: "#704170",
        steel: "#60A1B8",
        fire: "#E62829",
        water: "#2980EF",
        grass: "#3FA129",
        electric: "#FAC000",
        psychic: "#EF41CB",
        ice: "#3DCEF3",
        dragon: "#5060E1",
        dark: "#624D4E",
        fairy: "#EF70EF",
        stellar: "40B5A5"
    }

    $: x = d3  
        .scaleLinear()
        .domain([0, 1100])
        .range([marginLeft, width - marginRight]);
    $: y = d3  
        .scaleLinear()
        .domain(d3.extent(data, (d) => d.bst))
        .nice()
        .range([height - marginBottom, marginTop]);
    
    //axis
    $: d3.select(gx).call(d3.axisBottom(x));
    $: d3.select(gy).call(d3.axisLeft(y));

    //tooltip
    let tooltipPt = null;
    
    function onPointerMove(event) {
        if (!fullyLoaded) { return; }
        const [xValue, yValue] = d3.pointer(event);

        closest = findClosestCircle([xValue, yValue]);
        
        tooltipPt = dataFilter[closest];
    }

    $: closest, selectCircle(closest);

    $: d3.select(svg)
    .on('pointerenter pointermove', onPointerMove)

    //filtering
    function update(selectedType) {
        if (!fullyLoaded) { return; }
        dataFilter = [];
        positionFilter.length = 0;
        
        // Create new data with the selection
        for (let i = 0; i < data.length; i++) {
            //handle if they contain the type
            if (selectedType ==="all") {
                dataFilter.push({
                    name: data[i].name,
                    bst: data[i].bst,
                    type1: data[i].type1,
                    type2: data[i].type2,
                    order: data[i].order,
                    sprite: data[i].sprite,
                })
            }
            else if (containsType(data[i], selectedType)){
                dataFilter.push({
                    name: data[i].name,
                    bst: data[i].bst,
                    type1: data[i].type1,
                    type2: data[i].type2,
                    order: data[i].order,
                    sprite: data[i].sprite,
                })
            }
        }
        //delete all circles
        d3.select(circles).selectAll("*").remove();
        //put them back
        for (let i = 0; i < dataFilter.length; i++) {
            d3.select(circles).append("circle")
                .attr("key", dataFilter[i].order)
                .attr("cx", x(dataFilter[i].order))
                .attr("cy", y(dataFilter[i].bst))
                .attr("fill", color[dataFilter[i].type1])
                .attr("r", 2.5);

            positionFilter.push({
                cx: x(dataFilter[i].order),
                cy: y(dataFilter[i].bst),
            });
        }
        currCircles = Array.from(d3.select(circles).selectAll("circle"));
    }

    function findClosestCircle(mousePosition) {
        const [mx, my] = mousePosition;
        let closestCircle = null;
        let minDistance = Infinity;
        
        for (let i = 0; i < positionFilter.length; i++) {
            const cx = positionFilter[i]["cx"];
            const cy = positionFilter[i]["cy"];
            const distance = Math.sqrt(Math.pow(cx - mx, 2) + Math.pow(cy - my, 2));
            if (distance < minDistance) {
                minDistance = distance;
                closestCircle = i;
            }
        }

        return closestCircle;
    }

    function containsType(d,type) {
        if (d.type1 === type) {
            return true;
        }
        else if (d.type2 === type) {
            return true;
        }
        return false
    }

    function selectCircle(index) {
        if (!fullyLoaded) {return;}
        for (let i = 0; i < currCircles.length; i++) {
            if (i === index) {
                currCircles[i].setAttribute("r", 10);
            }
            else {
                currCircles[i].setAttribute("r", 2.5);
            }
        }
        
    }

    var allTypes = ["all", "normal", "fighting", "flying", "poison", "ground", "rock", "bug", "ghost", "steel", "fire", "water", "grass", "electric", "psychic", "ice", "dragon", "dark", "fairy"]

    $: d3.select(selectButt)
        .attr("id", "dropdown")
        .selectAll('myOptions')
            .data(allTypes)
        .enter()
            .append('option')
        .text(function (d) { return d; }) // text showed in the menu
        .attr("value", function (d) { return d; })

    $: d3.select(selectButt).on("change", function(d) {
        // recover the option that has been chosen
        var selectedOption = d3.select(this).property("value")
        // run the updateChart function with this selected option
        update(selectedOption)
    })

    function firstLoad() {
        if(fullyLoaded) { return; }
        for (let i = 0; i < data.length; i++) {
            positionFilter.push({
                cx: x(data[i].order),
                cy: y(data[i].bst),
            });
        }
        currCircles = Array.from(d3.select(circles).selectAll("circle"));
        fullyLoaded = true;
    }

</script>

<select bind:this={selectButt}></select>

<div class="pokemon-plot">
    <svg
    bind:this={svg}
    {width}
    {height}
    viewBox = "0 0 {width} {height}"
    style="max-width: 100%; height:auto;"
    >
    <g bind:this={gx} transform="translate(0, {height - marginBottom})"></g>
    <g bind:this={gy} transform="translate({marginLeft},0)"></g>

    <text class="axis-Title" text-anchor="end" x={width-marginRight} y={height-55}>ID</text>
    <text class="axis-Title" text-anchor="end" transform="rotate(-90)" x={marginTop-40} y={marginLeft-40}>Base Stat Total</text>

    <g bind:this={circles} stroke="#000" stroke-opacity="0.2">
        {#each data as d, i}
            <circle 
                key={d.order} 
                cx={x(d.order)} 
                cy={y(d.bst)} 
                fill={color[d.type1]} 
                r="2.5"
                />
            {#if i === data.length-1}
                {firstLoad()}
            {/if}
        {/each}
        
    </g>
    {#if tooltipPt}
        <g transform="translate({x(tooltipPt.order)},{y(tooltipPt.bst)})">
            <text font-weight="bold">
                {tooltipPt.name}
            </text>
        </g>
        <g transform="translate(200,100)">
            <text font-weight="bold">
                {tooltipPt.name}
            </text>
            <image xlink:href="{tooltipPt.sprite}" alt="{tooltipPt.name}">
        </g>
        <g transform="translate(300,120)">
            <text font-weight="bold">
                {#if tooltipPt.type2==="none"}
                    Type: {tooltipPt.type1}
                {:else}
                    Type: {tooltipPt.type1} {tooltipPt.type2}
                {/if}
            </text>
        </g>
        <g transform="translate(300,140)">
            <text font-weight="bold">
                Base Stat Total: {tooltipPt.bst}
            </text>
        </g>
        <g transform="translate(300,160)">
            <text font-weight="bold">
                ID: {tooltipPt.order}
            </text>
        </g>
    {/if}
    
    
    
    </svg>
    <p>
        All projects where I'm given free reign, I focus on whatever I'm interested in. 
        This makes projects more fun and so I put in a little more effort. 
        In this case, I've been playing a lot of <a href="https://pokerogue.net" target="_blank">PokéRogue</a>, a rogue-like Pokemon fan game you can play in your browser. 
        I tried using this dataset during my undergrad, but I had trouble utilizing APIs. Now is not that time.
    </p>
    <p>
        I worked on this alone, spending  maybe 12-15 hours over the course of the week. 
        Setting up new websites on GitHub will always be tricky for me, but I'm getting used to it. 
        At first I was going to make a radial sunburst chart with every Pokémon, but there was no bigger question to ask besides "Hey, look at these Pokémon!" 
        I stuck with the scatterplot since it makes it easy to find trends (and it helps that the professor used on as an example).<br><br>
    </p>
    <p>
        The first deviation from the original was API usage. I had to learn how to make API requests from <a href="https://pokeapi.co" target="_blank">PokéAPI</a> which was simple.
        The issue I faced in the past was this. Grabbing all of the Pokémon does not give you their stats. 
        It gives you their name and a link to make another API request for their stats. So what did I do? 
        A caveman approach of for loops making API requests for over 1,000 Pokémon. I could just save it to a csv and upload it. 
        But I left it as is since the point of an API is getting data when it updates.
    </p>
    <p>
        I was going to model base stat total compared to BMI, but Pokémon proportions are ridiculous. 
        Then compare base stat total to weight, sort of a "does bulk mean strength?"
        The graph for that looked awful since so many Pokémon are in low weight classes and then some Pokémon have ??? weight but high base stats which screwed with everything. 
        Eventually I settled on the question, "As more Pokémon were released, did the average Base Stat Total increase?" 
        A simple question at first, but I play competitive. Every generation of Pokémon release new ones that outclass older generations. 
        Articuno, a mythical Pokémon from Generation 1, is completely unused in high tiers of play. There are multiple reasons for this. 
        Newer Pokémon can have better base stat totals, move sets, base stat spreads, or some other gimmick. Base stat totals is more objective to calculate so I went with that.
    </p>
    <p>
        So I followed the lecture and made a scatterplot, added some axis titles, and then some more features. First was the sprites. 
        The APIs actually link to a GitHub hosting the sprites so using my fetching know-how, I grabbed those. 
        Next I wanted a filter types feature which was a little tricky to figure out. 
        When you select a type, all the circles are deleted and then all data containing that type is put into a filtered array which then redraws the circles.
    </p>
    <p>
        By this point we had the discussion section. They gave me suggestions like using the types for colors since the gradient I had originally served zero purpose. 
        They also encouraged me not to use the bisect and instead find the closest circle to the cursor. This was tough and alot of ChatGPT was used, but I actually came up with my own solution. 
        A separate array of coordinates of circles is made alongside the filtered data, and so you find the index of the coordinates closest to your mouse. 
        Then use that to find the data you want.
    </p>
    <p>
        A change I made was what the Pokémon were ordered by. The API has "id" and "order." 
        I initially used order which claimed to be the national dex order while alternate forms like Mega Evolutions were by their originals. 
        Then I found out that it wasn't actually correctly ordered so I switched to use id which was more in line, but lacked alternate forms. 
        So sadly we had to let go of base stat 1125 Eternamax-Eternatus.
    </p>
    <p>
        A huge bug I encountered was that when filtering for a type, it would jump between Pokémon circles and skip over some. 
        This took a while but I found out the html elements for creating the initial circles and position data array went off every type change. 
        Making it so there was position data for circles that weren't there. I made a boolean to only allow it to fire at the start and that solved it.
    </p>
    <p>
        Lastly were the fonts. The title is obviously inspired by the Pokémon logo. The text was meant to be like the GBA Pokémon text I grew up with. 
        I tried changing the axis and select button font, but after a couple hours, I'm moving on. <br><br>
    </p>
    <p>
        Overall it was fun to create a site that I wanted to do, though it did come at the same time as the midterm in DSC 256. 
        If I had more time, I would work on getting those fonts figured out. Add some color, patterns, or even images to the background. 
        Definitey color the title to be like the Pokémon logo more. Maybe implement a click function so you can compare Pokémon. 
        However, I am satisfied with and proud of this product.

    </p>
</div>

<style>
    @import url('https://fonts.cdnfonts.com/css/sf-pixelate');


    .axis-Title, text, p {
        font-family: 'SF Pixelate', sans-serif;
    }

    svg {
        text-align: center;
    }

    p {
    text-indent:50px;
    }

    

</style>