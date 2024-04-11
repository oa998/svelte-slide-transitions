<script lang="ts">
	import hljs from "highlight.js/lib/core";
	import js from "highlight.js/lib/languages/javascript";
	// import "highlight.js/styles/github.css";
	import "highlight.js/styles/base16/atelier-cave.css";
	hljs.registerLanguage("javascript", js);
	export let code: string;
	const s = hljs.highlight(code, { language: "javascript" }).value;
</script>

<div class="bg-slate-900 py-6 px-10 font-mono whitespace-pre-wrap text-gray-400">
	{@html s}
</div>
