<script lang="ts">
	import { Html } from '@dfinity/gix-components';
	import { nonNullish } from '@dfinity/utils';
	import { getContext } from 'svelte';
	import type { RewardCampaignDescription } from '$env/types/env-reward';
	import RewardDateBadge from '$lib/components/rewards/RewardDateBadge.svelte';
	import Badge from '$lib/components/ui/Badge.svelte';
	import Img from '$lib/components/ui/Img.svelte';
	import { REWARDS_BANNER, REWARDS_STATUS_BUTTON } from '$lib/constants/test-ids.constants';
	import { i18n } from '$lib/stores/i18n.store';
	import {
		REWARD_ELIGIBILITY_CONTEXT_KEY,
		type RewardEligibilityContext
	} from '$lib/stores/reward.store';
	import { replacePlaceholders, resolveText } from '$lib/utils/i18n.utils';
	import { isEndedCampaign } from '$lib/utils/rewards.utils';

	interface Props {
		onclick: () => void;
		reward: RewardCampaignDescription;
		testId?: string;
	}

	let { onclick, reward, testId }: Props = $props();

	const { getCampaignEligibility } = getContext<RewardEligibilityContext>(
		REWARD_ELIGIBILITY_CONTEXT_KEY
	);

	const campaignEligibility = getCampaignEligibility(reward.id);
	const isEligible = $derived($campaignEligibility?.eligible ?? false);
	const hasEnded = $derived(isEndedCampaign(reward.endDate));
</script>

<button {onclick} class="flex flex-col" data-tid={testId}>
	<div class="-mb-7">
		<div class="max-h-66 overflow-hidden rounded-2xl">
			<Img
				src={reward.cardBanner}
				testId={REWARDS_BANNER}
				grayscale={hasEnded}
				alt={replacePlaceholders($i18n.rewards.alt.reward_banner, {
					$campaignName: resolveText({ i18n: $i18n, path: reward.cardTitle })
				})}
			/>
		</div>
	</div>

	<div class="relative rounded-lg bg-primary p-4">
		<article class="h-full">
			<section>
				<div
					class="flex flex-col-reverse items-center text-start text-lg font-semibold md:flex-row"
				>
					<div class="mr-auto flex flex-col items-center md:flex-row">
						<div>
							{resolveText({ i18n: $i18n, path: reward.cardTitle })}
						</div>
						{#if isEligible && !hasEnded}
							<span class="mr-auto inline-flex md:mx-1">
								<Badge
									variant="success"
									testId={nonNullish(testId) ? `${testId}-badge` : undefined}
								>
									{$i18n.rewards.text.youre_eligible}
								</Badge>
							</span>
						{/if}
					</div>

					<span class="mr-auto inline-flex md:ml-auto md:mr-0">
						<RewardDateBadge
							date={reward.endDate}
							testId={nonNullish(testId) ? `${testId}-date-badge` : undefined}
						/>
					</span>
				</div>

				<p class="m-0 mt-2 text-start text-xs text-tertiary">
					<Html text={resolveText({ i18n: $i18n, path: reward.oneLiner })} />
				</p>
			</section>
			<section class="bottom-4 left-4 mt-3 flex">
				<div
					data-tid={REWARDS_STATUS_BUTTON}
					class="rounded-xl bg-brand-primary px-4 py-3 font-bold text-primary-inverted"
					>{hasEnded ? $i18n.rewards.text.view_details : $i18n.rewards.text.check_status}
				</div>
			</section>
		</article>
	</div>
</button>
