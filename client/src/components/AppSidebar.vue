<template>
  <!-- Backdrop only renders while the mobile drawer is open; hidden on desktop via CSS -->
  <div
    v-if="mobileOpen"
    class="sidebar-backdrop"
    aria-hidden="true"
    @click="$emit('close')"
  ></div>

  <aside
    id="app-sidebar"
    ref="sidebarEl"
    class="sidebar"
    :class="{ 'icons-only': iconsOnly, 'rail-overlay': railOverlay, open: mobileOpen }"
  >
    <div class="sidebar-brand">
      <div class="brand-mark" aria-hidden="true">{{ brandInitial }}</div>
      <div class="brand-text">
        <span class="brand-name">{{ t('nav.companyName') }}</span>
        <span class="brand-subtitle">{{ t('nav.subtitle') }}</span>
      </div>
    </div>

    <nav class="sidebar-nav" :aria-label="t('nav.mainNavigation')">
      <!-- '/' uses exact matching only; other items use prefix (active-class) matching -->
      <router-link
        v-for="item in navItems"
        :key="item.to"
        :to="item.to"
        class="nav-item"
        :active-class="item.exact ? '' : 'active'"
        exact-active-class="active"
        :title="iconsOnly ? t(item.labelKey) : null"
      >
        <svg
          class="nav-icon"
          width="20"
          height="20"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="1.75"
          stroke-linecap="round"
          stroke-linejoin="round"
          aria-hidden="true"
        >
          <path v-for="d in item.icon" :key="d" :d="d" />
        </svg>
        <span class="nav-label">{{ t(item.labelKey) }}</span>
      </router-link>
    </nav>

    <div class="sidebar-footer">
      <LanguageSwitcher />
      <ProfileMenu
        @show-profile-details="$emit('show-profile-details')"
        @show-tasks="$emit('show-tasks')"
      />
      <button
        ref="toggleEl"
        type="button"
        class="collapse-toggle"
        aria-controls="app-sidebar"
        :aria-label="iconsOnly ? t('nav.expandSidebar') : t('nav.collapseSidebar')"
        :aria-expanded="iconsOnly ? 'false' : 'true'"
        :title="iconsOnly ? t('nav.expandSidebar') : t('nav.collapseSidebar')"
        @click="toggleSidebar"
      >
        <svg
          class="collapse-icon"
          width="20"
          height="20"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="1.75"
          stroke-linecap="round"
          stroke-linejoin="round"
          aria-hidden="true"
        >
          <path d="M11 17l-5-5 5-5" />
          <path d="M18 17l-5-5 5-5" />
        </svg>
        <span class="nav-label">{{ iconsOnly ? t('nav.expandSidebar') : t('nav.collapseSidebar') }}</span>
      </button>
    </div>
  </aside>
</template>

<script>
import { ref, computed, watch, onMounted, onBeforeUnmount } from 'vue'
import { useRoute } from 'vue-router'
import { useI18n } from '../composables/useI18n'
import LanguageSwitcher from './LanguageSwitcher.vue'
import ProfileMenu from './ProfileMenu.vue'

// Persisted desktop preference only. Rail and drawer modes never write this key.
const STORAGE_KEY = 'sidebar-collapsed'

// Viewport breakpoints in px. CSS media queries cannot read custom properties, so the same
// values are written as literals in the <style> blocks below and in App.vue. Keep them in sync.
//   < TABLET            -> off-canvas drawer + hamburger
//   TABLET to DESKTOP-1 -> icons-only rail (always visible); the user can expand it as an overlay
//   >= DESKTOP          -> full sidebar, with a user-controlled collapse that persists
const BREAKPOINTS = {
  TABLET: 768,
  DESKTOP: 1280
}

// Icon path data (24x24 viewBox, stroked). Paths within an icon are unique, so they double as v-for keys.
const NAV_ITEMS = [
  {
    to: '/',
    labelKey: 'nav.overview',
    exact: true,
    icon: ['M3 3h7v9H3z', 'M14 3h7v5h-7z', 'M14 12h7v9h-7z', 'M3 16h7v5H3z']
  },
  {
    to: '/inventory',
    labelKey: 'nav.inventory',
    icon: ['M21 8l-9-5-9 5 9 5 9-5z', 'M3 8v8l9 5 9-5V8', 'M12 13v8']
  },
  {
    to: '/orders',
    labelKey: 'nav.orders',
    icon: [
      'M9 3h6a1 1 0 011 1v2H8V4a1 1 0 011-1z',
      'M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h12a2 2 0 002-2V7a2 2 0 00-2-2h-2',
      'M9 12h6',
      'M9 16h4'
    ]
  },
  {
    to: '/spending',
    labelKey: 'nav.finance',
    icon: ['M12 3v18', 'M17 7H9.5a3 3 0 000 6h5a3 3 0 010 6H6']
  },
  {
    to: '/demand',
    labelKey: 'nav.demandForecast',
    icon: ['M3 17l6-6 4 4 8-8', 'M15 7h6v6']
  },
  {
    to: '/reports',
    labelKey: 'nav.reports',
    icon: ['M4 20h16', 'M7 16v-6', 'M12 16V5', 'M17 16v-4']
  }
]

