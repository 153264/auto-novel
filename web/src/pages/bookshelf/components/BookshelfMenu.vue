<script lang="ts" setup>
import { MenuOption } from 'naive-ui';

import { Locator } from '@/data';

import BookshelfMenuItem from './BookshelfMenuItem.vue';
import { useBookshelfStore } from '../BookshelfStore';

const message = useMessage();

const { isSignedIn } = Locator.userDataRepository();

const store = useBookshelfStore();

onMounted(async () => {
  try {
    await store.loadFavoredList();
  } catch (e) {
    message.error(`获取收藏列表失败：${e}`);
  }
});

const menuOption = (
  type: 'web' | 'wenku' | 'local',
  id: string,
  title: string,
): MenuOption => ({
  label: () => h(BookshelfMenuItem, { id, title, type }),
  key: `${type}/${id}`,
});

const menuOptions = computed(() => {
  const localGroup = {
    type: 'group',
    label: '本地小说',
    children: store.local.map(({ id, title }) =>
      menuOption('local', id, title),
    ),
  };
  if (isSignedIn.value) {
    return [
      {
        type: 'group',
        label: '网络小说',
        children: store.web.map(({ id, title }) =>
          menuOption('web', id, title),
        ),
      },
      {
        type: 'divider',
        key: 'divider',
        props: { style: { marginLeft: '32px' } },
      },
      {
        type: 'group',
        label: '文库小说',
        children: store.wenku.map(({ id, title }) =>
          menuOption('wenku', id, title),
        ),
      },
      {
        type: 'divider',
        key: 'divider',
        props: { style: { marginLeft: '32px' } },
      },
      localGroup,
    ];
  } else {
    return [localGroup];
  }
});
</script>

<template>
  <n-scrollbar>
    <n-menu v-bind="$attrs" :options="menuOptions" />
  </n-scrollbar>
</template>
