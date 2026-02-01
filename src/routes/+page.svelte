<script lang="ts">
	import CellFiller from '$lib/components/CellFiller.svelte';
	import { type Session } from '$lib/types/session';
	import { RED, BLUE } from '$lib/consts/colors';

	const cells = Array.from({ length: 6 }, () => Array.from({ length: 20 }, () => null));

	const data = Array.from({ length: 6 }, () =>
		Array.from({ length: 20 }, () => {
			return {
				session: { player: 0, banker: 0, playerPair: false, bankerPair: false },
				enable: false,
				previous: [-1, -1]
			};
		})
	);

	function appendSession(session: Session) {
		const playerWon = session.player >= session.banker;

		// find index where next session is empty
		let i;
		for (i = 0; i <= cells[0].length; i++) {
			if (i === cells[0].length) {
				break;
			}

			if (i === 0 && !data[0][0].enable) {
				break;
			}

			const iPlayerWon = data[0][i].session.player >= data[0][i].session.banker;

			if (!data[0][i + 1].enable) {
				if (playerWon === iPlayerWon) {
					break;
				} else {
					i++;
					break;
				}
			}
		}

		if (i === cells[0].length) {
			for (let j = 0; j < data.length; j++) {
				for (let k = 0; k < data[j].length; k++) {
					const datum = data[j][k + 1];
					data[j][k] = data[j][k + 1] || {
						session: { player: 0, banker: 0, playerPair: false, bankerPair: false },
						enable: false,
						previous: [datum.previous[0], datum.previous[1] - 1]
					};
				}
			}
			i--;
		}

		// follow baccarat shoe board rules to place session
		const [row, col, pr, pc] = getScorePosition(0, i);
		data[row][col] = { session, enable: true, previous: [pr, pc] };

		dataUpdateStack.push({ row, col });
	}

	function getScorePosition(row: number, col: number): number[] {
		if (!data[row][col].enable) {
			return [row, col];
		}

		const playerWon = data[row][col].session.player >= data[row][col].session.banker;
		let prevRow = row;
		let prevCol = col;
		while (true) {
			if (!data[row][col].enable) {
				return [row, col, prevRow, prevCol];
			}

			prevRow = row;
			prevCol = col;

			if (row + 1 < data.length)
				if (data[row + 1][col].enable)
					if (data[row + 1][col].previous[0] === row && data[row + 1][col].previous[1] === col)
						row++;
					else col++;
				else if (
					col - 1 >= 0 &&
					data[row + 1][col - 1].enable &&
					!(
						data[row + 1][col - 1].previous[0] === row || data[row + 1][col - 1].previous[1] === col
					) &&
					data[row + 1][col - 1].session.player >= data[row + 1][col - 1].session.banker ===
						playerWon
				)
					col++;
				else row++;
			else col++;
		}
	}

	function confirmInput() {
		const inputs = Array.from({ length: 6 }, (_, idx) => {
			const input = document.getElementById(`digit-${idx}`) as HTMLInputElement;
			const value = input.value.toUpperCase();

			if (value === 'A') {
				return 1;
			} else if (value === 'J') {
				return 0;
			} else if (value === 'Q') {
				return 0;
			} else if (value === 'K') {
				return 0;
			} else {
				return value;
			}
		});

		const session: Session = {
			player: (parseInt(inputs[0]) + parseInt(inputs[2]) + parseInt(inputs[4])) % 10,
			banker: (parseInt(inputs[1]) + parseInt(inputs[3]) + parseInt(inputs[5])) % 10,
			playerPair: inputs[0] === inputs[2],
			bankerPair: inputs[1] === inputs[3]
		};

		appendSession(session);

		// Clear inputs
		inputs.forEach((_, idx) => {
			const input = document.getElementById(`digit-${idx}`) as HTMLInputElement;
			input.value = '';
		});

		// Focus first input if computer
		if ('ontouchstart' in window === false) {
			const firstInput = document.getElementById('digit-0') as HTMLInputElement;
			firstInput.focus();
		}
	}

	function onDigitInput(event: InputEvent) {
		const input = event.target as HTMLInputElement;
		const value = input.value;
		const nextId = input.dataset.next;

		console.log(value);
		if (!value.match(/^[0-9ajqkAJQK]$/)) {
			input.value = '';
			return;
		}

		if (value.length === 1 && nextId) {
			const nextInput = document.getElementById(nextId) as HTMLInputElement;

			input.value = value.toUpperCase();

			if (nextInput) {
				nextInput.focus();
			}
		}
	}

	function onKeyDown(event: InputEvent) {
		const input = event.target as HTMLInputElement;
		const value = input.value;

		if (event.key === 'Backspace') {
			const digit = input.id.match(/digit-(\d+)/);
			if (!digit) {
				return;
			}

			const idx = parseInt(digit[1]);
			if (idx <= 0 || value.length !== 0) {
				return;
			}

			const prevInput = document.getElementById(`digit-${idx - 1}`) as HTMLInputElement;
			if (!prevInput) {
				return;
			}

			prevInput.focus();
			return;
		}

		if (event.key === 'Enter') {
			confirmInput();
		}
	}

	const KEYS = 'A234567890JQK';

	function clickKey(key: string) {
		return () => {
			const inputs = Array.from({ length: 6 }, (_, idx) => {
				return document.getElementById(`digit-${idx}`) as HTMLInputElement;
			});

			for (const input of inputs) {
				if (input.value === '') {
					input.value = key;
					break;
				}
			}
		};
	}

	function cancelInput() {
		const inputs = Array.from({ length: 6 }, (_, idx) => {
			return document.getElementById(`digit-${idx}`) as HTMLInputElement;
		});

		for (const input of inputs) {
			input.value = '';
		}

		const firstInput = document.getElementById('digit-0') as HTMLInputElement;
		firstInput.focus();
	}

	const dataUpdateStack = [];

	function undoLast() {
		const last = dataUpdateStack.pop();
		if (!last) {
			return;
		}
		const { row, col } = last;
		data[row][col] = {
			session: { player: 0, banker: 0, playerPair: false, bankerPair: false },
			enable: false,
			previous: [-1, -1]
		};
	}
