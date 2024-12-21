<script lang="ts">
	type Player = 0 | 1

	type CurrentSet = 0 | 1 | 2

	type Point = 0 | 15 | 30 | 40 | 'Ad'

	type Game = [Point, Point]

	type Set = [number, number]

	const pointScoringMap = new Map<Point, Point>([
		[0, 15],
		[15, 30],
		[30, 40],
		[40, 0],
	])

	// State
	const point: Game = $state([0, 0])

	const currentSet: CurrentSet = $state(0)

	const sets: Set[] = $state([
		[0, 0],
		[0, 0],
		[0, 0],
	])

	const isDeuce = $derived(point.every(score => score === 40))

	const resetGame = () => {
		point[0] = 0
		point[1] = 0
	}

	const deuce = () => {
		point[0] = 40
		point[1] = 40
	}

	// State mutators
	const scorePoint = (winner: Player) => {
		const loser = winner === 0 ? 1 : 0

		if (point[winner] === 'Ad') {
			resetGame()
			sets[currentSet][winner]++
			return
		}

		if (point[loser] === 'Ad') {
			deuce()
			return
		}

		if (isDeuce) {
			point[winner] = 'Ad'
			return
		}

		point[winner] = pointScoringMap.get(point[winner]) as Point

		if (point[winner] === 0) {
			sets[currentSet][winner]++
			resetGame()
		}
	}

	const handleClick = (winner: Player) => {
		scorePoint(winner)
	}
</script>

<div class="scoreboard">
	{#each sets as set}
		<div>
			<div>{set[0]}</div>
			<div>{set[1]}</div>
		</div>
	{/each}

	<div>
		<div>
			{point[0]}
		</div>
		<div>
			{point[1]}
		</div>
	</div>
</div>

<button onclick={() => handleClick(0)}> Player 1 </button>
<button onclick={() => handleClick(1)}> Player 2 </button>

<style>
	.scoreboard {
		display: flex;
		gap: 1rem;
	}
</style>
