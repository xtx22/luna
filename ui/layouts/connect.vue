<script lang="ts" setup>
import KokoSftpTransferCenter from "#koko/components/FileManagement/SftpTransferCenter.vue";
import {
  SettingsAboutPage,
  SettingsAppearancePage,
  SettingsApplicationPage,
  SettingsGeneralPage,
  SettingsUserPage
} from "~/composables/loadSettingsSection";
import { shouldShowSessionTabStrip } from "~/composables/useSessionWindowConnect";

const route = useRoute();
const { isMacOS } = usePlatform();
const { focusMode, nativeFullscreen } = useWorkspaceTabs();
useWorkspaceFullscreenShortcuts();
const isSessionWindow = computed(() => route.path.startsWith("/session/"));
const showSessionTabs = computed(() => shouldShowSessionTabStrip(route.path, route.query));
const showSessionTabChrome = computed(() => showSessionTabs.value && !focusMode.value);
const macTrafficLightInset = computed(() => isDesktopRuntime() && isMacOS.value && !nativeFullscreen.value);
const { open: settingsOpen, activeSection: activeSettingsSection } = useSettingsWindow();
const settingsSectionPages = {
  user: SettingsUserPage,
  general: SettingsGeneralPage,
  appearance: SettingsAppearancePage,
  application: SettingsApplicationPage,
  about: SettingsAboutPage
} as const;
const activeSettingsPage = computed(() => settingsSectionPages[activeSettingsSection.value] || SettingsAboutPage);
</script>

<template>
  <div class="flex h-dvh w-full flex-col overflow-hidden" :style="{ backgroundColor: 'var(--app-main-bg)' }">
    <HeaderDesktopTitleBar v-if="isSessionWindow && !isMacOS" :show-menus="false">
      <WorkspaceTabHeader v-if="showSessionTabChrome" standalone />
      <template v-if="showSessionTabChrome" #trailing>
        <HeaderActionButtons :show-profile="true" />
      </template>
    </HeaderDesktopTitleBar>
    <HeaderDesktopTitleBar v-else :show-menus="false" />
    <header
      v-if="isDesktopRuntime() && isMacOS && !isSessionWindow"
      data-desktop-drag-region
      class="flex h-8 min-h-8 items-center px-20 text-xs font-medium"
    >
      <span class="truncate">JumpServer</span>
    </header>
    <WorkspaceTopHeader v-if="showSessionTabChrome && (!isDesktopRuntime() || isMacOS)" :show-profile="true">
      <div class="h-full" :class="macTrafficLightInset ? 'pl-20' : ''">
        <WorkspaceTabHeader standalone />
      </div>
    </WorkspaceTopHeader>
    <div class="min-h-0 min-w-0 flex-1 overflow-hidden">
      <slot />
    </div>
    <KokoSftpTransferCenter />

    <Transition name="settings-overlay">
      <div
        v-if="settingsOpen"
        role="dialog"
        aria-modal="true"
        :aria-label="$t('Common.Settings')"
        data-state="open"
        class="fixed inset-0 z-200"
      >
        <SettingsShell mode="inline" :active-section="activeSettingsSection" class="h-full">
          <Transition name="settings-section" mode="out-in">
            <KeepAlive>
              <component
                :is="activeSettingsPage"
                :key="activeSettingsSection"
                v-bind="activeSettingsSection === 'application' ? { embedded: true } : {}"
              />
            </KeepAlive>
          </Transition>
        </SettingsShell>
      </div>
    </Transition>
  </div>
</template>
