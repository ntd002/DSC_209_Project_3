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
    let selectedCircle = null;

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
        //.domain(d3.extent(data, (d) => d.order))
        .domain([0, 1200])
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
        
        // Create new data with the selection
        for (let i = 0; i < data.length; i++) {
                //handle if they contain the type
                if (selectedType ==="any") {
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
        createCircles();
    }

    function createCircles() {
        positionFilter = [];
        for (let i = 0; i < dataFilter.length; i++) {
            d3.select(circles).append("circle")
                .attr("id", "")
                .attr("key", dataFilter[i].order)
                .attr("cx", x(dataFilter[i].order))
                .attr("cy", y(dataFilter[i].bst))
                .attr("fill", color[dataFilter[i].type1])
                .attr("r", 2.5)
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

    var allTypes = ["any", "normal", "fighting", "flying", "poison", "ground", "rock", "bug", "ghost", "steel", "fire", "water", "grass", "electric", "psychic", "ice", "dragon", "dark", "fairy"]

    $: d3.select(selectButt)
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

    <text class="axis-Title" text-anchor="end" x={width-marginRight} y={height-55}>Order</text>
    <text class="axis-Title" text-anchor="end" transform="rotate(-90)" x={marginTop-40} y={marginLeft-40}>Base Stat Total</text>

    <g bind:this={circles} stroke="#000" stroke-opacity="0.2">
        {#each data as d, i}
            <circle 
                id=""
                key={d.order} 
                cx={x(d.order)} 
                cy={y(d.bst)} 
                fill={color[d.type1]} 
                r="2.5"
                />
            {positionFilter.push({
                cx: x(d.order),
                cy: y(d.bst),
            })}
            {#if i === data.length-1}
                {currCircles = Array.from(d3.select(circles).selectAll("circle"))}
                {fullyLoaded = true}
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
                Order: {tooltipPt.order}
            </text>
        </g>
    {/if}
    
    
    
    </svg>
</div>

<style>
    @import url('https://fonts.cdnfonts.com/css/pokemon-solid');

    .axis-Title {
        font-weight: bold;
    }

    svg {
        text-align: center;
    }

</style>