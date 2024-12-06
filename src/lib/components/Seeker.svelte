<script lang="ts">
	import { Spring } from 'svelte/motion';

	const { icon, scale = 1 } = $props();

	let mousePos = $state({ x: 100, y: 100 });

	let alive = $state(false);

	let coords = new Spring(mousePos, { precision: 0, damping: 0.1, stiffness: 0.01 });
	let size = new Spring(1);
	let spin = new Spring(0);

	let rotate = $derived(spin.current * 360 + (coords.target.x - coords.current.x) * 0.1);

	function fire(e: MouseEvent) {
		if (alive) {
			return;
		}
		coords.set(
			{ x: globalThis.window.innerWidth / 2, y: globalThis.window.innerHeight },
			{ instant: true }
		);
		coords.target = { x: e.clientX, y: e.clientY };
		alive = true;
	}

	function die() {
		alive = false;
	}

	$effect(() => {
		if (!alive) {
			return;
		}

		if (
			Math.abs(coords.current.x - coords.target.x) < 20 &&
			Math.abs(coords.current.y - coords.target.y) < 20
		) {
			die();
		}
	});
</script>

<svg role="presentation">
	<g transform="translate({mousePos.x},{mousePos.y})">
		<circle cx="0" cy="0" r="5" fill="red" stroke="" class="laser-dot" />
	</g>
</svg>

{#if !alive}
	<div
		class="explosion absolute"
		style="top: {mousePos.y}px; left: {mousePos.x}px; transform: translate(-50%, -50%);"
	></div>
{/if}

<svg
	role="presentation"
	onmousemove={(e) => {
		mousePos.x = e.clientX;
		mousePos.y = e.clientY;
		coords.target = mousePos;
	}}
	onmousedown={(e) => {
		spin.target += 1;
	}}
	onmouseup={(e) => {
		fire(e);
	}}
>
	<g transform="translate({coords.current.x},{coords.current.y})">
		<g transform="scale({size.current}) rotate({rotate})">
			<g transform="translate(-20, -20) scale({scale})" fill="#f5e9d6" stroke="#322">
				{#if alive}
					{@render icon()}
				{/if}
			</g>
		</g>
	</g>
</svg>

<style>
	svg {
		position: fixed;
		left: 0;
		top: 0;
		width: 100%;
		height: 100%;
	}

	.laser-dot {
		filter: drop-shadow(0px 0px 5px red);
	}

	.explosion {
		width: 128px;
		height: 128px;
		background: url('/explosion.png');
		animation: explode 0.35s steps(6) forwards;
	}

	@keyframes explode {
		100% {
			background-position: top right;
		}
	}
</style>
