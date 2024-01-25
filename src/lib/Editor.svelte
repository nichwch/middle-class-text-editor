<script lang="ts">
	import { afterUpdate } from 'svelte';

	export let splitFunc: (str: string) => string[];
	export let content: string;
	export let keywords: string[];

	const regex = new RegExp(`(${keywords.join('|')})`, 'g');

	$: paragraphs = splitFunc(content);
	$: formatted_paragraphs = paragraphs.map((para) => {
		return para.split(regex);
	});
	$: console.log(formatted_paragraphs);
	let keywordLocations: { [key: string]: { top: number; left: number } } = {};
	let editorRef: Element | null = null;
	afterUpdate(() => {
		keywordLocations = {};
		formatted_paragraphs.forEach((paragraph, paragraph_index) => {
			paragraph.forEach((clause, clause_index) => {
				if (keywords.includes(clause)) {
					const underblock = document.getElementById(
						`underblock-${paragraph_index},${clause_index}`
					);
					const clientRect = underblock?.getClientRects()?.[0];
					const editorRect = editorRef?.getBoundingClientRect();
					console.log(clientRect);
					keywordLocations[`${paragraph_index},${clause_index}`] = {
						top: clientRect.y - editorRect.y || 0,
						left: clientRect.x - editorRect.x || 0
					};
				}
			});
		});
		keywordLocations = keywordLocations;
	});
	$: console.log({ keywordLocations });
</script>

<div class="relative h-96 overflow-y-auto border border-black" bind:this={editorRef}>
	<div class="absolute h-full w-full top-0">
		<div class="leading-6 w-full h-full p-3 absolute top-0 whitespace-pre-line break-after-right">
			{#each formatted_paragraphs as paragraph, paragraph_index}
				{#if paragraph.length === 0}
					<br />
				{:else}
					<div class="relative block whitespace-pre-wrap">
						{#each paragraph as clause, clause_index}
							{#if keywords.includes(clause)}
								<span id="underblock-{paragraph_index},{clause_index}">{clause}</span>
							{:else}
								<span>{clause}</span>
							{/if}
						{/each}
					</div>
				{/if}
			{/each}
		</div>
		<div
			contenteditable="plaintext-only"
			class="w-full min-h-full p-3 leading-6 resize-none block text-red-900 absolute top-0 whitespace-pre-line break-after-right caret-black z-10 bg-transparent"
			bind:innerText={content}
		/>
		<div class="leading-6">
			{#each formatted_paragraphs as paragraph, paragraph_index}
				{#each paragraph as clause, clause_index}
					{#if keywords.includes(clause)}
						<span
							id="inline-block overblock-{paragraph_index},{clause_index}"
							class="z-20 leading-4 bg-red-200 hover:bg-red-300 outline outline-black"
							style:position="absolute"
							style:top="{keywordLocations[`${paragraph_index},${clause_index}`]?.top}px"
							style:left="{keywordLocations[`${paragraph_index},${clause_index}`]?.left}px"
						>
							{clause}
						</span>
					{/if}
				{/each}
			{/each}
		</div>
	</div>
</div>
