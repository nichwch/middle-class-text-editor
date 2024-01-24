<script>
	import { Editor } from '$lib';
	let editorContents = `Hello world 

This is some example editor content that spans multiple lines.

As you can see, we're able to apply some cool styles to different blocks.
Hello world 

This is some example editor content that spans multiple lines.

As you can see, we're able to apply some cool styles to different blocks.
Hello world 

This is some example editor content that spans multiple lines.

As you can see, we're able to apply some cool styles to different blocks.

Hello world 

This is some example editor content that spans multiple lines.

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

	<Editor addEditorClasses="mt-5" bind:content={editorContents} />
</div>