export default {
  name: 'AppSidebar',
  components: {
    LanguageSwitcher,
    ProfileMenu
  },
  props: {
    // Mobile drawer visibility (< 768px); owned by App.vue so the top bar hamburger can open it
    mobileOpen: {
      type: Boolean,
      default: false
    }
  },
  emits: ['close', 'show-profile-details', 'show-tasks'],
  setup(props, { emit }) {
    const { t } = useI18n()
    const route = useRoute()

    const sidebarEl = ref(null)
    const toggleEl = ref(null)

    // User's desktop collapse preference (>= DESKTOP only)
    const isCollapsed = ref(localStorage.getItem(STORAGE_KEY) === 'true')

    // Viewport mode via matchMedia listeners (no resize polling).
    // Initial values are read synchronously so the first render already has the right layout.
    const mqDesktop = window.matchMedia(`(min-width: ${BREAKPOINTS.DESKTOP}px)`)
    const mqTablet = window.matchMedia(`(min-width: ${BREAKPOINTS.TABLET}px)`)
    const isDesktop = ref(mqDesktop.matches)
    const isTabletUp = ref(mqTablet.matches)

    // Temporary overlay expansion in rail mode. Not persisted; reset on breakpoint change and route change.
    const railExpanded = ref(false)

    // Rail range is tablet-or-wider but narrower than desktop
    const isRail = computed(() => isTabletUp.value && !isDesktop.value)

    // Icons-only applies to the desktop preference and to the rail. The mobile drawer is always full width.
    const iconsOnly = computed(() => {
      if (isDesktop.value) return isCollapsed.value
      if (isRail.value) return !railExpanded.value
      return false
    })

    // Expanded rail floats over the content (the grid column stays at rail width), so it needs a shadow
    const railOverlay = computed(() => isRail.value && railExpanded.value)

    const brandInitial = computed(() => t('nav.companyName').charAt(0))

    const toggleSidebar = () => {
      if (isDesktop.value) {
        isCollapsed.value = !isCollapsed.value
        localStorage.setItem(STORAGE_KEY, String(isCollapsed.value))
      } else if (isRail.value) {
        // Temporary overlay only: the saved desktop preference is intentionally left untouched
        railExpanded.value = !railExpanded.value
      }
    }

    const handleBreakpointChange = () => {
      isDesktop.value = mqDesktop.matches
      isTabletUp.value = mqTablet.matches
      // The overlay only makes sense in the rail range, so drop it on any breakpoint change
      railExpanded.value = false
    }

    // Navigating away closes the rail overlay (same trigger App.vue uses for the mobile drawer)
    watch(() => route.fullPath, () => {
      railExpanded.value = false
    })

    // Outside click closes the rail overlay. Clicks inside the sidebar (including the toggle) are ignored.
    const handlePointerDown = (event) => {
      if (railExpanded.value && sidebarEl.value && !sidebarEl.value.contains(event.target)) {
        railExpanded.value = false
      }
    }

    // Escape closes the mobile drawer or the rail overlay
    const handleKeydown = (event) => {
      if (event.key !== 'Escape') return

      if (props.mobileOpen) {
        emit('close')
      }

      if (railExpanded.value) {
        // Collapsing removes the focused element from view; return focus to the toggle so it is not lost
        const focusWasInside = sidebarEl.value && sidebarEl.value.contains(document.activeElement)
        railExpanded.value = false
        if (focusWasInside && toggleEl.value) {
          toggleEl.value.focus()
        }
      }
    }

    onMounted(() => {
      mqDesktop.addEventListener('change', handleBreakpointChange)
      mqTablet.addEventListener('change', handleBreakpointChange)
      window.addEventListener('keydown', handleKeydown)
      document.addEventListener('pointerdown', handlePointerDown)
    })

    onBeforeUnmount(() => {
      mqDesktop.removeEventListener('change', handleBreakpointChange)
      mqTablet.removeEventListener('change', handleBreakpointChange)
      window.removeEventListener('keydown', handleKeydown)
      document.removeEventListener('pointerdown', handlePointerDown)
    })

    return {
      t,
      navItems: NAV_ITEMS,
      sidebarEl,
      toggleEl,
      iconsOnly,
      railOverlay,
      toggleSidebar,
      brandInitial
    }
  }
}
</script>

<style scoped>
.sidebar {
  position: sticky;
  top: 0;
  height: 100vh;
  width: var(--sidebar-w);
  display: flex;
  flex-direction: column;
  background: var(--sidebar-bg);
  color: var(--sidebar-text);
  z-index: var(--z-sidebar);
  transition: width var(--transition), transform var(--transition);
  /* Footer dropdowns open upward and may extend past the sidebar edge; do not clip them */
  overflow: visible;
}

/* Brand block */
.sidebar-brand {
  position: relative;
  height: var(--topbar-h);
  flex-shrink: 0;
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: 0 var(--space-4);
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
  overflow: hidden;
}

