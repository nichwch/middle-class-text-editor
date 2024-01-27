<button
	on:click={() => {
		window.alert('project');
	}}
	class="bg-green-200 hover:bg-green-300 outline outline-green-700 text-green-700"
>
	<slot />
</button>
