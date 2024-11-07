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
    let circles;
    let selectButt;

    $: x = d3  
        .scaleLinear()
        .domain(d3.extent(data, (d) => d.order))
        .range([marginLeft, width - marginRight]);
    $: y = d3  
        .scaleLinear()
        .domain(d3.extent(data, (d) => d.bst))
        .nice()
        .range([height - marginBottom, marginTop]);

    //lowest pokemon bst is 175, highest is 1125
    $: color = d3
        .scaleSequential()
        .domain([150, 1200])
        .interpolator(d3.interpolateRdYlBu)
    
    //axis
    $: d3.select(gx).call(d3.axisBottom(x));
    $: d3.select(gy).call(d3.axisLeft(y));

    //tooltip
    const bisect = d3.bisector((d) => d.order).center;
    let tooltipPt = null;
    //const quadtree = d3.quadtree(data, d => d.bst, d => d.order);
    function onPointerMove(event) {
        const [xValue, yValue] = d3.pointer(event);
        const i = bisect(dataFilter, x.invert(xValue))
        tooltipPt = dataFilter[i];

        //d3.select(circles).selectAll("*").remove();
        //const closestPoint = quadtree.find(xValue, yValue);
        //console.log(closestPoint)
        //tooltipPt = quadtree.find(yValue, xValue);
    }

    $: d3.select(svg)
    .on('pointerenter pointermove', onPointerMove)

    //filtering
    function update(selectedType) {
        dataFilter = []
        // Create new data with the selection
        if (selectedType === "any") {
            dataFilter = data
        }
        else {
            for (let i = 0; i < data.length; i++) {
                //handle if they contain the type
                if (containsType(data[i], selectedType)){
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
        }
        
        //delete all circles
        d3.select(circles).selectAll("*").remove();

        //replace them
        for (let i = 0; i < dataFilter.length; i++) {
            d3.select(circles).append("circle")
                .attr("key", dataFilter[i].order)
                .attr("cx", x(dataFilter[i].order))
                .attr("cy", y(dataFilter[i].bst))
                .attr("fill", color(dataFilter[i].bst))
                .attr("r", 2.5)
        }
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

    <text class="axis-Title" text-anchor="end" x={width-marginRight} y={height-55}>Base Stat Total</text>
    <text class="axis-Title" text-anchor="end" transform="rotate(-90)" x={marginTop-40} y={marginLeft-40}>Order</text>

    <g bind:this={circles} stroke="#000" stroke-opacity="0.2">
        {#each data as d}
            <circle 
                key={d.order} 
                cx={x(d.order)} 
                cy={y(d.bst)} 
                fill={color(d.bst)} 
                r="2.5"/>
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
    
    
    <style>
        .axisTitle {
            font-weight: bold;
        }
    
    </style>
    </svg>
</div>

