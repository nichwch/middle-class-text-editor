<script lang="ts">
	export let index: number;
	export let focusedIndex: number;
	$: console.log(index, focusedIndex);
</script>

<button
	on:click={() => {
		window.alert('project');
	}}
	class:bg-green-300={index === focusedIndex}
	class:bg-green-200={index !== focusedIndex}
	class=" hover:bg-green-300 border-box border border-green-700 text-green-700 transition-colors"
>
	<slot />
</button>
