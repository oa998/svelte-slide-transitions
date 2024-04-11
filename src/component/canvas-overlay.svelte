<script lang="ts">
	import { browser } from "$app/environment";
	import { onMount } from "svelte";
	import { writable } from "svelte/store";
	import KeyDetector from "./key-detector.svelte";

	let height = 0;
	let width = 0;
	const can = writable<HTMLCanvasElement>();
	const ctx = writable<CanvasRenderingContext2D>();
	const color = writable<"red" | "yellow" | "green">("yellow");
	const opacity = writable<1 | 0.3>(1);

	type Drawing = { x: number; y: number; shape: "line" | "rect"; color: string };

	const getColor = () => {
		return {
			red: `rgba(255,0,0,${$opacity})`,
			yellow: `rgba(255,255,0,${$opacity})`,
			green: `rgba(0,175,20,${$opacity})`
		}[$color];
	};

	const shape = writable<"line" | "rect">("rect");

	let [clientX, clientY] = [0, 0];

	let currentLine: Drawing[] = [];
	const finishedLines = writable<Drawing[][]>([]);
	let mouseDown = false;
	const mousePos = { x: 0, y: 0 };

	function trackMouse(e) {
		mousePos.x = e.clientX;
		mousePos.y = e.clientY;
	}

	function clear() {
		if ($ctx && $can) {
			$ctx.beginPath();
			$ctx.clearRect(0, 0, $can.width, $can.height);
			$ctx.closePath();
			$finishedLines = [];
		}
	}
	function undo() {
		$finishedLines = $finishedLines.slice(0, -1);
	}

	function drawLine(points: Drawing[]) {
		if ($ctx && points.length == 2) {
			$ctx.strokeStyle = points[0].color;
			$ctx.lineWidth = 20;
			const p1 = points.at(0)!;
			const p2 = points.at(1)!;
			$ctx.beginPath();
			$ctx.moveTo(p1.x, p1.y);
			var xc = p1.x + p2.x;
			var yc = p1.y + p2.y;
			$ctx.lineTo(p2.x, p2.y);
			$ctx.closePath();
			$ctx.stroke();
		}
	}

	function drawRect(points: Drawing[]) {
		if ($ctx && points.length == 2) {
			$ctx.strokeStyle = points[0].color;
			$ctx.lineWidth = 2;
			const p1 = points.at(0)!;
			const p2 = points.at(1)!;
			$ctx.beginPath();
			$ctx.moveTo(p1.x, p1.y);
			$ctx.rect(p1.x, p1.y, p2.x - p1.x, p2.y - p1.y);
			$ctx.closePath();
			$ctx.stroke();
		}
	}

	$: update = () => {
		if (mouseDown) {
			$ctx.clearRect(0, 0, $can.width, $can.height);
			if ($shape == "line") drawLine(currentLine);
			if ($shape == "rect") drawRect(currentLine);
			if (currentLine.length == 2) {
				currentLine = [currentLine[0]];
			}
			currentLine.push({ x: clientX, y: clientY, shape: $shape, color: getColor() });
		} else {
			if (currentLine.length == 2) {
				$finishedLines = [...$finishedLines, currentLine];
				console.log($finishedLines);
			}
			currentLine = [];
			if ($ctx) $ctx.clearRect(0, 0, $can.width, $can.height);
		}
		for (const line of $finishedLines) {
			if (line[0].shape == "line") drawLine(line);
			if (line[0].shape == "rect") drawRect(line);
		}
		// if (ctx && highlights.length > 2) {
		// 	ctx.beginPath();
		// 	ctx.shadowBlur = 4;
		// 	ctx.shadowColor = "rgba(0,0,0,0.1)";
		// 	ctx.lineWidth = 2;
		// 	ctx.lineJoin = "round";
		// 	ctx.strokeStyle = `hsla(${200},50%,50%,1)`;
		// 	ctx.moveTo(highlights[0].x, highlights[0].y);
		// 	let i = 0; // use i after the loop
		// 	for (i = 0; i < highlights.length - 2; i++) {
		// 		const c = highlights[i]; // current point
		// 		const n = highlights[i + 1]; // next point

		// 		var xc = (c.x + n.x) / 2;
		// 		var yc = (c.y + n.y) / 2;
		// 		ctx.quadraticCurveTo(c.x, c.y, xc, yc);
		// 		ctx.stroke();
		// 	}
		// 	ctx.quadraticCurveTo(
		// 		highlights[i].x,
		// 		highlights[i].y,
		// 		highlights[i + 1].x,
		// 		highlights[i + 1].y
		// 	);
		// 	ctx.stroke();
		// 	ctx.closePath();
		// }

		requestAnimationFrame(update);
	};

	onMount(() => {
		update();
		can.subscribe((canvas) => {
			if (browser && canvas) {
				$ctx = canvas.getContext("2d")!;
			}
		});
	});
