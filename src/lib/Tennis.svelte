<script lang="ts">
	// Types
	type Player = 0 | 1

	type CurrentSetIndex = 0 | 1 | 2

	type TiebreakPoint = [number, number]

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

	// Explicit State
	const point: Game = $state([0, 0])

	const tiebreakPoint: TiebreakPoint = $state([0, 0])

	let currentSetIndex: CurrentSetIndex = $state(0)

	const sets: Set[] = $state([
		[0, 0],
		[0, 0],
		[0, 0],
	])

	const setWinners: Player[] = $state([])

	let isTiebreak = $state(false)

	// Derived State
	const winner = $derived.by(() => {
		if (setWinners.filter(player => player === 0).length === 2) {
			return 'Player 1 Wins!'
		}
		if (setWinners.filter(player => player === 1).length === 2) {
			return 'Player 2 Wins!'
		}
	})

	const isWinner = $derived(!!winner)

	const isDeuce = $derived(point.every(score => score === 40))

	// State mutation functions
	const resetGame = () => {
		point[0] = 0
		point[1] = 0
		tiebreakPoint[0] = 0
		tiebreakPoint[1] = 0
	}

	const deuce = () => {
		point[0] = 40
		point[1] = 40
	}

	const scoreTiebreak = (winner: Player) => {
		const loser = winner === 0 ? 1 : 0
		const setScore = sets[currentSetIndex]

		tiebreakPoint[winner]++

		if (
			tiebreakPoint[winner] >= 7 &&
			tiebreakPoint[loser] < tiebreakPoint[winner] - 1
		) {
			setScore[winner]++
			currentSetIndex++
			setWinners.push(winner)
			resetGame()

			isTiebreak = false
		}
	}

	const scorePoint = (winner: Player) => {
		const loser = winner === 0 ? 1 : 0
		const setScore = sets[currentSetIndex]

		if (point[winner] === 'Ad') {
			setScore[winner]++
			resetGame()
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
		}

		if (
			(setScore[winner] === 6 && setScore[loser] < 5) ||
			(setScore[winner] >= 7 && setScore[loser] < setScore[winner] - 1)
		) {
			currentSetIndex++
			setWinners.push(winner)
		}

		if (setScore[winner] === 6 && setScore[loser] === 6) {
			isTiebreak = true
		}
	}

	// Event handlers
	const handleClick = (winner: Player) => {
		if (isTiebreak) {
			scoreTiebreak(winner)
		} else {
			scorePoint(winner)
		}
	}
</script>

<div class="app">
	<h1>JS Tennis 2025 (Svelte 5)</h1>
	<div class="scoreboard">
		{#each sets as set}
			<div>
				<div>{set[0]}</div>
				<div>{set[1]}</div>
			</div>
		{/each}
		{#if isTiebreak}
			<div class="point">
				<div>{tiebreakPoint[0]}</div>
				<div>{tiebreakPoint[1]}</div>
			</div>
		{:else}
			<div>
				<div class="point">
					{point[0]}
				</div>
				<div class="point">
					{point[1]}
				</div>
			</div>
		{/if}
	</div>

	{#if isWinner}
		<p>{winner}</p>
	{:else}
		<div class="buttons">
			<button onclick={() => handleClick(0)}> Player 1 </button>
			<button onclick={() => handleClick(1)}> Player 2 </button>
		</div>
	{/if}
</div>

<style>
	.app {
		min-height: 100dvh;
		display: grid;
		place-content: center;
		justify-items: center;
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
