<template>
  <!-- 使用ToastUI包装整个布局以提供NMessageProvider -->
  <ToastUI>
    <NLayout style="position: fixed; inset: 0; width: 100vw; height: 100vh;
    max-height: 100vh;
    overflow: hidden; display: flex; min-height: 0;"
    content-style="height: 100%; max-height: 100%; min-height: 0; overflow: hidden;"
    >

      <NFlex vertical style="position: fixed; inset: 0; width: 100vw; max-height: 100vh; height: 100vh; min-height: 0;">
      <!-- 顶部导航栏 -->
      <NLayoutHeader class="theme-header nav-header-enhanced">
        <NFlex justify="space-between" align="center" class="w-full nav-content" :wrap="false" :size="[16, 12]">
          <!-- 左侧：Logo + 标题 + 核心导航 -->
          <NFlex align="center" :size="16" :wrap="false">
            <!-- Logo + 标题 -->
            <NButton
              text
              class="brand-link"
              @click="openBrandWebsite"
            >
              <NFlex align="center" :size="8" :wrap="false">
                <AppPreviewImage
                  :src="logoSrc"
                  alt="Logo"
                  :width="logoSize"
                  :height="logoSize"
                  object-fit="cover"
                  class="logo-image"
                  :show-toolbar="false"
                  :preview-disabled="true"
                  :fallback-src="fallbackLogoSrc"
                />
                <NText class="text-lg sm:text-xl font-bold theme-title" tag="h2">
                  <slot name="title">{{ t('common.appName') }}</slot>
                </NText>
              </NFlex>
            </NButton>

            <!-- 核心导航元素 -->
            <div v-if="!isMobile" class="core-navigation">
              <slot name="core-nav"></slot>
            </div>
          </NFlex>

          <!-- 右侧：操作按钮 -->
          <NFlex v-if="!isMobile" align="center" :size="8" :wrap="true" justify="end" class="nav-actions">
            <slot name="actions"></slot>
          </NFlex>
          <NButton
            v-else
            quaternary
            circle
            size="large"
            class="mobile-menu-button"
            aria-label="Open menu"
            @click="openMobileMenu"
          >
            <template #icon>
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" aria-hidden="true">
                <path d="M4 7h16" />
                <path d="M4 12h16" />
                <path d="M4 17h16" />
              </svg>
            </template>
          </NButton>
        </NFlex>
      </NLayoutHeader>

      <Transition name="mobile-menu">
        <div
          v-if="isMobile && mobileMenuOpen"
          class="mobile-menu-overlay"
          role="dialog"
          aria-modal="true"
          aria-label="Main menu"
        >
          <div class="mobile-menu-drawer">
            <div class="mobile-menu-drawer-header">
              <NButton
                quaternary
                circle
                size="large"
                class="mobile-menu-close"
                aria-label="Close menu"
                @click="closeMobileMenu"
              >
                <template #icon>
                  <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" aria-hidden="true">
                    <path d="M6 6l12 12" />
                    <path d="M18 6L6 18" />
                  </svg>
                </template>
              </NButton>
            </div>
            <div class="mobile-drawer-core-navigation">
              <slot name="core-nav"></slot>
            </div>
            <div class="mobile-drawer-actions">
              <slot name="actions"></slot>
            </div>
          </div>
        </div>
      </Transition>

      <!-- 主要内容区域 - 严格控制在剩余空间内 -->
      <NLayoutContent has-sider
        style="flex: 1; min-height: 0; overflow: hidden;"
        :content-style="layoutContentStyle"
      >
        <div class="main-content-wrapper">
          <slot name="main"></slot>
        </div>
      </NLayoutContent>
      </NFlex>

      <!-- 弹窗插槽 -->
      <slot name="modals"></slot>

    </NLayout>
  </ToastUI>
</template>

<script setup lang="ts">
import { computed, ref, onMounted, onUnmounted } from 'vue'
import type { CSSProperties } from 'vue'

import { useI18n } from 'vue-i18n'
import { NButton, NLayout, NLayoutHeader, NLayoutContent, NFlex, NText } from 'naive-ui'
import ToastUI from './Toast.vue'
import logoImage from '../assets/logo.png'
import AppPreviewImage from './media/AppPreviewImage.vue'
import { openExternalUrl } from '../utils/open-external-url'

const { t } = useI18n()

// Logo图片配置
const logoSrc = logoImage

// 创建简单的SVG fallback logo
const createFallbackSvg = () => {
  const svg = `data:image/svg+xml,${encodeURIComponent(`
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32" fill="none">
      <rect width="32" height="32" rx="6" fill="#3b82f6"/>
      <text x="16" y="21" text-anchor="middle" fill="white" font-family="system-ui" font-size="14" font-weight="bold">P</text>
    </svg>
  `)}`
  return svg
}

const fallbackLogoSrc = createFallbackSvg()

// 响应式Logo尺寸 - 使用更智能的检测
const windowWidth = ref(typeof window !== 'undefined' ? window.innerWidth : 1024)
const isMobile = computed(() => windowWidth.value < 640)
const mobileMenuOpen = ref(false)

const openMobileMenu = () => {
  mobileMenuOpen.value = true
}

const closeMobileMenu = () => {
  mobileMenuOpen.value = false
}

const handleKeydown = (event: KeyboardEvent) => {
  if (event.key === 'Escape') {
    closeMobileMenu()
  }
}

const updateWindowWidth = () => {
  windowWidth.value = window.innerWidth
  if (window.innerWidth >= 640) {
    closeMobileMenu()
  }
}

