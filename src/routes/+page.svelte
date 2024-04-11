<script lang="ts">
	import { linear } from "svelte/easing";
	import { writable } from "svelte/store";
	import CanvasOverlay from "../component/canvas-overlay.svelte";
	import KeyDetector from "../component/key-detector.svelte";
	import KeyPad from "../component/key-pad.svelte";
	import Slide_1 from "../slides/slide_1.svelte";

	const animationDuration = 500;
	const slides = [
		["A", 0, 0, "B"],
		["AD1", "AR1", "AR2", "AR3"],
		["AD2", "AD1R1", "AD1R2", "AD1R3"],
		[0, "AD2R1", 0, "AD2R3"]
	];

	const coord = writable<{
		x: number;
		y: number;
		lastPress: "up" | "down" | "left" | "right";
	}>({ x: 0, y: 0, lastPress: "up" });

	function skale(e) {
		return {
			duration: 1000,
			easing: linear,
			css: (t: number) => `
        transform: scale(${t});
        top: 0;
        left: 0; 
        opacity: ${t};
      `
		};
	}

	function slyde(node: Element, { lastPress }: { lastPress: "up" | "down" | "left" | "right" }) {
		const base = {
			duration: animationDuration,
			easing: linear
		};

		if (lastPress == "up") {
			return {
				...base,
				css: (t: number) => `
        top: -100vh;
        transform: translateY(${100 * t}%);
        
      `
			};
		} else if (lastPress == "down") {
			return {
				...base,
				css: (t: number) => `
        top: 100vh;
        transform: translateY(${-100 * t}%);
      `
			};
		} else if (lastPress == "left") {
			return {
				...base,
				css: (t: number) => `
        left: -100vw;
        transform: translateX(${100 * t}%);
      `
			};
		} else if (lastPress == "right") {
			return {
				...base,
				css: (t: number) => `
        left: 100vw;
        transform: translateX(${-100 * t}%);
      `
			};
		}
		return base;
	}

	function exists({ x, y }: { x: number; y: number }) {
		return !!slides[y][x];
	}

	let c = { x: $coord.x, y: $coord.y };
	let content = slides[0][0];
	let [u, d, l, r] = [false, false, false, false];
	$: {
		c = { x: $coord.x, y: $coord.y };
		if (exists(c)) {
			content = slides[c.y][c.x];
		}
	}
	$: if ($coord) {
		const copy = { x: $coord.x, y: $coord.y };
		const startStr = JSON.stringify(copy);

		const uxy = { ...copy, y: Math.max($coord.y - 1, 0) };
		u = startStr != JSON.stringify(uxy) && exists(uxy);
		const dxy = { ...copy, y: Math.min($coord.y + 1, slides.length - 1) };
		d = startStr != JSON.stringify(dxy) && exists(dxy);
		const lxy = { ...copy, x: Math.max($coord.x - 1, 0) };
		l = startStr != JSON.stringify(lxy) && exists(lxy);
		const rxy = { ...copy, x: Math.min($coord.x + 1, slides[0].length - 1) };
		r = startStr != JSON.stringify(rxy) && exists(rxy);
	}
</script>

<div class="wrapper">
	{#key content}
		<div class="slide" out:skale in:slyde={{ lastPress: $coord.lastPress }}>
			<Slide_1 />
		</div>
	{/key}
	<CanvasOverlay />
</div>
<KeyDetector
	on:ArrowDown={() => {
		const xy = { ...$coord, y: Math.min($coord.y + 1, slides.length - 1) };
		if (exists(xy)) {
			$coord = { ...xy, lastPress: "down" };
		}
	}}
	on:ArrowUp={() => {
		const xy = { ...$coord, y: Math.max($coord.y - 1, 0) };
		if (exists(xy)) {
			$coord = { ...xy, lastPress: "up" };
		}
	}}
	on:ArrowRight={() => {
		const xy = { ...$coord, x: Math.min($coord.x + 1, slides[0].length - 1) };
		if (exists(xy)) {
			$coord = { ...xy, lastPress: "right" };
		}
	}}
	on:ArrowLeft={() => {
		const xy = { ...$coord, x: Math.max($coord.x - 1, 0) };
		if (exists(xy)) {
			$coord = { ...xy, lastPress: "left" };
		}
	}}
/>
<KeyPad {u} {r} {d} {l} />

<style lang="postcss">
	.slide {
		width: 100vw;
		height: 100vh;
		@apply absolute bg-blue-800 place-items-center grid top-0 left-0;
	}
</style>
