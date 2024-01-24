<script lang="ts">
	export let content: string;

	$: paragraphs = content.split('\n');
</script>

<div class="relative h-96 overflow-y-auto border border-black">
	<div class="absolute h-full w-full top-0">
		<div class="w-full h-full p-3 absolute top-0">
			{#each paragraphs as paragraph}
				{#if paragraph.trim().length === 0}
					<br />
				{:else}
					<div class="relative block whitespace-pre-line">{paragraph}</div>
				{/if}
			{/each}
		</div>
		<div
			contenteditable="true"
			class="w-full p-3 resize-none block text-red-500 absolute top-0 whitespace-pre-line caret-black z-10 bg-transparent"
			bind:innerText={content}
		/>
	</div>
</div>
