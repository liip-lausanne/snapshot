<script setup lang="ts">
import { ref, computed, toRefs, watch } from 'vue';
import { useRouter } from 'vue-router';
import { clone } from '@snapshot-labs/snapshot.js/src/utils';
import { useStrategies } from '@/composables/useStrategies';
import { validateSchema } from '@snapshot-labs/snapshot.js/src/utils';
import { useNetworksFilter } from '@/composables/useNetworksFilter';
import { encodeJson } from '@/helpers/b64';

const defaultParams = {
  symbol: 'DAI',
  address: '0x6B175474E89094C44Da98b954EedeAC495271d0F',
  decimals: 18
};

const props = defineProps<{
  open: boolean;
  strategy: {
    name: string;
    network: string;
    params: Record<string, any>;
  };
  defaultNetwork?: string;
}>();

const emit = defineEmits(['add', 'close']);
const { open } = toRefs(props);
const searchInput = ref('');
const textAreaJsonIsValid = ref(true);
const loading = ref(false);
const input = ref({
  name: '',
  network: '1',
  params: {} as Record<string, any>
});

const {
  filterStrategies,
  getStrategies,
  loadingStrategies,
  getExtendedStrategy,
  extendedStrategy,
  strategyDefinition
} = useStrategies();
const strategiesResults = computed(() => filterStrategies(searchInput.value));

const { getNetworksSpacesCount } = useNetworksFilter();

const strategyValidationErrors = computed(
  () => validateSchema(strategyDefinition.value, input.value.params) ?? []
);
const strategyIsValid = computed(() =>
  strategyValidationErrors.value === true ? true : false
);

const router = useRouter();

const playgroundLink = computed(() => {
  const route = router.resolve({
    name: 'playground',
    query: {
      query: encodeJson({
        params: input.value.params,
        network: input.value.network,
        snapshot: '',
        addresses: extendedStrategy.value?.examples?.[0].addresses || []
      })
    },
    params: { name: input.value.name }
  });

  return new URL(route.href, window.location.origin).href;
});

function handleSubmit() {
  const strategyObj = clone(input.value);
  emit('add', strategyObj);
  emit('close');
}
async function initStrategy(strategyName) {
  loading.value = true;
  await getExtendedStrategy(strategyName);
}
async function selectStrategy(strategyName) {
  input.value.name = strategyName;
  await initStrategy(strategyName);
  const params =
    extendedStrategy.value?.examples?.[0]?.strategy?.params || defaultParams;
  input.value.params = strategyDefinition.value ? {} : params;
  loading.value = false;
}
async function editStrategy(strategyName) {
  input.value = props.strategy;
  await initStrategy(strategyName);
  loading.value = false;
}

watch(open, () => {
  input.value.network = props.defaultNetwork ?? '';
  // compute the spaces count for network ordering.
  getNetworksSpacesCount();
  if (props.open && !props.strategy?.name) getStrategies();
  if (props.strategy?.name) {
    editStrategy(props.strategy.name);
  } else {
    input.value = {
      name: '',
      network: props.defaultNetwork ?? '1',
      params: defaultParams
    };
  }
});
</script>

<template>
  <BaseModal :open="open" @close="$emit('close')">
    <template #header>
      <h3 v-text="input.name ? input.name : $t('settings.addStrategy')" />
    </template>
    <BaseSearch
      v-if="!strategy.name && !input.name"
      v-model="searchInput"
      :placeholder="$t('searchPlaceholder')"
      modal
    />
    <div v-if="input.name" class="m-4">
      <LoadingRow v-if="loading" class="px-0" />
      <div v-else>
        <div class="min-h-[280px] space-y-3">
          <ComboboxNetwork
            :network="input.network"
            @select="value => (input.network = value)"
          />
          <div>
            <InputObject
              v-if="strategyDefinition"
              v-model="input.params"
              :definition="strategyDefinition"
              :errors="strategyValidationErrors"
            />
            <TextareaJson
              v-else
              v-model="input.params"
              v-model:is-valid="textAreaJsonIsValid"
              :placeholder="$t('strategyParameters')"
              class="input text-left"
            />
          </div>
        </div>
      </div>
    </div>

    <div v-else class="my-4 mx-0 min-h-[300px] md:mx-4">
      <LoadingRow v-if="loadingStrategies" block />
      <div v-else class="space-y-3">
        <BaseStrategyItem
          v-for="str in strategiesResults"
          :key="str.id"
          :strategy="str"
          @click="selectStrategy(str.id)"
        />
        <BaseNoResults v-if="strategiesResults.length < 1" />
      </div>
    </div>
    <template v-if="input.name" #footer>
      <BaseLink :link="playgroundLink" hide-external-icon class="mb-2 block">
        <BaseButton class="w-full">
          {{ $t('settings.testInPlayground') }}
          <i-ho-external-link class="mb-[2px] inline-block text-xs" />
        </BaseButton>
      </BaseLink>
      <BaseButton
        :disabled="
          !textAreaJsonIsValid ||
          (strategyDefinition && !strategyIsValid) ||
          loading
        "
        class="w-full"
        primary
        @click="handleSubmit"
      >
        {{ strategy.name ? $t('save') : $t('add') }}
      </BaseButton>
    </template>
  </BaseModal>
</template>
