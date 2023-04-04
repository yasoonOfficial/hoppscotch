<template>
  <div>
    <header
      class="flex items-center justify-between flex-1 flex-shrink-0 px-2 py-2 space-x-2 overflow-x-auto overflow-y-hidden"
    >
      <div
        class="inline-flex items-center space-x-2"
        :style="{
          paddingTop: platform.ui?.appHeader?.paddingTop?.value,
          paddingLeft: platform.ui?.appHeader?.paddingLeft?.value,
        }"
      >
        <HoppButtonSecondary
          class="tracking-wide !font-bold !text-secondaryDark hover:bg-primaryDark focus-visible:bg-primaryDark uppercase"
          label="yasoon hop"
          to="/"
        />
      </div>
      <div class="inline-flex items-center space-x-2">
        <HoppButtonSecondary
          v-tippy="{ theme: 'tooltip', allowHTML: true }"
          :title="`${t('app.search')} <kbd>/</kbd>`"
          :icon="IconSearch"
          class="rounded hover:bg-primaryDark focus-visible:bg-primaryDark"
          @click="invokeAction('modals.search.toggle')"
        />
        <HoppButtonSecondary
          v-tippy="{ theme: 'tooltip', allowHTML: true }"
          :title="`${
            mdAndLarger ? t('support.title') : t('app.options')
          } <kbd>?</kbd>`"
          :icon="IconLifeBuoy"
          class="rounded hover:bg-primaryDark focus-visible:bg-primaryDark"
          @click="invokeAction('modals.support.toggle')"
        />
        <HoppButtonPrimary
          :label="t('header.login')"
          @click="invokeAction('modals.login.toggle')"
        />
      </div>
    </header>
    <AppAnnouncement v-if="!network.isOnline" />
    <TeamsModal :show="showTeamsModal" @hide-modal="showTeamsModal = false" />
  </div>
</template>

<script setup lang="ts">
import { computed, reactive, ref } from "vue"
import IconUser from "~icons/lucide/user"
import IconSettings from "~icons/lucide/settings"
import IconDownload from "~icons/lucide/download"
import IconSearch from "~icons/lucide/search"
import IconLifeBuoy from "~icons/lucide/life-buoy"
import IconUploadCloud from "~icons/lucide/upload-cloud"
import IconUserPlus from "~icons/lucide/user-plus"
import { breakpointsTailwind, useBreakpoints, useNetwork } from "@vueuse/core"
import { pwaDefferedPrompt, installPWA } from "@modules/pwa"
import { platform } from "~/platform"
import { useI18n } from "@composables/i18n"
import { useReadonlyStream } from "@composables/stream"
import { invokeAction } from "@helpers/actions"

const t = useI18n()

/**
 * Once the PWA code is initialized, this holds a method
 * that can be called to show the user the installation
 * prompt.
 */

const showInstallButton = computed(() => !!pwaDefferedPrompt.value)

const showTeamsModal = ref(false)

const breakpoints = useBreakpoints(breakpointsTailwind)
const mdAndLarger = breakpoints.greater("md")

const network = reactive(useNetwork())

const currentUser = useReadonlyStream(
  platform.auth.getProbableUserStream(),
  platform.auth.getProbableUser()
)

// Template refs
const tippyActions = ref<any | null>(null)
const profile = ref<any | null>(null)
const settings = ref<any | null>(null)
const logout = ref<any | null>(null)
</script>
