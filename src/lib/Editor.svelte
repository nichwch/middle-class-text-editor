<script lang="ts">
	export let splitFunc: (str: string) => string[];
	export let content: string;

	$: paragraphs = splitFunc(content);
</script>

<div class="relative h-96 overflow-y-auto border border-black">
	<div class="absolute h-full w-full top-0">
		<div class="w-full h-full p-3 absolute top-0">
			{#each paragraphs as paragraph}
				{#if paragraph.trim().length === 0}
					<br />
				{:else}
					<div class="relative block whitespace-pre-wrap">{paragraph}</div>
				{/if}
			{/each}
		</div>
		<div
			contenteditable="true"
			class="w-full min-h-full p-3 resize-none block text-red-500 absolute top-0 whitespace-pre-line break-after-right caret-black z-10 bg-transparent"
			bind:innerText={content}
		/>
	</div>
</div>
