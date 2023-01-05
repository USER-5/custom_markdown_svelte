<script lang="ts">
	import CustomMarkdown from '../lib/CustomMarkdown.svelte';
	import { marked } from 'marked';

	let markdownInput: string = `# This is my cool page
look at how it's formatted in **markdown** with a few styles
* Even lists
* Work __somewhat__

**Except** you can define your own elements, too: {counter|Hey there!|2} {counter|Inc by ten|10}`;

	// Define our own token (adapted from marked documentation)
	const counter = {
		name: 'counter',
		level: 'inline', // Is this a block-level or inline-level tokenizer?
		start(src: any) {
			return src.match(/{counter\|/)?.index;
		}, // Hint to Marked.js to stop and check for a match
		tokenizer(src: any, tokens: any) {
			const rule = /^{counter\|([^\|}]*)\|([0-9]*)}/; // Regex for the complete token, anchor to string start
			const match = rule.exec(src);
			if (match) {
				const token = {
					// Token to generate
					type: 'counter', // Should match "name" above
					raw: match[0], // Text to consume from the source
					text: match[1].trim(), // Additional custom properties
					incrementAmount: parseInt(match[2])
				};
				return token;
			}
		},
		// we don't use this, but the marked parser does. Not a bad idea to have some fallback...
		renderer(token: any) {
			return `"default render for a counter"`; // parseInline to turn child tokens into HTML
		}
	};

	marked.use({ extensions: [counter] });

	setTimeout(() => ast = marked.lexer(markdownInput));

	$: ast = marked.lexer(markdownInput);

	$: raw = marked.parser(ast);

</script>

<p>Utilising the marked AST in order to extend it with whatever components we want.</p>
<p>Only a very small subsection of markdown is implemented (the rest should be trivial), and a single custom component: a counter which increments a certain amount each time it's clicked - but converted from markdown at <strong>runtime</strong></p>

<textarea bind:value={markdownInput} />

<h2>Custom Formatted</h2>
<CustomMarkdown {ast} />

<h2>AST representation</h2>
<pre>{JSON.stringify(ast, null, 1)}</pre>

<h2>Default Formatter (mostly works, except can only render pure HTML - no svelte or framework)</h2>
{@html raw}

<style>
	textarea {
		width: 100%;
		height: 10em;
	}
	h2 {
		background-color: black;
		color: white;
	}
	pre {
		max-height: 20em;
		background-color: #eee;
		overflow-y: auto;
	}
</style>