.brand-mark {
  width: 32px;
  height: 32px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: var(--radius-md);
  background: var(--sidebar-active-indicator);
  color: #ffffff;
  font-weight: 700;
  font-size: 0.875rem;
}

.brand-text {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.brand-name {
  font-size: 1rem;
  font-weight: 700;
  color: #ffffff;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.brand-subtitle {
  font-size: 0.75rem;
  color: var(--sidebar-text-muted);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Navigation */
.sidebar-nav {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  padding: var(--space-3);
  overflow-y: auto;
  overflow-x: hidden;
}

.nav-item {
  position: relative;
  display: flex;
  align-items: center;
  gap: var(--space-3);
  height: 40px;
  padding: 0 var(--space-3);
  border-radius: var(--radius-md);
  color: var(--sidebar-text);
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 500;
  white-space: nowrap;
  transition: background var(--transition), color var(--transition);
}

.nav-item:hover {
  background: var(--sidebar-hover);
  color: #ffffff;
}

.nav-item.active {
  background: var(--sidebar-active-bg);
  color: var(--sidebar-active-text);
}

/* 3px active indicator on the left edge of the item */
.nav-item.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 8px;
  bottom: 8px;
  width: 3px;
  border-radius: 0 3px 3px 0;
  background: var(--sidebar-active-indicator);
}

.nav-icon {
  flex-shrink: 0;
}

.nav-label {
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Footer: pinned below the scrollable nav */
.sidebar-footer {
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  padding: var(--space-3);
  border-top: 1px solid rgba(255, 255, 255, 0.06);
  overflow: visible;
}

.collapse-toggle {
  position: relative;
  display: flex;
  align-items: center;
  gap: var(--space-3);
  height: 40px;
  padding: 0 var(--space-3);
  background: none;
  border: none;
  border-radius: var(--radius-md);
  color: var(--sidebar-text-muted);
  font-family: inherit;
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
  white-space: nowrap;
  overflow: hidden;
  transition: background var(--transition), color var(--transition);
}

.collapse-toggle:hover {
  background: var(--sidebar-hover);
  color: #ffffff;
}

.collapse-icon {
  flex-shrink: 0;
  transition: transform var(--transition);
}

/* Focus outlines for every interactive element, including the footer components' buttons */
.nav-item:focus-visible,
.collapse-toggle:focus-visible,
.sidebar-footer :deep(button:focus-visible) {
  outline: 2px solid var(--sidebar-active-indicator);
  outline-offset: 2px;
}

.sidebar-backdrop {
  display: none;
}

/*
 * Icons-only mode (desktop preference collapsed, or the tablet rail).
 * Labels are visually hidden rather than display:none so the links keep an accessible name;
 * the title attribute on each nav item provides the hover tooltip.
 */
.sidebar.icons-only {
  width: var(--sidebar-w-collapsed);
}

.sidebar.icons-only .brand-text,
.sidebar.icons-only .nav-label {
  position: absolute;
  width: 1px;
  height: 1px;
  margin: -1px;
  padding: 0;
  overflow: hidden;
  clip: rect(0 0 0 0);
  white-space: nowrap;
  border: 0;
}

.sidebar.icons-only .sidebar-brand,
.sidebar.icons-only .nav-item,
.sidebar.icons-only .collapse-toggle {
  justify-content: center;
}

.sidebar.icons-only .sidebar-brand {
  padding: 0;
}

.sidebar.icons-only .nav-item,
.sidebar.icons-only .collapse-toggle {
  padding: 0;
}

.sidebar.icons-only .collapse-icon {
  transform: rotate(180deg);
}

/* Footer components: icon/avatar only when collapsed */
.sidebar.icons-only .sidebar-footer :deep(.profile-name),
.sidebar.icons-only .sidebar-footer :deep(.language-label),
.sidebar.icons-only .sidebar-footer :deep(.chevron) {
  display: none;
}

.sidebar.icons-only .sidebar-footer :deep(.profile-button),
.sidebar.icons-only .sidebar-footer :deep(.language-button) {
  justify-content: center;
  padding-left: 0;
  padding-right: 0;
}

/* Tablet rail (768px to 1279px): expanded rail is an overlay, so it casts a shadow over the content */
@media (min-width: 768px) and (max-width: 1279px) {
  .sidebar.rail-overlay {
    box-shadow: var(--shadow-lg);
  }
}

/* Mobile (< 768px): off-canvas drawer */
@media (max-width: 767px) {
  .sidebar {
    position: fixed;
    left: 0;
    top: 0;
    bottom: 0;
    transform: translateX(-100%);
    /* visibility removes the closed drawer from the tab order; delayed so the slide-out animation finishes */
    visibility: hidden;
    transition: transform var(--transition), visibility 0s linear 0.2s;
    box-shadow: var(--shadow-lg);
  }

  .sidebar.open {
    transform: translateX(0);
    visibility: visible;
    transition: transform var(--transition), visibility 0s;
  }

  .collapse-toggle {
    display: none;
  }

  .sidebar-backdrop {
    display: block;
    position: fixed;
    inset: 0;
    background: rgba(15, 23, 42, 0.4);
    z-index: var(--z-backdrop);
  }
}
</style>
