<script lang="ts">
	import type { WizardStep } from '@dfinity/gix-components';
	import { isNullish } from '@dfinity/utils';
	import { createEventDispatcher, getContext, setContext } from 'svelte';
	import { writable } from 'svelte/store';
	import EthFeeContext from '$eth/components/fee/EthFeeContext.svelte';
	import EthSendForm from '$eth/components/send/EthSendForm.svelte';
	import EthSendReview from '$eth/components/send/EthSendReview.svelte';
	import { sendSteps } from '$eth/constants/steps.constants';
	import { send as executeSend } from '$eth/services/send.services';
	import {
		ETH_FEE_CONTEXT_KEY,
		type EthFeeContext as FeeContextType,
		initEthFeeContext,
		initEthFeeStore
	} from '$eth/stores/eth-fee.store';
	import type { EthereumNetwork } from '$eth/types/network';
	import type { ProgressStep } from '$eth/types/send';
	import { shouldSendWithApproval } from '$eth/utils/send.utils';
	import { isErc20Icp } from '$eth/utils/token.utils';
	import { assertCkEthMinterInfoLoaded } from '$icp-eth/services/cketh.services';
	import { ckEthMinterInfoStore } from '$icp-eth/stores/cketh.store';
	import { toCkErc20HelperContractAddress } from '$icp-eth/utils/cketh.utils';
	import { mapAddressStartsWith0x } from '$icp-eth/utils/eth.utils';
	import ButtonBack from '$lib/components/ui/ButtonBack.svelte';
	import InProgressWizard from '$lib/components/ui/InProgressWizard.svelte';
	import {
		TRACK_COUNT_ETH_SEND_ERROR,
		TRACK_COUNT_ETH_SEND_SUCCESS
	} from '$lib/constants/analytics.contants';
	import { ethAddress } from '$lib/derived/address.derived';
	import { authIdentity } from '$lib/derived/auth.derived';
	import { exchanges } from '$lib/derived/exchange.derived';
	import { WizardStepsSend } from '$lib/enums/wizard-steps';
	import { trackEvent } from '$lib/services/analytics.services';
	import { i18n } from '$lib/stores/i18n.store';
	import { SEND_CONTEXT_KEY, type SendContext } from '$lib/stores/send.store';
	import { toastsError } from '$lib/stores/toasts.store';
	import type { ContactUi } from '$lib/types/contact';
	import type { OptionAmount } from '$lib/types/send';
	import type { Token, TokenId } from '$lib/types/token';
	import { invalidAmount, isNullishOrEmpty } from '$lib/utils/input.utils';
	import { parseToken } from '$lib/utils/parse.utils';

	export let currentStep: WizardStep | undefined;

	/**
	 * Send context store
	 */

	const { sendTokenDecimals, sendTokenId, sendToken } = getContext<SendContext>(SEND_CONTEXT_KEY);

	/**
	 * Props
	 */

	export let destination = '';
	export let sourceNetwork: EthereumNetwork;
	export let amount: OptionAmount = undefined;
	export let sendProgressStep: string;
	export let selectedContact: ContactUi | undefined = undefined;
	// Required for the fee and also to retrieve ck minter information.
	// i.e. Ethereum or Sepolia "main" token.
	export let nativeEthereumToken: Token;

	let sendWithApproval: boolean;
	$: sendWithApproval = shouldSendWithApproval({
		to: destination,
		tokenId: $sendTokenId,
		erc20HelperContractAddress: toCkErc20HelperContractAddress(
			$ckEthMinterInfoStore?.[nativeEthereumToken.id]
		)
	});

	/**
	 * Fee context store
	 */

	const feeStore = initEthFeeStore();

	const feeSymbolStore = writable<string | undefined>(undefined);
	$: feeSymbolStore.set(nativeEthereumToken.symbol);

	const feeTokenIdStore = writable<TokenId | undefined>(undefined);
	$: feeTokenIdStore.set(nativeEthereumToken.id);

	const feeDecimalsStore = writable<number | undefined>(undefined);
	$: feeDecimalsStore.set(nativeEthereumToken.decimals);

	const feeExchangeRateStore = writable<number | undefined>(undefined);
	$: feeExchangeRateStore.set($exchanges?.[nativeEthereumToken.id]?.usd);

	let feeContext: EthFeeContext | undefined;
	const evaluateFee = () => feeContext?.triggerUpdateFee();

	setContext<FeeContextType>(
		ETH_FEE_CONTEXT_KEY,
		initEthFeeContext({
			feeStore,
			feeSymbolStore,
			feeTokenIdStore,
			feeDecimalsStore,
			feeExchangeRateStore,
			evaluateFee
		})
	);

	/**
	 * Send
	 */

	const dispatch = createEventDispatcher();

	const send = async () => {
		if (isNullishOrEmpty(destination)) {
			toastsError({
				msg: { text: $i18n.send.assertion.destination_address_invalid }
			});
			return;
		}

		if (invalidAmount(amount) || isNullish(amount)) {
			toastsError({
				msg: { text: $i18n.send.assertion.amount_invalid }
			});
			return;
		}

		if (isNullish($feeStore)) {
			toastsError({
				msg: { text: $i18n.send.assertion.gas_fees_not_defined }
			});
			return;
		}

		const { valid } = assertCkEthMinterInfoLoaded({
			minterInfo: $ckEthMinterInfoStore?.[nativeEthereumToken.id]
		});

		if (!valid) {
			return;
		}

		// https://github.com/ethers-io/ethers.js/discussions/2439#discussioncomment-1857403
		const { maxFeePerGas, maxPriorityFeePerGas, gas } = $feeStore;

		// https://docs.ethers.org/v5/api/providers/provider/#Provider-getFeeData
		// exceeds block gas limit
		if (isNullish(maxFeePerGas) || isNullish(maxPriorityFeePerGas)) {
			toastsError({
				msg: { text: $i18n.send.assertion.max_gas_fee_per_gas_undefined }
			});
			return;
		}

		// Unexpected errors
		if (isNullish($ethAddress)) {
			toastsError({
				msg: { text: $i18n.send.assertion.address_unknown }
			});
			return;
		}

		dispatch('icNext');

		try {
			await executeSend({
				from: $ethAddress,
				to: isErc20Icp($sendToken) ? destination : mapAddressStartsWith0x(destination),
				progress: (step: ProgressStep) => (sendProgressStep = step),
				token: $sendToken,
				amount: parseToken({
					value: `${amount}`,
					unitName: $sendTokenDecimals
				}),
				maxFeePerGas,
				maxPriorityFeePerGas,
				gas,
				sourceNetwork,
				identity: $authIdentity,
				minterInfo: $ckEthMinterInfoStore?.[nativeEthereumToken.id]
			});

			trackEvent({
				name: TRACK_COUNT_ETH_SEND_SUCCESS,
				metadata: {
					token: $sendToken.symbol
				}
			});

			setTimeout(() => close(), 750);
		} catch (err: unknown) {
			trackEvent({
				name: TRACK_COUNT_ETH_SEND_ERROR,
				metadata: {
					token: $sendToken.symbol
				}
			});

			toastsError({
				msg: { text: $i18n.send.error.unexpected },
				err
			});

			dispatch('icBack');
		}
	};

	const close = () => dispatch('icClose');
	const back = () => dispatch('icSendBack');
</script>

<EthFeeContext
	bind:this={feeContext}
	sendToken={$sendToken}
	sendTokenId={$sendTokenId}
	{amount}
	{destination}
	observe={currentStep?.name !== WizardStepsSend.SENDING}
	{sourceNetwork}
	{nativeEthereumToken}
>
	{#if currentStep?.name === WizardStepsSend.REVIEW}
		<EthSendReview on:icBack on:icSend={send} {destination} {selectedContact} {amount} />
	{:else if currentStep?.name === WizardStepsSend.SENDING}
		<InProgressWizard
			progressStep={sendProgressStep}
			steps={sendSteps({ i18n: $i18n, sendWithApproval })}
		/>
	{:else if currentStep?.name === WizardStepsSend.SEND}
		<EthSendForm
			on:icNext
			on:icClose={close}
			on:icBack
			on:icTokensList
			{selectedContact}
			bind:destination
			bind:amount
			{nativeEthereumToken}
		>
			<ButtonBack onclick={back} slot="cancel" />
		</EthSendForm>
	{:else}
		<slot />
	{/if}
</EthFeeContext>
