<script setup>
import { computed, onMounted } from 'vue';
import { useApp } from '@/composables/useApp';
import aliases from '@/../snapshot-spaces/spaces/aliases.json';
import { useRouter, useRoute } from 'vue-router';
import { formatSpace } from '@/helpers/utils';
import { useExtendedSpaces } from '@/composables/useExtendedSpaces';

const route = useRoute();
const router = useRouter();
const { domain } = useApp();
const aliasedSpace = aliases[domain] || aliases[route.params.key];
const { loadExtentedSpaces, extentedSpaces } = useExtendedSpaces();

// Redirect the user to the ENS address if the space is aliased.
if (aliasedSpace) {
  const updatedPath = route.path.replace(
    domain || route.params.key,
    aliasedSpace
  );
  router.replace({ path: updatedPath });
}

const spaceKey = computed(() => aliasedSpace || domain || route.params.key);
const space = computed(() =>
  formatSpace(extentedSpaces.value?.find(s => s.id === spaceKey.value))
);

const sourceSpaceRoute = computed(() => route.params.sourceSpace);
const sourceSpace = computed(() =>
  formatSpace(extentedSpaces.value?.find(s => s.id === sourceSpaceRoute.value))
);

onMounted(async () => {
  await loadExtentedSpaces([spaceKey.value, sourceSpaceRoute.value]);
  if (!space.value) {
    router.push('/');
  }
});
</script>

<template>
  <router-view v-if="space" :space="space" :source-space="sourceSpace" />
  <div v-else>
    <!-- Lazy loading skeleton for space page with left sidebar layout -->
    <TheLayout
      v-if="
        $route.name === 'spaceProposals' ||
        $route.name === 'spaceAbout' ||
        $route.name === 'spaceTreasury'
      "
    >
      <template #sidebar-left>
        <SpaceSidebarSkeleton />
      </template>
      <template #content-right>
        <LoadingRow block />
      </template>
    </TheLayout>

    <!-- Default page loading for none sidebar left layout space pages -->
    <TheLayout v-else class="!px-4">
      <template #content-left>
        <LoadingPage />
      </template>
    </TheLayout>
  </div>
</template>
