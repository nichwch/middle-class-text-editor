<script lang="ts">
	import { Editor } from '$lib';
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

<div class="w-full px-5 my-20 md:mx-auto md:w-[36rem]">
	<h1 class="text-2xl mb-3">MIDDLE CLASS TEXT EDITOR</h1>
	<h2 class="italic">"A rich text editor? In this economy?"</h2>
	<h2><a class="link" href="https://nicholaschen.io">By Nicholas Chen</a></h2>
	<h2><a class="link" href="https://github.com/nichwch/pseudo-wysiwyg">source</a></h2>

	<p class="mt-3">
		MIDDLE CLASS TEXT EDITOR is an editor library that sprinkles some rich text features on top of a
		plain textarea without becoming a full fledged full-text editor, hopefully gaining most of the
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
	<p class="my-3">
		Type @ to mention a person, type # to mention a project. Hover over a person or project for more
		information.
	</p>
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
	/>
	<h1 class="text-2xl mt-5">Some nice details:</h1>
	<p class="mt-3">Moving the cursor into a keyword moves it through that keyword</p>
	<p class="mt-3">
		Dashes are automatically replaced with non-breaking dashes in keywords. This prevents the
		styling from falling apart.
	</p>
	<p class="mt-3">From what I can tell, it works reasonably well on mobile.</p>
	<h1 class="text-2xl mt-5">Some things that don't quite work:</h1>
	<p class="mt-3">
		You can still get the styling to break by piling a bunch of keywords onto a line. This results
		in a pretty localized styling failure that doesn't screw up the entire overlay, and it's also
		not likely to happen with actual user input, so I'm fine with it.
	</p>
	<img
		src="bug.png"
		class="w-full border border-black mt-2"
		alt="the bug where keywords pile up on a line"
	/>
	<p class="mt-3">Undo doesn't work after deleting a keyword. This is really annoying.</p>
	<p class="mt-3"></p>

	<h1 class="text-2xl mt-5">Is MIDDLE CLASS TEXT EDITOR right for you?</h1>
	<p class="mt-3">
		This library is powerful, but it has some limitations. Because of the overlay styling strategy
		(explained below), it's not possible to offer bold, italics, or headers. Adding images and links
		is theoretically possible but a little involved.
	</p>
	<p class="mt-3">
		However, there are lots of places on the web where you don't need rich text functionality beyond
		mentions. Instagram photo captions and Slack messages are two good examples of this - Insta
		captions don't offer rich text at all, while Slack's message editor offers formatting, but is
		rarely ever used. In both of these cases, the only "rich text" feature that's used is mentions.
		For these kinds of scenarios, this library is a perfect fit.
	</p>
	<h1 class="text-2xl mt-5">Use MIDDLE CLASS TEXT EDITOR in your project:</h1>
	<p class="mt-3">
		It's small enough that you can just copy paste the component into your project, <a
			class="link"
			href="https://ui.shadcn.com/">ShadCN style</a
		>. The only external dependency is
		<a class="link" href="https://www.npmjs.com/package/@floating-ui/dom">@floating-ui/dom</a>, used
		for the autocomplete popup window.
	</p>
	<p class="mt-3">
		<a
			class="link"
			href="https://github.com/nichwch/middle-class-text-editor/blob/main/src/lib/Editor.svelte"
			>You can find the source for the component here, in the repo.</a
		>
	</p>
	<h1 class="text-2xl mt-5">API Specification</h1>
	<p class="mt-3">The component takes two props:</p>
	<p class="mt-3"><b>content:</b> The contents of the textarea.</p>
	<p class="mt-3">
		<b>keywordMap:</b> An object, where the keys are regex matches for "keyword" and the values are svelte
		components that will be rendered over those keywords. In the example above, the @mentions and #projects
		were keywords.
	</p>
	<p class="mt-3">
		The keyword components accept two props: index, and focusedIndex. index is the order in which
		the keyword appears on the page, and focusedIndex is the index of the keyword overlapping the
		caret. This is helpful if you want to highlight keywords when the user moves their caret through
		one.
	</p>
	<h1 class="text-2xl mt-5">How does it work?</h1>
	<p class="mt-3">Great question! Short answer: By abusing absolute positioning.</p>
	<p class="mt-3">
		Long answer: I use three elements positioned absolutely so that they're all overlapping: an
		"underlay div", the textarea, and an "overlay div".
	</p>
	<p class="mt-3">
		The textarea is where the user types. As they type, the underlay clones the content one for one,
		but renders keywords differently using a regex match.
	</p>
	<p class="mt-3">
		The overlay renders all the keywords (again, using a regex match), and positions them absolutely
		to match their corresponding keyword in the underlay. I use a separate overlay instead of
		rendering keywords directly in the underlay because otherwise the z-index and stacking context
		would prevent the keywords from being clickable.
	</p>
	<p class="mt-3">Here's a handy visualization of the setup.</p>
	<img
		src="explanation.jpg"
		class="mt-2 w-full border border-black"
		alt="An visualization of the underlay, textarea, overlay setup."
	/>
	<p class="mt-3">
		If you got this far and liked my project, <a
			class="link"
			href="https://github.com/nichwch/middle-class-text-editor"
			>consider giving it a star on Github.</a
		>
	</p>
</div>
