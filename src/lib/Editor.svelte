<script lang="ts">
	import { SvelteComponent, afterUpdate } from 'svelte';

	export let splitFunc: (str: string) => string[];
	export let content: string;
	export let keywordMap: {
		[key: string]: {
			component?: SvelteComponent;
			includeFunction?: (str: string) => boolean;
		};
	} = {};

	const makeDashesNonBreaking = (str: string) => str.replaceAll('-', '‑');
	const makeDashesBreaking = (str: string) => str.replaceAll('‑', '-');

	$: keywords = Object.keys(keywordMap) || [];
	$: regex = new RegExp(`(${keywords.join('|')})`, 'g');

	const getKeywordMatch = (str: string): string | undefined => {
		if (keywords.includes(str)) return str;
		for (const keyword of keywords) {
			if (new RegExp(keyword).test(str)) {
				const includeFunction = keywordMap[keyword].includeFunction;
				if (includeFunction !== undefined) {
					const included = includeFunction(makeDashesBreaking(str));
					if (included) return keyword;
					else return undefined;
				} else {
					return keyword;
				}
			}
		}
		return undefined;
	};

	$: formatted_paragraphs = splitFunc(content).map((para) => {
		return para.split(regex);
	});
	let keywordLocations: { [key: string]: { top: number; left: number } } = {};
	let editorScrollHeight = 0;
	let textareaRef: Element | null = null;
	afterUpdate(() => {
		keywordLocations = {};
		formatted_paragraphs.forEach((paragraph, paragraph_index) => {
			paragraph.forEach((clause, clause_index) => {
				if (getKeywordMatch(clause)) {
					const underblock = document.getElementById(
						`underblock-${paragraph_index},${clause_index}`
					);
					keywordLocations[`${paragraph_index},${clause_index}`] = {
						top: underblock?.offsetTop || 0,
						left: underblock?.offsetLeft || 0
					};
				}
			});
		});
		keywordLocations = keywordLocations;
	});
</script>

<div class="relative h-96 overflow-y-auto border border-black">
	<div class="absolute h-full w-full top-0">
		<!-- THE UNDERLAY -->
		<div class="leading-6 w-full h-full p-3 absolute top-0 whitespace-pre-line">
			{#each formatted_paragraphs as paragraph, paragraph_index}
				{#if paragraph.length === 0}
					<br />
				{:else}
					<div class="block whitespace-pre-wrap">
						{#each paragraph as clause, clause_index}
							{#if getKeywordMatch(clause) !== undefined}
								<div class="inline-block" id="underblock-{paragraph_index},{clause_index}">
									{clause}
								</div>
							{:else}
								<span>{clause}</span>
							{/if}
						{/each}
					</div>
				{/if}
			{/each}
		</div>
		<!-- THE EDITOR -->
		<textarea
			style:height="{editorScrollHeight}px"
			class="w-full min-h-full p-3 leading-6 resize-none block absolute top-0 whitespace-pre-line break-after-right caret-black z-10 bg-transparent"
			value={makeDashesNonBreaking(content)}
			bind:this={textareaRef}
			on:input={(evt) => {
				editorScrollHeight = textareaRef?.scrollHeight || 0;
				//@ts-ignore
				content = evt?.target?.value;
				console.log('selection start', evt.target.selectionStart);
				console.log('selection end', evt.target.selectionEnd);
			}}
		/>
		<!-- THE OVERLAY -->
		<div class="absolute top-0 w-full h-fullp-3 whitespace-pre-wrap leading-6">
			{#each formatted_paragraphs as paragraph, paragraph_index}
				{#each paragraph as clause, clause_index}
					{@const match = getKeywordMatch(clause)}

					{#if match !== undefined}
						{@const component = keywordMap[match].component}
						{#if component}
							<span
								class="z-20 leading-6"
								style:position="absolute"
								style:top="{keywordLocations[`${paragraph_index},${clause_index}`]?.top}px"
								style:left="{keywordLocations[`${paragraph_index},${clause_index}`]?.left}px"
							>
								<svelte:component this={component}>
									{clause}
								</svelte:component>
							</span>
						{:else}
							<span
								class="z-20 leading-6 bg-red-200 hover:bg-red-300 outline outline-black"
								style:position="absolute"
								style:top="{keywordLocations[`${paragraph_index},${clause_index}`]?.top}px"
								style:left="{keywordLocations[`${paragraph_index},${clause_index}`]?.left}px"
							>
								{clause}
							</span>
						{/if}
					{/if}
				{/each}
			{/each}
		</div>
	</div>
</div>
