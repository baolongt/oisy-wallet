<script lang="ts">
	import { Popover } from '@dfinity/gix-components';
	import AboutWhyOisy from '$lib/components/about/AboutWhyOisy.svelte';
	import IconGitHub from '$lib/components/icons/IconGitHub.svelte';
	import IconHelpCircle from '$lib/components/icons/IconHelpCircle.svelte';
	import ChangelogLink from '$lib/components/navigation/ChangelogLink.svelte';
	import DocumentationLink from '$lib/components/navigation/DocumentationLink.svelte';
	import SupportLink from '$lib/components/navigation/SupportLink.svelte';
	import ButtonIcon from '$lib/components/ui/ButtonIcon.svelte';
	import Hr from '$lib/components/ui/Hr.svelte';
	import { USER_MENU_ROUTE } from '$lib/constants/analytics.contants';
	import { OISY_REPO_URL } from '$lib/constants/oisy.constants';
	import {
		NAVIGATION_MENU_BUTTON,
		NAVIGATION_MENU,
		NAVIGATION_MENU_SUPPORT_BUTTON,
		NAVIGATION_MENU_DOC_BUTTON
	} from '$lib/constants/test-ids.constants';
	import { i18n } from '$lib/stores/i18n.store';

	let visible = $state(false);

	let button = $state<HTMLButtonElement | undefined>();

	const hidePopover = () => (visible = false);
</script>

<ButtonIcon
	bind:button
	onclick={() => (visible = true)}
	ariaLabel={$i18n.navigation.alt.menu}
	testId={NAVIGATION_MENU_BUTTON}
	colorStyle="tertiary-alt"
	link={false}
>
	{#snippet icon()}
		<IconHelpCircle size="22" />
	{/snippet}
	{$i18n.navigation.alt.menu}
</ButtonIcon>

<Popover bind:visible anchor={button} direction="rtl">
	<div
		class="mb-1 flex max-w-80 flex-col gap-1"
		data-tid={NAVIGATION_MENU}
		onclick={hidePopover}
		role="none"
	>
		<AboutWhyOisy
			asMenuItem
			asMenuItemCondensed
			onIcOpenAboutModal={hidePopover}
			trackEventSource={USER_MENU_ROUTE}
		/>

		<DocumentationLink
			asMenuItem
			asMenuItemCondensed
			trackEventSource={USER_MENU_ROUTE}
			testId={NAVIGATION_MENU_DOC_BUTTON}
		/>

		<SupportLink asMenuItem asMenuItemCondensed testId={NAVIGATION_MENU_SUPPORT_BUTTON} />

		<Hr />

		<a
			href={OISY_REPO_URL}
			rel="external noopener noreferrer"
			target="_blank"
			class="nav-item nav-item-condensed"
			aria-label={$i18n.navigation.text.source_code_on_github}
		>
			<IconGitHub />
			{$i18n.navigation.text.source_code}
		</a>

		<ChangelogLink asMenuItem asMenuItemCondensed trackEventSource={USER_MENU_ROUTE} />
	</div>
</Popover>
