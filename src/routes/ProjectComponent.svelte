<script lang="ts">
	import * as HoverCard from '$lib/components/ui/hover-card';
	export let index: number;
	export let focusedIndex: number;
</script>

<HoverCard.Root openDelay={0}>
	<HoverCard.Trigger
		><button
			on:click={() => {}}
			class:bg-green-300={index === focusedIndex}
			class:bg-green-200={index !== focusedIndex}
			class=" hover:bg-green-300 border-box border border-green-700 text-green-700 transition-colors"
		>
			<slot />
		</button></HoverCard.Trigger
	>
	<HoverCard.Content>More information about this project.</HoverCard.Content>
</HoverCard.Root>
