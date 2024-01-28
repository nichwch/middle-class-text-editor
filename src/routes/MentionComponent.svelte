<button
	on:click={() => {
		window.alert('hello');
	}}
	class="bg-blue-200 hover:bg-blue-300 border-box border md:border-2 border-blue-700 text-blue-700"
>
	<slot />
</button>