</script>

<div class="container">
	<div class="board">
		{#each Array(6) as _, index}
			<div class="row">
				{#each Array(20) as _, jndex}
					<div class="cell" id="cell-{index}-{jndex}" bind:this={cells[index][jndex]}>
						<CellFiller data={data[index][jndex].session} enable={data[index][jndex].enable}
						></CellFiller>
					</div>
				{/each}
			</div>
		{/each}
	</div>
	<div class="input">
		{#each Array(6) as _, idx}
			<input
				type="text"
				id={`digit-${idx}`}
				maxlength="1"
				data-next={`digit-${idx + 1}`}
				class="input-digit"
				oninput={onDigitInput}
				onkeydown={onKeyDown}
			/>
		{/each}
	</div>
	<div class="keyboard">
		<button onclick={undoLast} class="key">Undo</button>
		{#each KEYS.split('') as key}
			<button onclick={clickKey(key)} class="key">{key}</button>
		{/each}
		<button onclick={cancelInput} class="key">Clear</button>
		<button onclick={confirmInput} class="key">Enter</button>
	</div>
</div>

<style>
	.container {
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		height: 100vh;
		background-color: #f0f0f0;
	}

	.row {
		display: flex;
	}

	.cell {
		background-color: white;
		width: 60px;
		height: 60px;
		margin: 4px;
		border-radius: 8px;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.input {
		margin-top: 20px;
	}

	.input-digit {
		width: 60px;
		height: 60px;
		margin: 4px;
		padding: 0;
		font-size: 24px;
		text-align: center;
		border-radius: 4px;
		border: none;
	}

	.key {
		min-width: 40px;
		height: 40px;
		margin: 2px;
		font-size: 18px;
		border-radius: 4px;
		border: none;
		background-color: white;
		cursor: pointer;
	}
</style>
