<button
	on:click={() => {
		window.alert('project');
	}}
	class="bg-green-200 hover:bg-green-300 border-box border-2 border-green-700 text-green-700"
>
	<slot />
</button>
