<script lang="ts">
	import type { Snippet } from 'svelte';
	import type { ButtonColorStyle } from '$lib/types/style';

	interface Props {
		children: Snippet;
		onClick?: () => void;
		href: string;
		ariaLabel: string;
		colorStyle?: ButtonColorStyle;
	}

	let { children, onClick, href, ariaLabel, colorStyle = 'tertiary' }: Props = $props();
</script>

<a
	{href}
	rel="external noopener noreferrer"
	target="_blank"
	class={`as-button ${colorStyle} icon flex flex-col text-center text-xs font-normal`}
	onclick={onClick}
	aria-label={ariaLabel}
>
	{@render children()}
</a>
