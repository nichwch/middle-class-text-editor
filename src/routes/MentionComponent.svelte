<button
	on:click={() => {
		window.alert('hello ');
	}}
	class="bg-blue-200 hover:bg-blue-300 outline outline-black"
>
	<slot />
</button>