onMounted(() => {
  if (typeof window !== 'undefined') {
    windowWidth.value = window.innerWidth
    window.addEventListener('resize', updateWindowWidth)
    window.addEventListener('keydown', handleKeydown)
  }
})

onUnmounted(() => {
  if (typeof window !== 'undefined') {
    window.removeEventListener('resize', updateWindowWidth)
    window.removeEventListener('keydown', handleKeydown)
  }
})

const logoSize = computed(() => {
  if (windowWidth.value < 480) {
    return 20 // 超小屏幕
  } else if (windowWidth.value < 640) {
    return 24 // 小屏幕
  }
  return 28 // 默认尺寸
})

const layoutContentStyle = computed<CSSProperties>(() => ({
  height: '100%',
  maxHeight: '100%',
  minHeight: 0,
  boxSizing: 'border-box',
  padding: isMobile.value ? '5px' : '24px clamp(16px, 2vw, 48px) 40px',
  display: 'flex',
  flexDirection: 'column',
  alignItems: 'stretch',
  overflow: 'hidden',
}))

const openBrandWebsite = async () => {
  await openExternalUrl('https://always200.com', { logPrefix: 'MainLayout' })
}
</script>

<style>
.main-content-wrapper {
  width: 100%;
  margin: 0;
  height: 100%;
  display: flex;
  flex-direction: column;
  flex: 1;
  min-height: 0;
  overflow: auto;
}

.main-content-wrapper > * {
  flex: 1;
  min-height: 0;
}

/* 增强导航栏样式 */
.nav-header-enhanced {
  min-height: 64px !important;
  padding: 12px 16px !important;
}

.nav-content {
  min-height: 40px;
}

.nav-actions {
  min-height: 40px;
}

.mobile-menu-button,
.mobile-menu-close {
  width: 40px;
  height: 40px;
  flex-shrink: 0;
}

.mobile-menu-button svg,
.mobile-menu-close svg {
  width: 22px;
  height: 22px;
}

.mobile-menu-overlay {
  position: fixed;
  inset: 0;
  z-index: 3000;
  width: 100vw;
  height: 100vh;
  background: var(--n-color);
}

.mobile-menu-drawer {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
  padding: max(12px, env(safe-area-inset-top)) 20px max(24px, env(safe-area-inset-bottom));
  overflow: auto;
  background: var(--n-color);
  color: var(--n-text-color);
}

.mobile-menu-drawer-header {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  min-height: 44px;
  margin-bottom: 18px;
}

.mobile-drawer-core-navigation {
  padding-bottom: 20px;
  margin-bottom: 20px;
  border-bottom: 1px solid var(--n-border-color);
}

.mobile-drawer-core-navigation [data-testid="core-nav"] {
  display: flex;
  align-items: stretch;
  flex-direction: column;
  gap: 12px !important;
}

.mobile-drawer-actions {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  gap: 18px;
}

.mobile-drawer-actions .page-destination-group,
.mobile-drawer-actions .modal-action-group,
.mobile-drawer-actions .aux-icon-group {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
  margin-left: 0;
}

.mobile-drawer-actions .page-destination-group {
  padding-right: 0;
  padding-bottom: 18px;
  border-right: 0;
  border-bottom: 1px solid var(--n-border-color);
}

.mobile-menu-enter-active,
.mobile-menu-leave-active {
  transition: transform 0.26s ease, opacity 0.26s ease;
}

.mobile-menu-enter-from,
.mobile-menu-leave-to {
  transform: translateX(-100%);
  opacity: 0;
}

.mobile-menu-enter-to,
.mobile-menu-leave-from {
  transform: translateX(0);
  opacity: 1;
}

.brand-link {
  align-items: center;
  padding: 6px 10px 6px 6px;
  border-radius: 12px;
  color: inherit;
  transition:
    background-color 0.2s ease-in-out,
    box-shadow 0.2s ease-in-out,
    transform 0.2s ease-in-out;
}

.brand-link:hover {
  background: color-mix(in srgb, var(--n-primary-color) 10%, transparent);
  transform: translateY(-1px);
}

.brand-link:hover .logo-image {
  transform: scale(1.05);
}

.brand-link:hover .theme-title {
  opacity: 0.88;
}

.brand-link:focus-visible {
  outline: none;
  background: color-mix(in srgb, var(--n-primary-color) 14%, transparent);
  box-shadow: 0 0 0 2px color-mix(in srgb, var(--n-primary-color) 28%, transparent);
}

/* Logo样式优化 */
.logo-image {
  border-radius: 6px;
  transition: transform 0.2s ease-in-out;
  flex-shrink: 0;
}

/* 标题文字对齐优化 */
.theme-title {
  line-height: 1.2 !important;
  margin: 0 !important;
  white-space: nowrap;
  transition: opacity 0.2s ease-in-out;
}

/* 核心导航样式 */
.core-navigation {
  display: flex;
  align-items: center;
  margin-left: 16px;
  padding-left: 16px;
  border-left: 1px solid var(--n-border-color);
  min-height: 32px;
}

/* 响应式优化 */
@media (max-width: 639px) {
  .logo-image {
    border-radius: 4px;
  }

  .core-navigation {
    margin-left: 8px;
    padding-left: 8px;
  }

  .nav-header-enhanced {
    min-height: 56px !important;
    padding: 8px 12px !important;
  }
}

/* 内容区域移动端适配 */
@media (max-width: 639px) {
  .main-content-wrapper {
    padding: 0;
  }
}

.custom-select {
  -webkit-appearance: none !important;
  -moz-appearance: none !important;
  appearance: none !important;
  background-image: none !important;
}

.custom-select::-ms-expand {
  display: none;
}
</style>
