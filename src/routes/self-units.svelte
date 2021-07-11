<script>
import { stores } from "@sapper/app";
const { page } = stores();
import { savedSelfUnit, baseUnitConfig } from "@src/settings.js"
import { isValidUnitConfiguration, getBondStats } from "@src/logic/unit";
import Swal from 'sweetalert2'
import UnitCard from "@src/components/UnitCard.svelte";

async function updateDeck() {
    let postData = JSON.parse(JSON.stringify($savedSelfUnit));
    for (let unitConfig of postData) {
        unitConfig['prid_bond_stats'] = getBondStats(unitConfig)
    }
    // await fetch('https://107918c1-ced4-4f15-98b0-d9a169ca70fc.mock.pstmn.io', {
    //     method: 'POST',
    //     body: JSON.stringify(postData)
    // })
    await fetch('http://localhost:3333/update/self', {
        method: 'POST',
        body: JSON.stringify(postData)
    });
    await Swal.fire({
        title: 'Self arena deck updated!',
        icon: 'success'
    })

}
</script>

<div class="button green" on:click={updateDeck}>Update Prid Self Deck</div>
<br>
{#each Array(5) as _, i}
    <UnitCard bind:unit={$savedSelfUnit[i]} enemyOnly={false}/>
    {#if i !== 4}
        <hr>
    {/if}
{/each}
