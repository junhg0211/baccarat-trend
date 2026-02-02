<script lang="ts">
	import { type Session } from '$lib/types/session';
	import { RED, BLUE, YELLOW, GREEN } from '$lib/consts/colors';
	import { onMount } from 'svelte';

	export let data: Session = {
		player: 0,
		banker: 0,
		playerPair: false,
		bankerPair: false
	};
	export let enable: boolean = false;

	function isNatural() {
		return data.player >= 8 || data.banker >= 8;
	}

	function isTie() {
		return data.player === data.banker;
	}

	function getColor() {
		return data.player >= data.banker ? RED : BLUE;
	}

	let circleSvg: SVGCircleElement = null;
</script>

<div class="filler">
	{#if enable}
		<svg width="100%" height="100%" viewBox="0 0 60 60" xmlns="http://www.w3.org/2000/svg">
			<circle
				cx="30px"
				cy="30px"
				r="20px"
				fill="transparent"
				stroke={getColor()}
				stroke-width="10px"
				bind:this={circleSvg}
			/>
			{#if isNatural()}
				<circle cx="50%" cy="50%" r="10px" fill={YELLOW} />
			{/if}
			{#if isTie()}
				<line
					x1="38px"
					y1="22px"
					x2="50px"
					y2="10px"
					stroke={GREEN}
					stroke-width="8px"
					stroke-linecap="round"
				/>
			{/if}
			{#if data.playerPair}
				<circle cx="calc(50% - sqrt(2) * 10px)" cy="calc(50% - sqrt(2) * 10px)" r="10px" fill={RED}
				></circle>
			{/if}
			{#if data.bankerPair}
				<circle cx="calc(50% + sqrt(2) * 10px)" cy="calc(50% + sqrt(2) * 10px)" r="10px" fill={BLUE}
				></circle>
			{/if}
		</svg>
	{/if}
</div>
