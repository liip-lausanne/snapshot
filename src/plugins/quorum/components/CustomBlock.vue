<script>
import getProvider from '@snapshot-labs/snapshot.js/src/utils/provider';
import Plugin from '../index';
import { shorten } from '@/helpers/utils';
import { useIntl } from '@/composables/useIntl';

const { formatCompactNumber, formatPercentNumber } = useIntl();

export default {
  props: ['space', 'proposal', 'results', 'loaded', 'strategies'],
  setup() {
    return { shorten, formatCompactNumber, formatPercentNumber };
  },
  data() {
    return {
      loading: false,
      plugin: new Plugin(),
      totalVotingPower: 0
    };
  },
  computed: {
    totalScore() {
      const basicCount = this.space.plugins?.quorum?.basicCount;
      if (basicCount && this.proposal.type === 'basic') {
        return this.results.scores
          .filter((score, i) => basicCount.includes(i))
          .reduce((a, b) => a + b, 0);
      }
      return this.results.scores.reduce((a, b) => a + b, 0);
    },
    quorum() {
      return this.totalVotingPower === 0
        ? 0
        : this.totalScore / this.totalVotingPower;
    }
  },

  async created() {
    this.loading = true;

    this.totalVotingPower = await this.plugin.getTotalVotingPower(
      getProvider(this.space.network, 'brovider'),
      this.space.plugins.quorum,
      this.proposal.snapshot
    );

    this.loading = false;
  }
};
</script>

<template>
  <BaseBlock title="Quorum" :loading="!loaded">
    <div class="mb-1 text-skin-link">
      <span class="mr-1">
        {{ formatCompactNumber(totalScore) }} /
        {{ formatCompactNumber(totalVotingPower) }}
        {{ shorten(proposal.symbol || space.symbol, 'symbol') }}
      </span>
      <span class="float-right" v-text="formatPercentNumber(quorum)" />
    </div>
    <ProposalResultsProgressBar :value="quorum" :max="1" class="mb-3" />
  </BaseBlock>
</template>
