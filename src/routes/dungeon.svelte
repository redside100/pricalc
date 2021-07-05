<script>
    import { stores } from "@sapper/app";
    const { page } = stores();
    import { savedDungeonUnit } from "@src/settings.js"
    import Swal from 'sweetalert2'
    import UnitCard from "@src/components/UnitCard.svelte";

    async function updateDungeon() {
        if ($savedDungeonUnit['id'] === -1) {
            await Swal.fire({
                title: 'Dungeon unit cannot be empty.',
                icon: 'error'
            });
            return
        }
        await fetch('http://localhost:3333/update/dungeon', {
            method: 'POST',
            body: JSON.stringify($savedDungeonUnit)
        });
        await Swal.fire({
            title: 'Dungeon unit updated!',
            icon: 'success'
        })

    }
</script>

<div class="button green" on:click={updateDungeon}>Update Dungeon Unit</div>
<br>
<UnitCard bind:unit={$savedDungeonUnit} enemyOnly={true}/>