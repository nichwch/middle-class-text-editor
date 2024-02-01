<script>
	import { Editor, splitText } from '$lib';
	import MentionComponent from './MentionComponent.svelte';
	import ProjectComponent from './ProjectComponent.svelte';
	let editorContents = `Good morning team! 

Here are the tasks for the week:
- @nick will be working on the #svelte-refactor project
- @alice and @bob will be working on the #e2e-encryption project

Let's see how it works as we approach the borders of the div. #svelte-refactor
Let's see how it works as we approach the borders of the div. #project.project.project 
Let's see how it works as we approach the borders of the div. #project/project/project
`;
</script>

<div class="w-full px-5 mt-10 md:mx-auto md:w-[36rem]">
	<h1 class="text-2xl mb-3">PSEUDO WYSIWYG</h1>
	<h2><a class="link" href="https://nicholaschen.io">By Nicholas Chen</a></h2>
	<h2><a class="link" href="https://github.com/nichwch/pseudo-wysiwyg">source</a></h2>
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
	<p class="mt-3 mb-5">Editor? I hardly know her!</p>

	<Editor
		bind:content={editorContents}
		keywordMap={{
			'@': {
				component: MentionComponent,
				recognized: ['alice', 'bob', 'nick'],
				allowUnrecognized: false,
				includeFunction: (/** @type {string} */ str) => {
					const recognizedNames = ['alice', 'bob', 'nick'];
					const namesWithAt = recognizedNames.map((name) => `@${name}`);
					if (namesWithAt.includes(str)) return true;
					return false;
				}
			},
			'#': {
				component: ProjectComponent,
				recognized: [
					'svelte-refactor',
					'e2e-encryption',
					'project.project.project',
					'project/project/project'
				],
				allowUnrecognized: false,
				includeFunction: (/** @type {string} */ str) => {
					const recognizedNames = [
						'svelte-refactor',
						'e2e-encryption',
						'project.project.project',
						'project/project/project'
					];
					const namesWithAt = recognizedNames.map((name) => `#${name}`);
					if (namesWithAt.includes(str)) return true;
					return false;
				}
			}
		}}
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
