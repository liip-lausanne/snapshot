<script setup lang="ts">
import { useWeb3 } from '@/composables/useWeb3';
import { useRouter } from 'vue-router';
import { useApp } from '@/composables/useApp';

const props = defineProps<{
  address: string;
}>();

const emit = defineEmits(['switchWallet']);

const { domain } = useApp();
const { logout } = useWeb3();
const router = useRouter();

function handleAction(e) {
  if (e === 'viewProfile')
    // Link to profile page, if on custom domain then the link is external
    return domain
      ? window.open(`https://snapshot.org/#/profile/${props.address}`, '_blank')
      : router.push({
          name: 'profileActivity',
          params: { address: props.address }
        });
  if (e === 'switchWallet') return emit('switchWallet');
  return logout();
}
</script>

<template>
  <div>
    <BaseDropdown
      :items="[
        { text: 'View profile', action: 'viewProfile', icon: 'profile' },
        { text: 'Switch wallet', action: 'switchWallet', icon: 'switch' },
        { text: 'Log out', action: 'logout', icon: 'logout' }
      ]"
      @select="handleAction($event)"
    >
      <template #button>
        <slot />
      </template>
      <template #item="{ item }">
        <div class="flex items-center space-x-2">
          <div class="w-[24px]">
            <i-ho-user-circle v-if="item.icon === 'profile'" />
            <i-ho-refresh v-if="item.icon === 'switch'" />
            <i-ho-logout v-if="item.icon === 'logout'" class="ml-[2px]" />
          </div>
          <div>
            {{ item.text }}
          </div>
        </div>
      </template>
    </BaseDropdown>
  </div>
</template>
