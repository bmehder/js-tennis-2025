<script lang="ts">
	type Player = 0 | 1

	type CurrentSetIndex = 0 | 1 | 2

	type Point = 0 | 15 | 30 | 40 | 'Ad'

	type Game = [Point, Point]

	type Set = [number, number]

	// Data
	const pointScoringMap = new Map<Point, Point>([
		[0, 15],
		[15, 30],
		[30, 40],
		[40, 0],
	])

	// State
	const point: Game = $state([0, 0])

	let currentSetIndex: CurrentSetIndex = $state(0)

	const sets: Set[] = $state([
		[5, 5],
		[0, 0],
		[0, 0],
	])

	const setWinners: Player[] = $state([])

	const winner = $derived.by(() => {
		if (setWinners.filter(x => x === 0).length === 2) {
			return 'Player 1'
		}
		if (setWinners.filter(x => x === 1).length === 2) {
			return 'Player 2'
		}
	})

	const isDeuce = $derived(point.every(score => score === 40))

	// State mutators
	const resetGame = () => {
		point[0] = 0
		point[1] = 0
	}

	const deuce = () => {
		point[0] = 40
		point[1] = 40
	}

	const scorePoint = (winner: Player) => {
		const loser = winner === 0 ? 1 : 0

		if (point[winner] === 'Ad') {
			resetGame()
			sets[currentSetIndex][winner]++
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
			sets[currentSetIndex][winner]++
			resetGame()
			if (
				(sets[currentSetIndex][winner] === 6 && sets[currentSetIndex][loser] < 5) ||
				(sets[currentSetIndex][winner] >= 7 &&
					sets[currentSetIndex][loser] < sets[currentSetIndex][winner] - 1)
			) {
				currentSetIndex++
				setWinners.push(winner)
			}
		}
	}

	// Event handlers
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

<p>{winner}</p>

<style>
	.scoreboard {
		display: flex;
		gap: 1rem;
	}
</style>
