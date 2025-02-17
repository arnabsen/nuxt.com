<script setup lang="ts">
import type { NavItem } from '@nuxt/content/dist/runtime/types'
import type { Link } from '#ui-pro/types'

const logo = ref(null)
const navigation = inject<Ref<NavItem[]>>('navigation')

const stats = useStats()
const { metaSymbol } = useShortcuts()
const { copy } = useCopyToClipboard()

const route = useRoute()
const mobileNav = computed(() => {
  const links = mapContentNavigation(navigation.value)

  // Show Migration and Bridge on mobile only when user is reading them
  const docsLink = links.find((link) => link.to === '/docs')
  if (docsLink && !route.path.startsWith('/docs/bridge') && !route.path.startsWith('/docs/migration')) {
    docsLink.children = docsLink.children.filter((link) => !['/docs/bridge', '/docs/migration'].includes(link.to as string))
  }
  return links
})

const open = ref(false)
const dropdownItems = [
  [{
     label: 'Copy logo as SVG',
     icon: 'i-simple-icons-nuxtdotjs',
     click: () => copy(logo.value.$el.outerHTML, { title: 'Copied to clipboard' })
   },
   {
     label: 'Nuxt Brand Kit',
     icon: 'i-simple-icons-figma',
     to: 'https://www.figma.com/community/file/1296154408275753939/nuxt-brand-kit',
     target: '_blank'
   }],
  [{
    label: 'Browse Design Kit',
    icon: 'i-ph-shapes-duotone',
    to: '/design-kit'
  }]
]

defineProps<{
  links?: Link[]
}>()
</script>

<template>
  <UHeader :links="links">
    <template #left>
      <UDropdown
        v-model:open="open"
        :items="dropdownItems"
        :popper="{ strategy: 'absolute', placement: 'bottom-start' }"
        :ui="{
          container: 'mt-8',
          background: 'bg-white dark:bg-gray-950',
          item: { padding: 'gap-x-2.5 py-2.5', inactive: 'dark:bg-gray-950' },
        }"
      >
        <Logo ref="logo" class="block w-auto h-6" @click.right.prevent="open = true" @click.left.prevent="navigateTo('/')" />
      </UDropdown>
    </template>

    <template #center>
      <UHeaderLinks :links="links" :ui="{ default: { popover: { popper: { strategy: 'absolute' }, ui: { width: 'w-[256px]' } } } }" class="hidden lg:flex" />
    </template>

    <template #right>
      <UTooltip text="Search" :shortcuts="[metaSymbol, 'K']">
        <UContentSearchButton :label="null" />
      </UTooltip>

      <UTooltip :text="$colorMode.preference === 'dark' ? 'Switch to light mode' : 'Switch to dark mode'">
        <UColorModeButton />
      </UTooltip>

      <UTooltip text="GitHub Stars">
        <UButton
          icon="i-simple-icons-github"
          to="https://github.com/nuxt/nuxt"
          target="_blank"
          :label="stats ? formatNumber(stats.stars) : '...'"
          v-bind="($ui.button.secondary as any)"
        />
      </UTooltip>
    </template>

    <template #panel>
      <UNavigationTree :links="mobileNav" default-open :multiple="false" />
    </template>
  </UHeader>
</template>
