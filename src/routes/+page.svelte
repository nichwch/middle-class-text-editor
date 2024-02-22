<script lang="ts">
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

	let nameInput = `alice
bob
nick
`;
	let projectInput = `svelte-refactor
e2e-encryption
project.project.project
project/project/project
`;

	const getListFromString = (str: string): string[] =>
		str
			.split('\n')
			.map((str) => str.trim())
			.filter((str) => str.length > 0);
	$: names = getListFromString(nameInput);
	$: projects = getListFromString(projectInput);
</script>

<div class="w-full px-5 mt-10 md:mx-auto md:w-[36rem]">
	<h1 class="text-2xl mb-3">MIDDLE CLASS TEXT EDITOR</h1>
	<h2 class="italic">"A rich text editor? In this economy?"</h2>
	<h2><a class="link" href="https://nicholaschen.io">By Nicholas Chen</a></h2>
	<h2><a class="link" href="https://github.com/nichwch/pseudo-wysiwyg">source</a></h2>

	<p class="mt-3">
		MIDDLE CLASS TEXT EDITOR is an editor library that sprinkles some rich text features on top of a
		plain textarea, without becoming a full fledged full-text editor, hopefully gaining most of the
		advantages without all of the costs.
	</p>
	<p class="mt-3">
		Consider something like the caption for an image on Instagram. There is more or less no rich
		text functionality, except for the ability to tag people. There's no reason you should have to
		import a huge rich text editor library to implement just that feature - it's completely doable
		with just a textarea. This is proof of that.
	</p>
	<p class="mt-3">Try changing the names recognized by @: (separate values by newline)</p>
	<textarea class=" mt-3 resize-none border border-black h-32 p-2" bind:value={nameInput} />

	<p>Try changing the projects recognized by # (separate values by newline):</p>
	<textarea class=" resize-none border border-black h-32 p-2" bind:value={projectInput} />

	<Editor
		bind:content={editorContents}
		keywordMap={{
			'@': {
				component: MentionComponent,
				recognized: names,
				allowUnrecognized: false
			},
			'#': {
				component: ProjectComponent,
				recognized: projects,
				allowUnrecognized: false
			},
			'\\*': {
				component: ProjectComponent,
				recognized: [],
				allowUnrecognized: true
			}
		}}
		splitFunc={(/** @type {string} */ text) => text.split('\n')}
	/>
	<h1 class="text-2xl mt-3">Some nice details:</h1>
	<p class="mt-3">Moving the cursor into a keyword moves it through that keyword</p>
	<p class="mt-3">
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
	</p>
	<p class="mt-3">
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
	</p>
	<p class="mt-3">
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
	</p>
	<p class="mt-3">
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
		Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing. Lorem Ipsum for scroll testing.
	</p>
</div>
