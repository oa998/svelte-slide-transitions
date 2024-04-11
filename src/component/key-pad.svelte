<script lang="ts">
	export let u = false;
	export let d = false;
	export let l = false;
	export let r = false;
</script>

<div class="d-pad-grid">
	<div class:active={u} class="up cell" />

	<div class:active={l} class="left cell" />
	<div class:active={r} class="right cell" />
	<div class:active={d} class="down cell" />
</div>

<style>
	.d-pad-grid {
		z-index: 10;
		width: 90px;
		height: 90px;
		display: grid;
		grid-template-areas:
			"a u b"
			"l c r"
			"e d f";
		grid-template-rows: repeat(3, 30px);
		grid-template-columns: repeat(3, 30px);
		position: absolute;
		bottom: 10px;
		left: 10px;
	}

	.cell {
		display: grid;
		place-items: center;
		transform: rotateZ(45deg);
	}
	.active {
		background-color: rgba(100, 200, 90, 0.3);
	}
	.up {
		grid-area: u;
	}
	.left {
		grid-area: l;
	}
	.right {
		grid-area: r;
	}
	.down {
		grid-area: d;
	}
</style>
