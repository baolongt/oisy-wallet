<script lang="ts">
	import { nonNullish } from '@dfinity/utils';
	import HideTokenModal from '$eth/components/tokens/HideTokenModal.svelte';
	import IcHideTokenModal from '$icp/components/tokens/IcHideTokenModal.svelte';
	import AddressBookModal from '$lib/components/address-book/AddressBookModal.svelte';
	import DappModalDetails from '$lib/components/dapps/DappModalDetails.svelte';
	import ReferralCodeModal from '$lib/components/referral/ReferralCodeModal.svelte';
	import RewardModal from '$lib/components/rewards/RewardModal.svelte';
	import RewardsEligibilityContext from '$lib/components/rewards/RewardsEligibilityContext.svelte';
	import SettingsModal from '$lib/components/settings/SettingsModal.svelte';
	import VipQrCodeModal from '$lib/components/vip/VipQrCodeModal.svelte';
	import { authSignedIn } from '$lib/derived/auth.derived';
	import {
		modalDAppDetails,
		modalHideToken,
		modalIcHideToken,
		modalSolHideToken,
		modalSolHideTokenData,
		modalVipQrCode,
		modalSettingsState,
		modalReferralCode,
		modalAddressBook,
		modalVipQrCodeData,
		modalIcHideTokenData,
		modalHideTokenData,
		modalRewardDetails,
		modalRewardDetailsData
	} from '$lib/derived/modal.derived';
	import SolHideTokenModal from '$sol/components/tokens/SolHideTokenModal.svelte';

	/**
	 * Modals that must be declared at the root of the layout if they are used across routes - available on navigation.
	 */
</script>

{#if $authSignedIn}
	{#if $modalHideToken}
		<HideTokenModal fromRoute={$modalHideTokenData} />
	{:else if $modalIcHideToken}
		<IcHideTokenModal fromRoute={$modalIcHideTokenData} />
	{:else if $modalSolHideToken}
		<SolHideTokenModal fromRoute={$modalSolHideTokenData} />
	{:else if $modalDAppDetails}
		<DappModalDetails />
	{:else if $modalVipQrCode && nonNullish($modalVipQrCodeData)}
		<VipQrCodeModal codeType={$modalVipQrCodeData} />
	{:else if $modalSettingsState}
		<SettingsModal />
	{:else if $modalReferralCode}
		<ReferralCodeModal />
	{:else if $modalAddressBook}
		<AddressBookModal />
	{:else if $modalRewardDetails && nonNullish($modalRewardDetailsData)}
		<RewardsEligibilityContext>
			<RewardModal reward={$modalRewardDetailsData} />
		</RewardsEligibilityContext>
	{/if}
{/if}
