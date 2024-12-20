<script lang="ts">
	type Player = 0 | 1

	type Point = 0 | 15 | 30 | 40 | 'Ad'

	type Game = [Point, Point]

	type Set = [Game, Game]

	type Match = {
		set1: Game
		set2: Game
		set3: Game
	}

	const pointScoringMap = new Map<Point, Point>([
		[0, 15],
		[15, 30],
		[30, 40],
		[40, 0],
	])

	const points: Game = $state([0, 0])

	const resetGame = () => {
		points[0] = 0
		points[1] = 0
	}

	const scorePoint = (winner: Player) => {
		const loser = winner === 0 ? 1 : 0

		if (points[winner] === 'Ad') {
			resetGame()
			return
		}

		if (points[loser] === 'Ad') {
			points[winner] = 40
			points[loser] = 40
			return
		}

		if (points.every(point => point === 40)) {
			points[winner] = 'Ad'
			return
		}

		points[winner] = pointScoringMap.get(points[winner]) as Point
	}

	const handleClick = (player: Player) => {
		scorePoint(player)
	}
</script>

<div>
	{points[0]}
</div>

<div>
	{points[1]}
</div>

<button onclick={() => handleClick(0)}> Player 1 </button>
<button onclick={() => handleClick(1)}> Player 2 </button>
