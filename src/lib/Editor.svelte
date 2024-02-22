<script lang="ts">
	import { computePosition, shift, flip } from '@floating-ui/dom';
	import { SvelteComponent, afterUpdate, tick } from 'svelte';

	export let splitFunc: (str: string) => string[] = (text) => text.split('\n');
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
	$: splitKeywords = content.split(regex);
	$: keywordIndices = splitKeywords.reduce((acc, cur) => {
		const lastSegment = acc.length > 0 ? acc[acc.length - 1] : 0;
		const nextIndex = lastSegment + cur.length;
		acc.push(nextIndex);
		return acc;
	}, [] as number[]);
	$: console.log('keywordindices', keywordIndices);

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

	// returns true if keyword is under caret position
	// TODO: make it so that it returns the keyword and its location

	const isOnKeyword = (caretPos: number): number | undefined => {
		for (let i = 0; i < keywordIndices.length - 1; i++) {
			let currInd = keywordIndices[i];
			if (caretPos === currInd) return Math.floor(i / 2);
			if (caretPos >= currInd && caretPos < keywordIndices[i + 1]) {
				if (i % 2 === 0) return i / 2;
			}
		}
	};

	$: indexOfCursoredKeyword = isOnKeyword(caretPosition);
	$: console.log({ indexOfCursoredKeyword });

	// afterUpdate(() => {
	// 	tick().then(() => {
	// 		console.log(
	// 			'!!!2',
	// 			document?.activeElement?.selectionStart,
	// 			document?.activeElement?.selectionEnd
	// 		);
	// 		console.log('IND2', isOnKeyword(document?.activeElement?.selectionEnd));
	// 	});
	// });

	// returns the right boundary if caret position is on left boundary
	const isOnEndBoundary = (caretPos: number): number | undefined => {
		const indexOfKeyword = keywordIndices.indexOf(caretPos);
		// looking for EVEN keywords indices for ENDS of keywords
		if (indexOfKeyword >= 0 && indexOfKeyword % 2 === 1) {
			const targetIndex = keywordIndices[indexOfKeyword - 1];
			return targetIndex;
		}
		return undefined;
	};
	const isOnStartBoundary = (caretPos: number): number | undefined => {
		const indexOfKeyword = keywordIndices.indexOf(caretPos);
		// looking for EVEN keywords indices for STARTS of keywords
		if (indexOfKeyword >= 0 && indexOfKeyword % 2 === 0) {
			const targetIndex = keywordIndices[indexOfKeyword + 1];
			return targetIndex;
		}
		return undefined;
	};

	const processKeyUp = (evt: KeyboardEvent) => {
		//@ts-ignore
		caretPosition = evt.target.selectionEnd;
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
		else if (
			showingSlashMenu !== null &&
			evt.key === 'Enter' &&
			shownMenuOptions[menuPosition] !== undefined
		) {
			const insertedStr = showingSlashMenu + shownMenuOptions[menuPosition] + ' ';
			insertMenuOption(insertedStr);
			evt.preventDefault();
			evt.stopPropagation();
			evt.stopImmediatePropagation();
			resetMenu();
		}
		// any other input should reset the menu
		else {
			/*
 	the regex split is gauranteed to alternate between keywords and non keywords
	
	this is important, because moving left at the end of a keyword and moving right at the start of 
	a keyword should move the user through the keyword, but NOT if they move RIGHT and the END or 
	LEFT at the START.
	*/
			// moving through keyword if to the left

			if (evt.key === 'ArrowLeft') {
				const targetIndex = isOnEndBoundary(caretPosition);
				// if first split is keyword, we are looking for
				// looking for ODD keywords indices for ENDS of keywords
				if (targetIndex !== undefined) {
					textAreaRef?.focus();
					textAreaRef?.setSelectionRange(targetIndex, targetIndex);
				}
				resetMenu();
			}
			// moving through keyword if to the right
			if (evt.key === 'ArrowRight') {
				const targetIndex = isOnStartBoundary(caretPosition);
				// if first split is keyword, we are looking for
				// looking for ODD keywords indices for ENDS of keywords
				if (targetIndex !== undefined) {
					textAreaRef?.focus();
					textAreaRef?.setSelectionRange(targetIndex, targetIndex);
				}
				resetMenu();
			}
			if (evt.key === 'Backspace') {
				// check if on end boundary to delete keyword afterwards
				// the second condition is if the user deletes a space before hand
				const targetIndex = isOnEndBoundary(caretPosition) || isOnEndBoundary(caretPosition - 1);
				console.log('target index', targetIndex);
				if (targetIndex !== undefined) {
					textAreaRef?.focus();
					textAreaRef?.setSelectionRange(targetIndex, caretPosition, 'backward');
					textAreaRef?.setRangeText('');
					textAreaRef?.setSelectionRange(targetIndex, targetIndex);
				}
				resetMenu();
			}
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
								<span id="underblock-{paragraph_index},{clause_index}">
									{clause}
								</span>
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
							insertMenuOption(showingSlashMenu + option + ' ');
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
			class="w-full min-h-full p-3 leading-6 resize-none block absolute top-0 caret-black z-10 bg-transparent"
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
			on:keyup={processKeyUp}
			on:keydown={processKeyDown}
			on:click={() => {
				showingSlashMenu = null;
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
