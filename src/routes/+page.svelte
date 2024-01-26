<script>
	import { Editor, splitText } from '$lib';
	let editorContents = `Hello world 

This is some example editor content keyword1 that spans multiple lines. keyword1

@hello @nick @test @recurse

keyword1 

This is some example editor content keyword1 that spans multiple lines. keyword2

keyword2 

As you can see, we're able to apply some cool styles to different blocks.`;
	$: console.log({ editorContents });
</script>

<div class="w-full px-5 mt-10 md:mx-auto md:w-[36rem]">
	<h1 class="text-2xl mb-3">PSEUDO WYSIWYG</h1>
	<p class="mt-3"><b>pseudo:</b> kind of ish</p>
	<p><b>WYSIWYG:</b> What You See Is What You Get</p>

	<p class="mt-3">
		Pseudo WYSIWYG is an editor library that sprinkles some rich text features on top of a plain
		textarea, without becoming a full fledged full-text editor, hopefully gaining most of the
		advantages without all of the costs.
	</p>
	<p class="mt-3">
		I feel like you can make some joke around rich text editors and this being slightly less than
		that, maybe play off the wealth connotation of rich. Workshopping a few jokes below.
	</p>
	<p class="mt-3">
		It's not a rich text editor. It's middle class. It's a text editor for your average joe.
	</p>
	<p class="mt-3 mb-5">A rich text editor? In this economy?</p>

	<Editor
		bind:content={editorContents}
		keywords={['keyword1', 'keyword2', '@\\w+']}
		splitFunc={(/** @type {string} */ text) => text.split('\n')}
	/>

	<p class="mt-3">
		Pseudo WYSIWYG is an editor library that sprinkles some rich text features on top of a plain
		textarea, without becoming a full fledged full-text editor, hopefully gaining most of the
		advantages without all of the costs.
	</p>
	<p class="mt-3">
		I feel like you can make some joke around rich text editors and this being slightly less than
		that, maybe play off the wealth connotation of rich. Workshopping a few jokes below.
	</p>
	<p class="mt-3">
		It's not a rich text editor. It's middle class. It's a text editor for your average joe.
	</p>
	<p class="mt-3 mb-5">A rich text editor? In this economy?</p>
	<p class="mt-3">
		Pseudo WYSIWYG is an editor library that sprinkles some rich text features on top of a plain
		textarea, without becoming a full fledged full-text editor, hopefully gaining most of the
		advantages without all of the costs.
	</p>
	<p class="mt-3">
		I feel like you can make some joke around rich text editors and this being slightly less than
		that, maybe play off the wealth connotation of rich. Workshopping a few jokes below.
	</p>
	<p class="mt-3">
		It's not a rich text editor. It's middle class. It's a text editor for your average joe.
	</p>
	<p class="mt-3 mb-5">A rich text editor? In this economy?</p>
	<p class="mt-3">
		Pseudo WYSIWYG is an editor library that sprinkles some rich text features on top of a plain
		textarea, without becoming a full fledged full-text editor, hopefully gaining most of the
		advantages without all of the costs.
	</p>
	<p class="mt-3">
		I feel like you can make some joke around rich text editors and this being slightly less than
		that, maybe play off the wealth connotation of rich. Workshopping a few jokes below.
	</p>
	<p class="mt-3">
		It's not a rich text editor. It's middle class. It's a text editor for your average joe.
	</p>
	<p class="mt-3 mb-5">A rich text editor? In this economy?</p>
</div>
