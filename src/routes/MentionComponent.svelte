<button
	on:click={() => {
		window.alert('hello');
	}}
	class="block bg-blue-200 hover:bg-blue-300 border-box border border-blue-700 text-blue-700"
>
	<slot />
</button>
