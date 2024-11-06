<script>
    import * as d3 from 'd3';
    export let data;
    const width = 923;
    const height = 600;
    const marginTop = 20;
    const marginBottom = 30;
    const marginRight = 30;
    const marginLeft = 40;

    $: x = d3  
        .scaleLinear()
        .domain(d3.extent(data, (d) => d.weightG))
        .range([marginLeft, width - marginRight]);
    $: y = d3  
        .scaleLinear()
        .domain(d3.extent(data, (d) => d.bst))
        .nice()
        .range([height - marginBottom, marginTop]);

    $: console.log(data.length);
</script>

<div class="pokemon-plot">
    <svg
    {width}
    {height}
    viewBox = "0 0 {width} {height}"
    style="max-width: 100%; height:auto;"
    >
    <g stroke="#000" stroke-opacity="0.2">
        {#each data as d, i}
            <circle key={i} cx={x(d.weightG)} cy={y(d.bst)} fill={'blue'} r="2.5"/>
        {/each}
    </g>

    </svg>
</div>