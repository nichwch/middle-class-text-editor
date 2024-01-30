<script lang="ts">
	import { text } from '@sveltejs/kit';
	import { SvelteComponent, afterUpdate, tick } from 'svelte';

	export let splitFunc: (str: string) => string[];
	export let content: string;
	export let keywordMap: {
		[key: string]: {
			component?: SvelteComponent;
			includeFunction?: (str: string) => boolean;
		};
	} = {};

	const NON_BREAKING_DASH = '‑';
	const makeDashesNonBreaking = (str: string) => str.replaceAll('-', NON_BREAKING_DASH);
	const makeDashesBreaking = (str: string) => str.replaceAll(NON_BREAKING_DASH, '-');
	// replace non breaking dashes in initial input
	content = makeDashesNonBreaking(content);

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
	let textAreaRef: HTMLTextAreaElement | null = null;
	let caretPosition = 0;
	$: textBeforeCaret = content.substring(0, caretPosition);

	let menuOptions = ['nick', 'alice', 'bob'];
	let showingSlashMenu = false;
	let slashMenuStartIndex = 0;
	let menuPosition = 0;
	let slashMenuInput: string | null = null;
	$: shownMenuOptions =
		slashMenuInput === null
			? menuOptions
			: menuOptions.filter((option) => {
					return option.includes(slashMenuInput!);
				});

	$: console.log('slashMenuInput', slashMenuInput);
	$: console.log('slashMenuOptions', shownMenuOptions);
	$: console.log('menuPosition', menuPosition);

	const resetMenu = () => {
		showingSlashMenu = false;
		slashMenuInput = null;
	};

	const insertMenuOption = (str: string) => {
		textAreaRef?.focus();
		textAreaRef?.setSelectionRange(slashMenuStartIndex, textAreaRef.selectionStart);
		textAreaRef?.setRangeText(str);
		const newCaretPosition = slashMenuStartIndex + str.length;
		textAreaRef?.setSelectionRange(newCaretPosition, newCaretPosition);
		content = textAreaRef?.value!;
	};

	const processKeyDown = (evt: KeyboardEvent) => {
		//@ts-ignore
		caretPosition = evt.target.selectionEnd;
		// close the menu if the user types out the entire option
		if (
			showingSlashMenu &&
			shownMenuOptions.length === 1 &&
			slashMenuInput + evt.key === shownMenuOptions[0]
		) {
			resetMenu();
		}
		// open the menu if the user types an @ and the menu isn't open
		else if (evt.key === '@' && !showingSlashMenu) {
			showingSlashMenu = true;
			slashMenuStartIndex = caretPosition;
		}
		// move up an option
		else if (showingSlashMenu && evt.key === 'ArrowUp') {
			menuPosition = Math.max(0, menuPosition - 1);
			evt.preventDefault();
			evt.stopPropagation();
		}
		// move down an option
		else if (showingSlashMenu && evt.key === 'ArrowDown') {
			menuPosition = Math.min(shownMenuOptions.length - 1, menuPosition + 1);
			evt.preventDefault();
			evt.stopPropagation();
		}
		// if the user continues typing, modify slashMenuInput
		// reset menuPosition because menu options will change to match new input
		else if (showingSlashMenu && evt.key.length === 1) {
			slashMenuInput = content.substring(slashMenuStartIndex + 1, caretPosition) + evt.key;
			menuPosition = 0;
		}
		// handle backspace
		else if (showingSlashMenu && evt.key === 'Backspace') {
			if (slashMenuInput === null || slashMenuInput!.length === 0) {
				resetMenu();
			} else {
				slashMenuInput = content.substring(slashMenuStartIndex + 1, caretPosition - 1);
				menuPosition = 0;
			}
		}
		// close menu when escape is handled
		else if (evt.key === 'Escape') {
			showingSlashMenu = false;
			evt.preventDefault();
			evt.stopPropagation();
			evt.stopImmediatePropagation();
		}
		// when the user presses enter, insert the selected option
		else if (showingSlashMenu && evt.key === 'Enter') {
			const insertedStr = '@' + shownMenuOptions[menuPosition] + ' ';
			insertMenuOption(insertedStr);
			evt.preventDefault();
			evt.stopPropagation();
			evt.stopImmediatePropagation();
			resetMenu();
		}
		// any other input should reset the menu
		else {
			resetMenu();
		}
	};

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

<div class="relative h-96 overflow-y-auto overscroll-none border border-black">
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
		<!-- UNDERLAY FOR CURSOR POSITION  -->
		<div class="leading-6 w-full h-full p-3 absolute top-0 whitespace-pre-line">
			<!-- {#if showingSlashMenu}
				<div class="inline-block relative top-10 z-30 w-72 h-24 border border-black bg-white"></div>
			{/if} -->
			<span class="relative text-transparent">
				{textBeforeCaret}
			</span>
			<!-- SLASH MENU -->

			<span class="inline-block w-2 h-6" id="caret">
				{#if showingSlashMenu}
					<div class="relative top-10 z-40 w-72 h-24 border border-black bg-white">
						{#each shownMenuOptions as option, index}
							<button
								class:bg-red-200={index === menuPosition}
								class="block w-full text-left px-2 hover:bg-red-100"
								on:click={() => {
									insertMenuOption('@' + option + ' ');
									resetMenu();
								}}
							>
								{option}
							</button>
						{/each}
					</div>
				{/if}
			</span>
		</div>
		<!-- THE EDITOR -->
		<textarea
			style:height="{editorScrollHeight}px"
			class="w-full min-h-full p-3 leading-6 resize-none block absolute top-0 whitespace-pre-line break-after-right caret-black z-10 bg-transparent"
			value={content}
			bind:this={textAreaRef}
			on:input={(evt) => {
				editorScrollHeight = textAreaRef?.scrollHeight || 0;
				// replace dashes with non breaking dashes, so tag components don't get broken
				//@ts-ignore
				if (evt.data === '-') {
					textAreaRef?.setSelectionRange(
						textAreaRef?.selectionStart - 1,
						textAreaRef.selectionStart
					);
					textAreaRef?.setRangeText(NON_BREAKING_DASH);
					textAreaRef?.setSelectionRange(
						textAreaRef.selectionStart + 1,
						textAreaRef.selectionStart + 1
					);
				}
				//@ts-ignore
				content = evt?.target?.value;
			}}
			on:keydown={processKeyDown}
			on:click={() => {
				showingSlashMenu = false;
			}}
			on:selectionchange={(evt) => {
				//@ts-ignore
				caretPosition = evt.target.selectionEnd;
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
