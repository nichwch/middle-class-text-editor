<button
	on:click={() => {
		window.alert('hello ');
	}}
	class="bg-blue-200 hover:bg-blue-300 outline outline-blue-700 text-blue-700"
>
	<slot />
</button>
