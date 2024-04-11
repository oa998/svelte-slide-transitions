<script lang="ts">
	import hljs from "highlight.js/lib/core";
	import js from "highlight.js/lib/languages/YAML";
	import "highlight.js/styles/github.css";
	hljs.registerLanguage("yaml", js);
	export let code: string;
	const s = hljs.highlight(code, { language: "yaml" }).value;
</script>

<div class="bg-gray-200 p-3 font-mono whitespace-pre-wrap">
	{@html s}
</div>
