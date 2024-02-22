<script lang="ts">
	export let index: number;
	export let focusedIndex: number;
</script>

<button
	on:click={() => {
		window.alert('hello');
	}}
	class:bg-blue-300={index === focusedIndex}
	class:bg-blue-200={index !== focusedIndex}
	class="block hover:bg-blue-300 border-box border border-blue-700 text-blue-700 transition-colors"
>
	<slot />
</button>
