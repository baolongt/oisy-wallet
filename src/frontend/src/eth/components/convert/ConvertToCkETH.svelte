<script lang="ts">
	import { isNullish, nonNullish } from '@dfinity/utils';
	import FeeStoreContext from '$eth/components/fee/FeeStoreContext.svelte';
	import { ethereumToken, ethereumTokenId } from '$eth/derived/token.derived';
	import type { IcCkToken } from '$icp/types/ic-token';
	import ConvertETH from '$icp-eth/components/convert/ConvertETH.svelte';
	import ConvertModal from '$lib/components/convert/ConvertModal.svelte';
	import IconCkConvert from '$lib/components/icons/IconCkConvert.svelte';
	import { modalConvertToTwinTokenCkEth } from '$lib/derived/modal.derived';
	import { pageToken } from '$lib/derived/page-token.derived';
	import { tokens } from '$lib/derived/tokens.derived';
	import { i18n } from '$lib/stores/i18n.store';
	import { findTwinToken } from '$lib/utils/token.utils';

	let ckEthToken: IcCkToken | undefined;
	$: (() => {
		if (nonNullish(ckEthToken) || isNullish($pageToken)) {
			return;
		}

		ckEthToken = findTwinToken({
			tokenToPair: $pageToken,
			tokens: $tokens
		});
	})();
</script>

<ConvertETH nativeTokenId={$ethereumTokenId} ariaLabel={$i18n.convert.text.convert_to_cketh}>
	<IconCkConvert size="24" slot="icon" />
	<span>{$ethereumToken.twinTokenSymbol ?? ''}</span>
</ConvertETH>

{#if $modalConvertToTwinTokenCkEth && nonNullish(ckEthToken)}
	<FeeStoreContext token={$ethereumToken}>
		<ConvertModal sourceToken={$ethereumToken} destinationToken={ckEthToken} />
	</FeeStoreContext>
{/if}
