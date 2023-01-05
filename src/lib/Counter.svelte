<script lang="ts">
	export let incrementAmount: number;
	let number = 0;
</script>

<!-- a rather boring widget, but a widget nonetheless -->
<button on:click={() => (number += incrementAmount)}><slot /></button> [{number}]
