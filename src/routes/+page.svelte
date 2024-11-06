<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import PokeAPI from './PokeAPI.svelte';

    let data = [];

    onMount(async() => {
            //grab data
        const res = await fetch (
            'https://pokeapi.co/api/v2/pokemon?limit=100000&offset=0'
        );
        const poke = await res.json();

            //save into long dataset

        // let tempRes = await fetch(poke.results[2].url)
        // let tempPoke = await tempRes.json()
        
        for (let i = 0; i < poke.results.length; i++) {
            let tempRes = await fetch(poke.results[i].url)
            let tempPoke = await tempRes.json()
            
                //handle if they have a second type
            if (tempPoke.types.length === 1) {
                data.push({
                name: poke.results[i].name.split(' ').map(word => word.charAt(0).toUpperCase() + word.slice(1)).join(' '),
                bst: tempPoke.stats[0].base_stat + tempPoke.stats[1].base_stat + tempPoke.stats[2].base_stat + tempPoke.stats[3].base_stat + tempPoke.stats[4].base_stat + tempPoke.stats[5].base_stat, 
                type1: tempPoke.types[0].type.name,
                type2: 'none',
                order: tempPoke.order,
                weightG: tempPoke.weight,
                sprite: tempPoke.sprites.front_default,
                })
            }
            else {
                data.push({
                name: poke.results[i].name.split(' ').map(word => word.charAt(0).toUpperCase() + word.slice(1)).join(' '),
                bst: tempPoke.stats[0].base_stat + tempPoke.stats[1].base_stat + tempPoke.stats[2].base_stat + tempPoke.stats[3].base_stat + tempPoke.stats[4].base_stat + tempPoke.stats[5].base_stat, 
                type1: tempPoke.types[0].type.name,
                type2: tempPoke.types[1].type.name,
                order: tempPoke.order,
                weightG: tempPoke.weight,
                sprite: tempPoke.sprites.front_default,
                })
            }
        }
        
        data = data;
    })
 
</script>

<main>
    <h1>Pokémon Base Stats and Weight Trends</h1>
    <PokeAPI {data}/>
</main>