</script>

<svelte:window on:mousemove={(e) => ({ clientX, clientY } = e)} />

<div
	bind:clientWidth={width}
	bind:clientHeight={height}
	class="z-10 absolute top-0 left-0"
	style="width: 100vw; height: 100vh;"
>
	<div class="flex flex-row gap-1 absolute right-1 bottom-1">
		<button
			on:click={() => ($shape = "line")}
			class:text-white={$shape == "line"}
			class="w-6 grid place-items-center border-white border rounded aspect-square text-xs active:bg-[rgba(200,200,200,0.3)]"
			>↗</button
		>
		<button
			on:click={() => ($shape = "rect")}
			class="w-6 grid place-items-center border border-white rounded aspect-square text-xs active:bg-[rgba(200,200,200,0.3)]"
		>
			<div
				class:border-white={$shape == "rect"}
				class:border-black={$shape != "rect"}
				class="w-3 h-3 border"
			/>
		</button>
		<button
			on:click={undo}
			class="px-2 grid place-items-center border border-black rounded text-xs active:bg-[rgba(200,200,200,0.3)]"
		>
			undo
		</button>
		<button
			class="w-6 grid place-items-center border rounded aspect-square text-xs active:bg-[rgba(200,200,200,0.3)] bg-red-600"
			class:border-white={$color == "red"}
			class:border-black={$color != "red"}
			on:click={() => ($color = "red")}
		/>
		<button
			class="w-6 grid place-items-center border rounded aspect-square text-xs active:bg-[rgba(200,200,200,0.3)] bg-yellow-600"
			class:border-white={$color == "yellow"}
			class:border-black={$color != "yellow"}
			on:click={() => ($color = "yellow")}
		/>
		<button
			class="w-6 grid place-items-center border rounded aspect-square text-xs active:bg-[rgba(200,200,200,0.3)] bg-green-600"
			class:border-white={$color == "green"}
			class:border-black={$color != "green"}
			on:click={() => ($color = "green")}
		/>
		<button
			class="px-2 grid place-items-center border border-white rounded text-xs active:bg-[rgba(200,200,200,0.3)]"
			class:border-white={$opacity == 1}
			class:border-black={$opacity != 1}
			on:click={() => ($opacity = 1)}>1</button
		>
		<button
			class="px-2 grid place-items-center border border-white rounded text-xs active:bg-[rgba(200,200,200,0.3)]"
			class:border-white={$opacity == 0.3}
			class:border-black={$opacity != 0.3}
			on:click={() => ($opacity = 0.3)}>0.3</button
		>
	</div>
	<canvas
		bind:this={$can}
		{width}
		{height}
		on:mousedown={() => {
			mouseDown = true;
		}}
		on:mouseup={() => (mouseDown = false)}
		on:mousemove={trackMouse}
		on:focus={() => {}}
	/>
	<KeyDetector
		on:Escape={undo}
		on:ArrowUp={clear}
		on:ArrowDown={clear}
		on:ArrowLeft={clear}
		on:ArrowRight={clear}
		on:c={clear}
	/>
</div>
