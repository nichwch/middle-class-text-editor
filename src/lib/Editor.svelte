<script lang="ts">
	export let splitFunc: (str: string) => string[];
	export let content: string;
	export let keywords: string[];

	const regex = new RegExp(`(${keywords.join('|')})`, 'g');

	$: paragraphs = splitFunc(content);
	$: formatted_paragraphs = paragraphs.map((para) => {
		return para.split(regex);
	});
	$: console.log(formatted_paragraphs);
</script>

<div class="relative h-96 overflow-y-auto border border-black">
	<div class="absolute h-full w-full top-0">
		<div class="w-full h-full p-3 absolute top-0 whitespace-pre-line break-after-right">
			{#each formatted_paragraphs as paragraph}
				{#if paragraph.length === 0}
					<br />
				{:else}
					<div class="relative block whitespace-pre-wrap">
						{#each paragraph as clause}
							{#if keywords.includes(clause)}
								<span class="bg-green-500">{clause}</span>
							{:else}
								<span>{clause}</span>
							{/if}
						{/each}
					</div>
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
