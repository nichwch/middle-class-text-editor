<script lang="ts">
	import { computePosition, offset, shift, arrow, flip } from '@floating-ui/dom';
	import { text } from '@sveltejs/kit';
	import { SvelteComponent, afterUpdate, tick } from 'svelte';

	export let splitFunc: (str: string) => string[];
	export let content: string;
	export let keywordMap: {
		[key: string]: {
			component?: SvelteComponent;
			recognized: string[];
			includeFunction?: (str: string) => boolean;
			allowUnrecognized: boolean;
		};
	} = {};

	const NON_BREAKING_DASH = '‑';
	const makeDashesNonBreaking = (str: string) => str.replaceAll('-', NON_BREAKING_DASH);
	const makeDashesBreaking = (str: string) => str.replaceAll(NON_BREAKING_DASH, '-');
	// replace non breaking dashes in initial input
	content = makeDashesNonBreaking(content);

	$: keywords = Object.keys(keywordMap) || [];
	const getRule = (str: string) => `${str}\\S+`;
	$: rules = keywords.map(getRule);
	$: regex = new RegExp(`(${rules.join('|')})`, 'g');

	const getKeywordMatch = (str: string): string | undefined => {
		if (keywords.includes(str)) return str;
		for (const keyword of keywords) {
			const rule = getRule(keyword);
			const { recognized, allowUnrecognized } = keywordMap[keyword];
			if (new RegExp(rule).test(str) && str.trim().length > 1) {
				if (!allowUnrecognized) {
					const namesWithKeyword = recognized.map((name) => `${keyword}${name}`);
					if (namesWithKeyword.includes(makeDashesBreaking(str))) return keyword;
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
	let caretRef: HTMLElement | null = null;
	let slashMenuRef: HTMLElement | null = null;
	let caretPosition = 0;
	let textBeforeCaret = '';
	let slashMenuX = 0;
	let slashMenuY = 0;
	afterUpdate(() => {
		computePosition(caretRef!, slashMenuRef!, {
			placement: 'bottom',
			middleware: [flip(), shift()]
		}).then(({ x, y }) => {
			slashMenuX = x;
			slashMenuY = y;
		});
	});

	let menuOptions: string[] | null = null;
	let showingSlashMenu: string | null = null;
	let slashMenuStartIndex = 0;
	let menuPosition = 0;
	let slashMenuInput: string | null = null;
	$: shownMenuOptions =
		slashMenuInput === null
			? menuOptions || []
			: menuOptions?.filter((option) => {
					return option.includes(slashMenuInput!);
				}) || [];

	$: console.log('slashMenuInput', slashMenuInput);
	$: console.log('slashMenuOptions', shownMenuOptions);
	$: console.log('menuPosition', menuPosition);

	const resetMenu = () => {
		showingSlashMenu = null;
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

	const scrollToMenuItem = () => {
		const menuItem = document.getElementById(`slash-menu-${menuPosition}`);
		menuItem?.scrollIntoView({ block: 'nearest', inline: 'nearest' });
	};

	const processKeyDown = (evt: KeyboardEvent) => {
		//@ts-ignore
		caretPosition = evt.target.selectionEnd;
		// close the menu if the user types out the entire option
		if (
			showingSlashMenu !== null &&
			shownMenuOptions.length === 1 &&
			slashMenuInput + evt.key === shownMenuOptions[0]
		) {
			resetMenu();
		}
		// open the menu if the user types a keyword and the menu isn't open
		else if (keywords.includes(evt.key) && showingSlashMenu === null) {
			menuOptions = keywordMap[evt.key].recognized;
			showingSlashMenu = evt.key;
			slashMenuStartIndex = caretPosition;
		}
		// move up an option
		else if (showingSlashMenu !== null && evt.key === 'ArrowUp') {
			menuPosition = Math.max(0, menuPosition - 1);
			scrollToMenuItem();
			evt.preventDefault();
			evt.stopPropagation();
		}
		// move down an option
		else if (showingSlashMenu !== null && evt.key === 'ArrowDown') {
			menuPosition = Math.min(shownMenuOptions.length - 1, menuPosition + 1);
			scrollToMenuItem();
			evt.preventDefault();
			evt.stopPropagation();
		}
		// if the user continues typing, modify slashMenuInput
		// reset menuPosition because menu options will change to match new input
		else if (showingSlashMenu !== null && evt.key.length === 1) {
			slashMenuInput = content.substring(slashMenuStartIndex + 1, caretPosition) + evt.key;
			menuPosition = 0;
			scrollToMenuItem();
		}
		// handle backspace
		else if (showingSlashMenu !== null && evt.key === 'Backspace') {
			if (slashMenuInput === null || slashMenuInput!.length === 0) {
				resetMenu();
			} else {
				slashMenuInput = content.substring(slashMenuStartIndex + 1, caretPosition - 1);
				menuPosition = 0;
				scrollToMenuItem();
			}
		}
		// close menu when escape is handled
		else if (evt.key === 'Escape') {
			showingSlashMenu = null;
			evt.preventDefault();
			evt.stopPropagation();
			evt.stopImmediatePropagation();
		}
		// when the user presses enter, insert the selected option
		else if (showingSlashMenu !== null && evt.key === 'Enter') {
			const insertedStr = showingSlashMenu + shownMenuOptions[menuPosition] + ' ';
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
		textBeforeCaret = content.substring(0, caretPosition);
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
			<span class="relative text-transparent inline">
				{textBeforeCaret}
			</span>

			<span class="inline-block w-1 h-6" id="caret" bind:this={caretRef} />
			<!-- SLASH MENU -->
			<div
				style:position="absolute"
				class:hidden={!showingSlashMenu}
				class="top-10 z-40 w-72 h-24 border border-black bg-white overflow-y-scroll"
				style:top="{slashMenuY}px"
				style:left="{slashMenuX}px"
				bind:this={slashMenuRef}
			>
				{#each shownMenuOptions as option, index}
					<button
						id="slash-menu-{index}"
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
				caretPosition = evt.target.selectionStart;
			}}
		/>
		<!-- THE OVERLAY -->
		<div class="absolute top-0 w-full h-fullp-3 whitespace-pre-wrap leading-6">
			{#each formatted_paragraphs as paragraph, paragraph_index}
				{#each paragraph as clause, clause_index}
					{@const match = getKeywordMatch(clause)}
					{@const location = keywordLocations[`${paragraph_index},${clause_index}`]}
					{#if match !== undefined && location !== undefined}
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
								style:top="{location?.top}px"
								style:left="{location?.left}px"
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
