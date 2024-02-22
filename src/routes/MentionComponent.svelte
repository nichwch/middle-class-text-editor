<script lang="ts">
	import * as HoverCard from '$lib/components/ui/hover-card';
	export let index: number;
	export let focusedIndex: number;
</script>

<HoverCard.Root>
	<HoverCard.Trigger
		><button
			on:click={() => {}}
			class:bg-blue-300={index === focusedIndex}
			class:bg-blue-200={index !== focusedIndex}
			class="block hover:bg-blue-300 border-box border border-blue-700 text-blue-700 transition-colors"
		>
			<slot />
		</button></HoverCard.Trigger
	>
	<HoverCard.Content>More information about this user.</HoverCard.Content>
</HoverCard.Root>
