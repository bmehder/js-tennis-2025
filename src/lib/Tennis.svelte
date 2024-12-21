<script lang="ts">
	// Types
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
			return 'Player 1 Wins!'
		}
		if (setWinners.filter(x => x === 1).length === 2) {
			return 'Player 2 Wins!'
		}
	})

	const isDisabled = $derived(!!winner)

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
		const setScore = sets[currentSetIndex]

		if (point[winner] === 'Ad') {
			resetGame()
			setScore[winner]++
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
			setScore[winner]++
			resetGame()
			if (
				(setScore[winner] === 6 && setScore[loser] < 5) ||
				(setScore[winner] >= 7 && setScore[loser] < setScore[winner] - 1)
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

<div class="app">
	<div class="scoreboard">
		{#each sets as set}
			<div>
				<div>{set[0]}</div>
				<div>{set[1]}</div>
			</div>
		{/each}
		<div>
			<div class="point">
				{point[0]}
			</div>
			<div class="point">
				{point[1]}
			</div>
		</div>
	</div>

	{#snippet button(winner: Player, playerNumber: number)}
		<button disabled={isDisabled} onclick={() => handleClick(winner)}>
			Player {playerNumber}
		</button>
	{/snippet}

	<div class="buttons">
		{@render button(0, 1)}
		{@render button(1, 2)}
	</div>

	<p>{winner}</p>
</div>

<style>
	.app {
		display: grid;
		place-items: center;
		gap: 1rem;
	}

	.scoreboard {
		display: flex;
		gap: 1rem;
		padding-inline-start: 1ch;
		font-size: 3rem;
	}

	.buttons {
		display: flex;
		gap: 1rem;
	}

	.point {
		width: 2ch;
		color: var(--accent);
	}
</style>
