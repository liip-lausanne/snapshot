<script setup>
import { ref, computed, toRefs, watch } from 'vue';
import { useValidationsFilter } from '@/composables/useValidationsFilter';

const defaultParams = {};

const props = defineProps({ open: Boolean, validation: Object });

const emit = defineEmits(['add', 'close']);

const { open } = toRefs(props);

const searchInput = ref('');
const isValid = ref(true);
const input = ref({
  name: '',
  params: defaultParams
});

const { filterValidations, getValidationsSpacesCount, loadingValidations } =
  useValidationsFilter();
const validations = computed(() => filterValidations(searchInput.value));

watch(
  () => props.open,
  () => {
    if (props.open) getValidationsSpacesCount();
  }
);

function select(n) {
  input.value.name = n;
}

function handleSubmit() {
  emit('add', input.value);
  emit('close');
}

watch(open, () => {
  input.value.name = '';
  if (props.validation?.params) {
    input.value.params = props.validation.params;
  } else {
    input.value = {
      name: '',
      params: defaultParams
    };
  }
});
</script>

<template>
  <BaseModal :open="open" @close="$emit('close')">
    <template #header>
      <h3>
        {{
          input.name
            ? $t('settings.editValidation')
            : $t('settings.selectValidation')
        }}
      </h3>
    </template>
    <BaseSearch
      v-if="!input.name"
      v-model="searchInput"
      :placeholder="$t('searchPlaceholder')"
      modal
    />
    <div class="my-4 mx-0 min-h-[339px] md:mx-4">
      <div v-if="input.name" class="mb-4 rounded-md border p-4 text-skin-link">
        <h4 class="mb-3 text-center" v-text="input.name" />
        <TextareaJson
          v-model="input.params"
          v-model:is-valid="isValid"
          :placeholder="$t('settings.validationParameters')"
          class="input text-left"
        />
      </div>
      <div v-if="!input.name">
        <LoadingRow v-if="loadingValidations" block />
        <div v-else class="space-y-3">
          <BaseValidationItem
            v-for="valId in validations"
            :key="valId"
            :validation="valId"
            @click="select(valId)"
          />

          <BaseNoResults v-if="Object.keys(validations).length < 1" />
        </div>
      </div>
    </div>
    <template v-if="input.name" #footer>
      <BaseButton
        :disabled="!isValid"
        class="w-full"
        primary
        @click="handleSubmit"
      >
        {{ validation.name ? $t('save') : $t('add') }}
      </BaseButton>
    </template>
  </BaseModal>
</template>
