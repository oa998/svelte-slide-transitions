<script lang="ts">
	import { createEventDispatcher } from "svelte";

	const dispatcher = createEventDispatcher();
	type KeyTypes = "ArrowDown" | "ArrowUp" | "ArrowRight" | "ArrowLeft" | "c";

	const onKeyDown = (event: KeyboardEvent) => {
		dispatcher(event?.key as KeyTypes);
	};
</script>

<svelte:window on:keydown|stopPropagation={onKeyDown} />
