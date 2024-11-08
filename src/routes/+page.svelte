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
        
        for (let i = 0; i < poke.results.length; i++) {
            let tempRes = await fetch(poke.results[i].url)
            let tempPoke = await tempRes.json()

                //ignore unordered pokemon
            if (tempPoke.order === -1) {
                //console.log(poke.results[i].name)
            }
            else {
                //handle if they have a second type
                if (tempPoke.types.length === 1) {
                    data.push({
                    name: poke.results[i].name,
                    bst: tempPoke.stats[0].base_stat + tempPoke.stats[1].base_stat + tempPoke.stats[2].base_stat + tempPoke.stats[3].base_stat + tempPoke.stats[4].base_stat + tempPoke.stats[5].base_stat, 
                    type1: tempPoke.types[0].type.name,
                    type2: 'none',
                    order: tempPoke.order,
                    sprite: tempPoke.sprites.front_default,
                    })
                }
                else {
                    data.push({
                    name: poke.results[i].name,
                    bst: tempPoke.stats[0].base_stat + tempPoke.stats[1].base_stat + tempPoke.stats[2].base_stat + tempPoke.stats[3].base_stat + tempPoke.stats[4].base_stat + tempPoke.stats[5].base_stat, 
                    type1: tempPoke.types[0].type.name,
                    type2: tempPoke.types[1].type.name,
                    order: tempPoke.order,
                    sprite: tempPoke.sprites.front_default,
                    })
                }
            }
            
                
        }
        
        data = data;
    })
 
</script>

<main>
    <h1>Pokémon Base Stats and National Dex Trend</h1>
    <PokeAPI {data}/>
</main>

<style>
    @import url('https://fonts.cdnfonts.com/css/pokemon-solid');
    h1 {
        font-family: 'Pokemon Solid', sans-serif;
        text-align: center;
    }
</style>

