<script>
import { getTable } from "@src/data/priconnedb"
import { getUnitIdBase, getUnitType, getUnlockedUnits, getSummonUnits } from "@src/logic/unit";
import { getUnitImg } from "@src/logic/ui"
import { sortByAttr } from "@src/utils"

export let unitId = -1;
export let selectCallback;

let UNLOCKED_UNITS = getUnlockedUnits();
let SUMMON_UNITS = getSummonUnits();

let bossBaseIds = [];
let enemyBaseIds = []
let bosses = getTable("enemy_parameter").filter(function(enemy) {
	if (getUnitType(enemy.unit_id) === "boss") {
		if (enemy.unit_id === 390100) return false; // "Omniscient Kaiser", weird data
		if (bossBaseIds.indexOf(getUnitIdBase(enemy.unit_id)) === -1) {
			bossBaseIds.push(getUnitIdBase(enemy.unit_id));
			return true;
		}
	}
	return false;
}).sort(sortByAttr("name"));
let enemies = getTable("enemy_parameter").filter(function(enemy) {
	if (getUnitType(enemy.unit_id) === "enemy" || getUnitType(enemy.unit_id) === "shadow") {
		if (enemyBaseIds.indexOf(getUnitIdBase(enemy.unit_id)) === -1) {
			enemyBaseIds.push(getUnitIdBase(enemy.unit_id));
			return true;
		}
	}
	return false;
}).sort(sortByAttr("name"));

let options = {
	bosses: bosses.map(function(enemy) {
		return {
			id: enemy.unit_id,
			name: enemy.name
		}
	}),
	enemies: enemies.map(function(enemy) {
		return {
			id: enemy.unit_id,
			name: enemy.name
		}
	}),
	characters: UNLOCKED_UNITS.concat(SUMMON_UNITS).map(function(character) {
		return {
			id: character.unit_id,
			name: character.unit_name
		}
	})
}

let isSelecting = false;
let selectTab = getInitialSelectTab(unitId);

function getInitialSelectTab(unitId) {
	let unitType = getUnitType(unitId);
	if (unitType === "boss") return "bosses";
	else if (unitType === "enemy" || unitType === "shadow") return "enemies";
	else return "bosses";
}

$: selectRows = getUnitSelectionRows(selectTab);

function getUnitSelectionRows(selectTab) {
	let rows = [];
	let selectionColumns = 6;
	if (selectTab === "bosses") selectionColumns = 4;
	options[selectTab].forEach(function(unit, i) {
		if (i % selectionColumns === 0) {
			rows.push([]);
		}
		rows.slice(-1)[0].push(unit);
	});
	return rows;
}

function switchTab(tabName) {
	return function() {
		selectTab = tabName;
	}
}

function createSelectHandler(id) {
	return function() {
		selectCallback(id);
	}
}
</script>

<div class="unit-list-header">
	<h2>Select a unit</h2>
	<div class="unit-tabs-wrap">
		<table class="unit-tabs"><tr>
<!--			<td><div class="tab" class:selected={selectTab === "characters"} on:click={switchTab("characters")}>Characters</div></td>-->
			<td><div class="tab" class:selected={selectTab === "enemies"} on:click={switchTab("enemies")}>Enemies</div></td>
			<td><div class="tab" class:selected={selectTab === "bosses"} on:click={switchTab("bosses")}>Bosses</div></td>
		</tr></table>
	</div>
	<!-- <div class="tabs-shadow"></div> -->
</div>
<div class="unit-list">
	<table class="units">
		{#each selectRows as selectRow}
		<tr>
			{#each selectRow as unit}
			<td class="select-unit" class:large-icon={selectTab==="bosses"} on:click={createSelectHandler(unit.id)}>
				<img class="select-icon" class:large-icon={selectTab==="bosses"} src={getUnitImg(unit.id, { usePrefabId: true })} /><br />
				<span class="select-unit-name">{unit.name}</span>
			</td>
			{/each}
		</tr>
		{/each}
	</table>
</div>

<style>
h2 {
/*	position: sticky;
	background-color: white;*/
}

table.units {
	border-spacing: 8px;
	margin: 0 auto;
}

table.unit-tabs {
	position: absolute;
	top:0; left: 0; right: 0;
	width: 100%;
	height: 40px;
	overflow-y: visible;
	table-layout: fixed;
	font-size: 14pt;
	border-spacing: 0;
	padding: 0; margin: 0;
}

div.unit-tabs-wrap {
	position: absolute;
	right: 0; left: 0; bottom: 3px;
	height: 40px;
	z-index: 1000;
	overflow-y: visible;
	background-color: #e7eef8;
	box-shadow: 0 0 7px #163b5a;
	/* You would not believe how fucking mad I got trying to get the tab border to
	show on top of the box shadow. Had to avoid using a background color on the tabs */
}

table.unit-tabs td {
	height: 40px;
	overflow-y: visible;
	padding: 0; margin: 0;
}

div.tab {
	color: #303b5a;
	height: 40px;
	overflow-y: visible;
	line-height: 40px;
	border-bottom: 3px solid transparent;
	cursor: pointer;
}

/*div.tabs-shadow {
	position: absolute;
	bottom: 3px; left: 0; right: 0;
	height: 10px;
	box-shadow: 0 0 7px #163b5a;
}*/

div.tab.selected {
	border-bottom-color: #7080af;
}

div.tab:hover {
	color: #ef7485;
	border-bottom-color: #ef7485;
}

img.select-icon {
	width: 76px;
	height: 76px;
}
img.select-icon.large-icon {
	width: 115px;
	height: 115px;
}

div.unit-list-header {
	position: absolute;
	text-align: center;
	top: 0;	left: 0; right: 0;
	height: 113px;
}

div.unit-list-header h2 {
	position: absolute;
	top: 0;	left: 0; right: 0; bottom: 43px;
	margin: 0;
	line-height: 70px;
	z-index: 2000;
	background-color: white;
}

div.unit-list {
	position: absolute;
	padding: 10px 0;
	top: 110px; left: 0; right: 0; bottom :0;
	overflow-y: auto;
}

td.select-unit {
	cursor: pointer;
	color: #303b5a;
	text-align: center;
	max-width: 76px;
	height: 124px;
}
td.select-unit.large-icon {
	max-width: 115px;
	height: 163px;
}
td.select-unit:hover {
	opacity: 0.8;
}

span.select-unit-name {
	padding-bottom: 5px;
}
</style>