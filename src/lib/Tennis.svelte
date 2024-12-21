<script lang="ts">
	// Types
	type Player = 0 | 1

	type Point = 0 | 15 | 30 | 40 | 'Ad'

	type Tiebreak = [number, number]

	type Game = [Point, Point]

	type Set = [number, number]

	type CurrentSet = 0 | 1 | 2

	// Data
	const pointScoringMap = new Map<Point, Point>([
		[0, 15],
		[15, 30],
		[30, 40],
		[40, 0],
	])

	// Explicit State
	const game: Game = $state([0, 0])

	const tiebreak: Tiebreak = $state([0, 0])

	let currentSet: CurrentSet = $state(0)

	const sets: Set[] = $state([
		[0, 0],
		[0, 0],
		[0, 0],
	])

	const setWinners: Player[] = $state([])

	let isTiebreak = $state(false)

	// Derived State
	const setScore = $derived(sets[currentSet])

	const winner = $derived.by(() => {
		if (setWinners.filter(player => player === 0).length === 2) {
			return 'Player 1 Wins!'
		}
		if (setWinners.filter(player => player === 1).length === 2) {
			return 'Player 2 Wins!'
		}
	})

	const isWinner = $derived(Boolean(winner))

	const isDeuce = $derived(game.every(point => point === 40))

	// Helper functions
	const getLoser = (winner: Player) => (winner === 0 ? 1 : 0)

	const isSetComplete = (winner: Player, loser: Player) =>
		(setScore[winner] === 6 && setScore[loser] < 5) ||
		(setScore[winner] >= 7 && setScore[loser] < setScore[winner] - 1)

	const isSetWinner = (set: CurrentSet) => {
		if (setWinners[set] != undefined) {
			return sets[set][0] > sets[set][1] ? 0 : 1
		}
	}

	const isTiebreakComplete = (winner: Player, loser: Player) =>
		tiebreak[winner] >= 7 && tiebreak[loser] < tiebreak[winner] - 1

	const is6GamesAll = (winner: Player, loser: Player) =>
		setScore[winner] === 6 && setScore[loser] === 6

	// State mutation functions
	const resetGame = () => {
		game[0] = 0
		game[1] = 0
		tiebreak[0] = 0
		tiebreak[1] = 0
	}

	const resetMatch = () => {
		resetGame()
		currentSet = 0
		sets[0] = [0, 0]
		sets[1] = [0, 0]
		sets[2] = [0, 0]
		setWinners.length = 0
		isTiebreak = false
	}

	const deuce = () => {
		game[0] = 40
		game[1] = 40
	}

	const scoreTiebreak = (winner: Player) => {
		const loser = getLoser(winner)

		tiebreak[winner]++

		if (isTiebreakComplete(winner, loser)) {
			setScore[winner]++
			currentSet++
			setWinners.push(winner)
			isTiebreak = false
			resetGame()
		}
	}

	const scorePoint = (winner: Player) => {
		const loser = getLoser(winner)

		if (game[winner] === 'Ad') {
			setScore[winner]++
			resetGame()
			return
		}

		if (game[loser] === 'Ad') {
			deuce()
			return
		}

		if (isDeuce) {
			game[winner] = 'Ad'
			return
		}

		game[winner] = pointScoringMap.get(game[winner]) as Point

		if (game[winner] === 0) {
			setScore[winner]++
			resetGame()
		}

		if (isSetComplete(winner, loser)) {
			currentSet++
			setWinners.push(winner)
		}

		if (is6GamesAll(winner, loser)) {
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

	const restart = () => {
		confirm('Are you sure you want to create a new match?') && resetMatch()
	}
</script>

<div class="app">
	<div class="title">JS Tennis 2025 (Svelte 5)</div>
	<div class="scoreboard">
		<div>
			<div>&nbsp;</div>
			<div contenteditable>Player 1</div>
			<div contenteditable>Player 2</div>
		</div>
		{#each sets as set, i}
			<div class="sets">
				<div class="label">Set {i + 1}</div>
				<div class:accent={isSetWinner(i as CurrentSet) === 0}>{set[0]}</div>
				<div class:accent={isSetWinner(i as CurrentSet) === 1}>{set[1]}</div>
			</div>
		{/each}
		{#if isTiebreak}
			<div class="points">
				<div class="label">Point</div>
				<div class="accent">{tiebreak[0]}</div>
				<div class="accent">{tiebreak[1]}</div>
			</div>
		{:else}
			<div class="points">
				<div class="label">Point</div>
				<div class="accent">
					{game[0]}
				</div>
				<div class="accent">
					{game[1]}
				</div>
			</div>
		{/if}
	</div>

	{#if isWinner}
		<div class="winner">{winner}</div>
		<button onclick={() => restart()}>Create New Match</button>
	{:else}
		<div class="buttons">
			<button onclick={() => handleClick(0)}> Player 1 </button>
			<button onclick={() => handleClick(1)}> Player 2 </button>
		</div>
	{/if}
</div>

<style>
	.app {
		display: grid;
		align-content: center;
		justify-items: center;
		gap: var(--size-3);
		padding-inline-start: var(--size-3);
	}

	.title,
	.winner {
		font-size: var(--size-2);
		font-weight: bold;
	}

	.scoreboard {
		display: flex;
		gap: var(--size-3);
		padding: var(--size-3);
		font-size: var(--size-2);
		border: 1px solid;
	}

	.points,
	.sets {
		justify-items: center;
	}

	.buttons {
		display: flex;
		gap: var(--size);
	}

	button {
		border: none;
		background-color: var(--accent);
		color: black;
		font-size: var(--size-1-5);
	}

	.accent {
		color: var(--accent);
	}
</style>
