<script lang="ts">
	import { marked } from 'marked';
	import unescape from 'unescape';
	import Counter from './Counter.svelte';

	export let ast: marked.TokensList;
</script>

<!-- subset of actual renderer, just for proof of concept.
		We could also change the default rendering of nodes, or adjust styling as we like. -->

<!-- This is a recursive structure -->
{#each ast as node}
	<!-- Find the leaf nodes first -->
	{#if node.type === 'text'}
		{#if Object.hasOwn(node, 'tokens')}
			<!-- text *can* have tokens as well? in which case, recurse...-->
			<svelte:self ast={node.tokens} />
		{:else}
			<!-- marked escapes characters like ',
		     and svelte will *helpfully* escape THOSE escape sequences
				 (so you end up with &#39; instead). This undoes one level of escape
				 and lets special characters print normally -->
			{unescape(node.text)}
		{/if}
	{:else if node.type === 'space'}
		<!-- ... do nothing -->

		<!-- our cool CUSTOM element. Typescript doesn't like it since 'counter' isn't really in the type... -->
	{:else if node.type === 'counter'}
		<Counter incrementAmount={node.incrementAmount}>{node.text}</Counter>

		<!-- Recursive nodes -->
	{:else if node.type === 'heading'}
		<h1><svelte:self ast={node.tokens} /></h1>
	{:else if node.type === 'paragraph'}
		<p><svelte:self ast={node.tokens} /></p>
	{:else if node.type === 'strong'}
		<strong><svelte:self ast={node.tokens} /></strong>
	{:else if node.type === 'list'}
		<!-- lists have two types -->
		{#if node.ordered}
			<ol><svelte:self ast={node.items} /></ol>
		{:else}
			<ul><svelte:self ast={node.items} /></ul>
		{/if}
	{:else if node.type === 'list_item'}
		<li><svelte:self ast={node.tokens} /></li>
	{/if}
{/each}
