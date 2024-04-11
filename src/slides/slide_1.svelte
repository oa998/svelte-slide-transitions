<script>
	import Javascript from "../component/javascript.svelte";
</script>

<div class="slide-outer">
	<pre>This is some content</pre>
	<Javascript
		code={`const c = 'code'; // don't think about it
const c = 'code'; // don't think about it
const c = 'code'; // don't think about it
const c = 'code'; // don't think about it`}
	/>
</div>

<style lang="postcss">
	.slide-outer {
		grid-template-columns: 1fr 1fr;
		@apply grid place-items-center w-full h-full;
	}
</style>
