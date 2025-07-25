<script lang="ts">
	import { Html, Modal } from '@dfinity/gix-components';
	import { nonNullish } from '@dfinity/utils';
	import DappTags from '$lib/components/dapps/DappTags.svelte';
	import IconGitHub from '$lib/components/icons/IconGitHub.svelte';
	import IconOpenChat from '$lib/components/icons/IconOpenChat.svelte';
	import IconTelegram from '$lib/components/icons/IconTelegram.svelte';
	import IconTwitter from '$lib/components/icons/IconTwitter.svelte';
	import ContentWithToolbar from '$lib/components/ui/ContentWithToolbar.svelte';
	import ExternalLink from '$lib/components/ui/ExternalLink.svelte';
	import ExternalLinkIcon from '$lib/components/ui/ExternalLinkIcon.svelte';
	import ImgBanner from '$lib/components/ui/ImgBanner.svelte';
	import Logo from '$lib/components/ui/Logo.svelte';
	import { TRACK_COUNT_DAPP_MODAL_OPEN_HYPERLINK } from '$lib/constants/analytics.contants';
	import { i18n } from '$lib/stores/i18n.store';
	import { modalStore } from '$lib/stores/modal.store';
	import type { OisyDappDescription } from '$lib/types/dapp-description';
	import type { Option } from '$lib/types/utils';
	import { replacePlaceholders, resolveText } from '$lib/utils/i18n.utils';

	export let dAppDescription: OisyDappDescription;
	$: ({
		id: dappId,
		website,
		screenshots,
		twitter,
		github,
		tags,
		name: dAppName,
		description,
		logo,
		callToAction,
		telegram,
		openChat
	} = dAppDescription);

	let websiteURL: Option<URL>;
	$: try {
		// TODO: use URL.parse
		websiteURL = new URL(website);
	} catch (_err: unknown) {
		websiteURL = null;
	}
</script>

<Modal on:nnsClose={modalStore.close}>
	<svelte:fragment slot="title">
		<span class="text-center text-xl">{resolveText({ i18n: $i18n, path: dAppName })}</span>
	</svelte:fragment>

	<ContentWithToolbar>
		<div class="flex flex-col gap-6">
			{#if nonNullish(screenshots) && screenshots.length > 0}
				<div class="overflow-hidden rounded-3xl">
					<ImgBanner
						styleClass="max-h-64"
						src={screenshots[0]}
						alt={replacePlaceholders($i18n.dapps.alt.website, {
							$dAppName: resolveText({ i18n: $i18n, path: dAppName })
						})}
					/>
				</div>
			{/if}

			<article>
				<div
					class="flex flex-wrap items-center justify-start gap-x-4 gap-y-2 border-b border-tertiary pb-2 sm:gap-4 sm:pb-4"
				>
					<Logo
						size="md"
						src={logo}
						alt={replacePlaceholders($i18n.dapps.alt.logo, {
							$dAppName: resolveText({ i18n: $i18n, path: dAppName })
						})}
					/>
					<div class="mr-auto">
						<div class="text-lg font-bold">{resolveText({ i18n: $i18n, path: dAppName })}</div>
						{#if nonNullish(websiteURL)}
							<ExternalLink
								iconVisible={false}
								ariaLabel={replacePlaceholders($i18n.dapps.text.open_dapp, {
									$dAppName: resolveText({ i18n: $i18n, path: dAppName })
								})}
								href={websiteURL.toString()}
								styleClass="text-sm text-tertiary">{websiteURL.hostname}</ExternalLink
							>
						{/if}
					</div>
					<div class="flex space-x-3">
						{#if nonNullish(telegram)}
							<ExternalLinkIcon
								href={telegram}
								ariaLabel={replacePlaceholders($i18n.dapps.alt.open_telegram, {
									$dAppName: resolveText({ i18n: $i18n, path: dAppName })
								})}
							>
								<IconTelegram size="22" />
							</ExternalLinkIcon>
						{/if}
						{#if nonNullish(openChat)}
							<ExternalLinkIcon
								href={openChat}
								ariaLabel={replacePlaceholders($i18n.dapps.alt.open_open_chat, {
									$dAppName: resolveText({ i18n: $i18n, path: dAppName })
								})}
							>
								<IconOpenChat size="22" />
							</ExternalLinkIcon>
						{/if}
						{#if nonNullish(twitter)}
							<ExternalLinkIcon
								href={twitter}
								ariaLabel={replacePlaceholders($i18n.dapps.alt.open_twitter, {
									$dAppName: resolveText({ i18n: $i18n, path: dAppName })
								})}
							>
								<IconTwitter size="22" />
							</ExternalLinkIcon>
						{/if}
						{#if nonNullish(github)}
							<ExternalLinkIcon
								href={github}
								ariaLabel={replacePlaceholders($i18n.dapps.alt.source_code_on_github, {
									$dAppName: resolveText({ i18n: $i18n, path: dAppName })
								})}
							>
								<IconGitHub size="22" />
							</ExternalLinkIcon>
						{/if}
					</div>
				</div>

				<p class="m-0 my-4 text-sm [&_ul]:list-disc [&_ul]:pl-6">
					<Html text={resolveText({ i18n: $i18n, path: description })} />
				</p>
				<DappTags {dAppName} {tags} />
			</article>
		</div>

		{#snippet toolbar()}
			{#if nonNullish(websiteURL)}
				<ExternalLink
					ariaLabel={replacePlaceholders($i18n.dapps.alt.open_dapp, {
						$dAppName: resolveText({ i18n: $i18n, path: dAppName })
					})}
					asButton
					fullWidth
					styleClass="primary padding-sm flex-1"
					href={websiteURL.toString()}
					iconAsLast
					trackEvent={{ name: TRACK_COUNT_DAPP_MODAL_OPEN_HYPERLINK, metadata: { dappId } }}
					>{callToAction
						? resolveText({ i18n: $i18n, path: callToAction })
						: replacePlaceholders($i18n.dapps.text.open_dapp, {
								$dAppName: resolveText({ i18n: $i18n, path: dAppName })
							})}</ExternalLink
				>
			{/if}
		{/snippet}
	</ContentWithToolbar>
</Modal>
