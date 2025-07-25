<script lang="ts">
	import { Modal } from '@dfinity/gix-components';
	import { isNullish } from '@dfinity/utils';
	import { GLDT_IC_DATA } from '$env/networks/networks.icrc.env';
	import { icrcTokens } from '$icp/derived/icrc.derived';
	import { loadCustomTokens } from '$icp/services/icrc.services';
	import { setCustomToken } from '$icp-eth/services/custom-token.services';
	import failedVipReward from '$lib/assets/failed-vip-reward.svg';
	import successfulBinanceReward from '$lib/assets/successful-binance-reward.svg';
	import successfulVipReward from '$lib/assets/successful-vip-reward.svg';
	import Sprinkles from '$lib/components/sprinkles/Sprinkles.svelte';
	import Button from '$lib/components/ui/Button.svelte';
	import ContentWithToolbar from '$lib/components/ui/ContentWithToolbar.svelte';
	import ImgBanner from '$lib/components/ui/ImgBanner.svelte';
	import { VIP_STATE_BUTTON, VIP_STATE_IMAGE_BANNER } from '$lib/constants/test-ids.constants';
	import { authIdentity } from '$lib/derived/auth.derived';
	import { enabledIcTokens } from '$lib/derived/tokens.derived';
	import { QrCodeType } from '$lib/enums/qr-code-types';
	import { nullishSignOut } from '$lib/services/auth.services';
	import { autoLoadSingleToken } from '$lib/services/token.services';
	import { i18n } from '$lib/stores/i18n.store';
	import { modalStore } from '$lib/stores/modal.store';
	import { replaceOisyPlaceholders } from '$lib/utils/i18n.utils';

	interface Props {
		isSuccessful: boolean;
		codeType?: QrCodeType;
	}

	let { isSuccessful, codeType = QrCodeType.VIP }: Props = $props();

	const goldToken = $derived(
		$enabledIcTokens.find((token) => token.ledgerCanisterId === GLDT_IC_DATA?.ledgerCanisterId)
	);

	const enableGldtToken = async () => {
		if (isNullish($authIdentity)) {
			await nullishSignOut();
			return;
		}

		const token = $icrcTokens.find(
			({ ledgerCanisterId }) => ledgerCanisterId === GLDT_IC_DATA?.ledgerCanisterId
		);

		await autoLoadSingleToken({
			token,
			identity: $authIdentity,
			setToken: setCustomToken,
			loadTokens: loadCustomTokens,
			errorMessage: $i18n.init.error.icrc_custom_token
		});
	};

	const close = async () => {
		if (codeType === QrCodeType.GOLD && isNullish(goldToken)) {
			await enableGldtToken();
		}

		modalStore.close();
	};
</script>

{#if isSuccessful}
	<Sprinkles />
{/if}

<Modal on:nnsClose={close}>
	<svelte:fragment slot="title">
		<span class="text-xl"
			>{isSuccessful
				? $i18n.vip.reward.text.title_successful
				: $i18n.vip.reward.text.title_failed}</span
		>
	</svelte:fragment>

	<ContentWithToolbar>
		<ImgBanner
			src={isSuccessful
				? codeType === QrCodeType.VIP
					? successfulVipReward
					: successfulBinanceReward
				: failedVipReward}
			styleClass="aspect-auto"
			testId={VIP_STATE_IMAGE_BANNER}
		/>

		<h3 class="my-3 text-center"
			>{isSuccessful
				? replaceOisyPlaceholders($i18n.vip.reward.text.reward_received)
				: $i18n.vip.reward.text.reward_failed}</h3
		>
		<span class="block w-full text-center"
			>{isSuccessful
				? $i18n.vip.reward.text.reward_received_description
				: $i18n.vip.reward.text.reward_failed_description}</span
		>

		{#snippet toolbar()}
			<Button
				paddingSmall
				colorStyle="secondary-light"
				type="button"
				fullWidth
				onclick={close}
				testId={VIP_STATE_BUTTON}
			>
				{isSuccessful ? $i18n.vip.reward.text.open_wallet : $i18n.vip.reward.text.open_wallet}
			</Button>
		{/snippet}
	</ContentWithToolbar>
</Modal>
