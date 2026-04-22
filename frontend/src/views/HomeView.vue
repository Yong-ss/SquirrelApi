<template>
  <!-- Custom Home Content: Full Page Mode -->
  <div v-if="homeContent" class="min-h-screen">
    <!-- iframe mode -->
    <iframe
      v-if="isHomeContentUrl"
      :src="homeContent.trim()"
      class="h-screen w-full border-0"
      allowfullscreen
    ></iframe>
    <!-- HTML mode - SECURITY: homeContent is admin-only setting, XSS risk is acceptable -->
    <div v-else v-html="homeContent"></div>
  </div>

  <!-- Default Home Page -->
  <div
    v-else
    class="relative flex min-h-screen flex-col overflow-hidden bg-gradient-to-br from-gray-50 via-primary-50/30 to-gray-100 dark:from-dark-950 dark:via-dark-900 dark:to-dark-950"
  >
    <!-- Background Decorations -->
    <div class="pointer-events-none absolute inset-0 overflow-hidden">
      <div
        class="absolute -right-40 -top-40 h-96 w-96 rounded-full bg-primary-400/20 blur-3xl"
      ></div>
      <div
        class="absolute -bottom-40 -left-40 h-96 w-96 rounded-full bg-primary-500/15 blur-3xl"
      ></div>
      <div
        class="absolute left-1/3 top-1/4 h-72 w-72 rounded-full bg-primary-300/10 blur-3xl"
      ></div>
      <div
        class="absolute bottom-1/4 right-1/4 h-64 w-64 rounded-full bg-primary-400/10 blur-3xl"
      ></div>
      <div
        class="absolute inset-0 bg-[linear-gradient(rgba(20,184,166,0.03)_1px,transparent_1px),linear-gradient(90deg,rgba(20,184,166,0.03)_1px,transparent_1px)] bg-[size:64px_64px]"
      ></div>
    </div>

    <!-- Header: Floating Glass Navbar -->
    <header class="fixed top-0 left-0 right-0 z-50 flex justify-center px-4 py-4 pointer-events-none">
      <nav class="pointer-events-auto flex w-full max-w-5xl items-center justify-between rounded-2xl border border-white/10 bg-white/60 px-4 py-2 shadow-glass backdrop-blur-glass transition-all duration-500 dark:border-white/5 dark:bg-dark-950/60 md:px-6">
        <!-- Logo Area -->
        <div class="flex items-center gap-3">
          <div class="group relative h-9 w-9 overflow-hidden rounded-xl shadow-inner-glow transition-transform hover:scale-105 active:scale-95">
            <div class="absolute inset-0 bg-primary-500/10 blur-sm group-hover:bg-primary-500/20"></div>
            <img :src="siteLogo || '/logo.png'" alt="Logo" class="relative z-10 h-full w-full object-contain p-0.5" />
          </div>
          <span class="hidden font-display text-lg font-bold tracking-tight text-gray-900 dark:text-white sm:block">
            Sub2API
          </span>
        </div>

        <!-- Nav Actions -->
        <div class="flex items-center gap-1.5 md:gap-3">
          <!-- Functional Group -->
          <div class="flex items-center gap-1">
            <LocaleSwitcher />

            <a
              v-if="docUrl"
              :href="docUrl"
              target="_blank"
              rel="noopener noreferrer"
              class="flex h-9 w-9 items-center justify-center rounded-xl text-gray-500 transition-all hover:bg-gray-100 hover:text-gray-900 dark:text-dark-400 dark:hover:bg-dark-800 dark:hover:text-white"
              :title="t('home.viewDocs')"
            >
              <Icon name="book" size="md" />
            </a>

            <button
              @click="toggleTheme"
              class="flex h-9 w-9 items-center justify-center rounded-xl text-gray-500 transition-all hover:bg-gray-100 hover:text-gray-900 dark:text-dark-400 dark:hover:bg-dark-800 dark:hover:text-white"
              :title="isDark ? t('home.switchToLight') : t('home.switchToDark')"
            >
              <Icon v-if="isDark" name="sun" size="md" />
              <Icon v-else name="moon" size="md" />
            </button>
          </div>

          <div class="mx-1 h-5 w-px bg-gray-200 dark:bg-dark-800"></div>

          <!-- Auth Group -->
          <div class="flex items-center gap-2">
            <template v-if="!isAuthenticated">
              <router-link
                to="/login"
                class="rounded-xl px-4 py-2 text-sm font-semibold text-gray-700 transition-all hover:bg-gray-100 dark:text-gray-300 dark:hover:bg-dark-800"
              >
                {{ t('home.login') }}
              </router-link>
              <router-link
                to="/login?tab=register"
                class="group relative overflow-hidden rounded-xl bg-primary-500 px-5 py-2 text-sm font-bold text-white shadow-glow transition-all hover:scale-105 hover:bg-primary-600 active:scale-95"
              >
                <span class="relative z-10">{{ t('home.getStarted') }}</span>
                <div class="absolute inset-0 -translate-x-full bg-gradient-to-r from-transparent via-white/25 to-transparent transition-transform duration-700 group-hover:translate-x-full"></div>
              </router-link>
            </template>
            <template v-else>
              <router-link
                :to="dashboardPath"
                class="group relative flex items-center gap-2 overflow-hidden rounded-xl bg-gray-950 px-4 py-1.5 text-sm font-bold text-white shadow-xl transition-all hover:scale-105 active:scale-95 dark:bg-primary-600"
              >
                <span class="flex h-6 w-6 items-center justify-center rounded-lg bg-gradient-to-br from-primary-400 to-primary-600 text-[10px] font-bold text-white">
                  {{ userInitial }}
                </span>
                <span class="relative z-10">{{ t('home.dashboard') }}</span>
                <i class="ri-arrow-right-line relative z-10 transition-transform group-hover:translate-x-1"></i>
                <div class="absolute inset-0 -translate-x-full bg-gradient-to-r from-transparent via-white/20 to-transparent transition-transform duration-700 group-hover:translate-x-full"></div>
              </router-link>
            </template>
          </div>
        </div>
      </nav>
    </header>

    <!-- Main Content -->
    <main class="relative z-10 flex-1 px-6 pb-24 pt-32 lg:pt-40">
      <div class="mx-auto max-w-6xl">
        <!-- Hero Section - Left/Right Layout -->
        <div class="relative mb-24 flex flex-col items-center justify-between gap-16 lg:flex-row">
          <!-- Background Glow Halo -->
          <div class="absolute -top-24 left-1/4 -z-10 h-[500px] w-[500px] rounded-full bg-primary-500/10 blur-[120px] dark:bg-primary-500/5"></div>

          <!-- Left: Text Content -->
          <div class="flex-1 text-center lg:text-left">
            <h1 class="mb-6 font-display text-5xl font-extrabold tracking-tight text-gray-900 dark:text-white md:text-6xl xl:text-7xl">
              <span class="block">{{ siteName }}</span>
              <span class="mt-2 block bg-gradient-to-r from-primary-600 to-teal-400 bg-clip-text text-transparent">
                {{ siteSubtitle }}
              </span>
            </h1>
            <p class="mx-auto mb-10 max-w-xl text-lg leading-relaxed text-gray-600 dark:text-gray-400 lg:mx-0 md:text-xl">
              {{ t('home.heroDescription') }}
            </p>

            <!-- CTA Buttons -->
            <div class="flex flex-col items-center gap-4 sm:flex-row lg:justify-start">
              <router-link
                :to="isAuthenticated ? dashboardPath : '/login'"
                class="group relative flex items-center gap-2 overflow-hidden rounded-2xl bg-gray-950 px-8 py-4 text-lg font-bold text-white shadow-2xl transition-all hover:scale-105 active:scale-95 dark:bg-primary-500"
              >
                <span class="relative z-10">{{ isAuthenticated ? t('home.goToDashboard') : t('home.getStarted') }}</span>
                <Icon name="arrowRight" size="md" class="relative z-10 transition-transform group-hover:translate-x-1" />
                <div class="absolute inset-0 -translate-x-full bg-gradient-to-r from-transparent via-white/20 to-transparent transition-transform duration-700 group-hover:translate-x-full"></div>
              </router-link>

              <a
                v-if="docUrl"
                :href="docUrl"
                target="_blank"
                class="flex items-center gap-2 rounded-2xl border border-gray-200 bg-white px-8 py-4 text-lg font-semibold text-gray-700 transition-all hover:bg-gray-50 dark:border-dark-700 dark:bg-dark-900 dark:text-gray-300 dark:hover:bg-dark-800"
              >
                {{ t('home.viewDocs') }}
              </a>
            </div>
          </div>

          <!-- Right: Premium Terminal -->
          <div class="flex flex-1 justify-center perspective-1000 lg:justify-end">
            <div class="terminal-window group relative w-full max-w-lg overflow-hidden rounded-2xl border border-white/10 bg-dark-950/80 shadow-glass-dark backdrop-blur-glass transition-all duration-500 hover:rotate-y-0 hover:scale-[1.02] md:rotate-y-[-5deg]">
              <!-- Terminal Header -->
              <div class="flex items-center justify-between border-b border-white/5 bg-white/5 px-4 py-3">
                <div class="flex gap-2">
                  <div class="h-3 w-3 rounded-full bg-[#FF5F56] shadow-[0_0_8px_#FF5F56/40]"></div>
                  <div class="h-3 w-3 rounded-full bg-[#FFBD2E] shadow-[0_0_8px_#FFBD2E/40]"></div>
                  <div class="h-3 w-3 rounded-full bg-[#27C93F] shadow-[0_0_8px_#27C93F/40]"></div>
                </div>
                <div class="font-mono text-[10px] font-medium tracking-widest text-gray-500 uppercase">bash — api-gateway</div>
                <div class="w-12"></div>
              </div>
              <!-- Terminal Content -->
              <div class="p-6 font-mono text-sm leading-relaxed sm:text-base">
                <div class="flex gap-3 opacity-0 animate-slide-up [animation-delay:400ms] [animation-fill-mode:forwards]">
                  <span class="text-primary-500 font-bold">➜</span>
                  <span class="text-gray-300">curl -X POST "/v1/messages" \</span>
                </div>
                <div class="flex gap-3 pl-7 opacity-0 animate-slide-up [animation-delay:600ms] [animation-fill-mode:forwards]">
                  <span class="text-gray-400">-H "Authorization: Bearer sk-***"</span>
                </div>
                <div class="my-3 flex items-center gap-3 opacity-0 animate-slide-up [animation-delay:1200ms] [animation-fill-mode:forwards]">
                  <span class="rounded bg-primary-500/20 px-2 py-0.5 text-xs font-bold text-primary-400 uppercase">Routing</span>
                  <span class="text-gray-500 italic">Dispatched to Claude-3...</span>
                </div>
                <div class="flex gap-3 opacity-0 animate-slide-up [animation-delay:2000ms] [animation-fill-mode:forwards]">
                  <span class="text-primary-400 font-bold">✓</span>
                  <span class="text-primary-200">HTTP 200 OK</span>
                </div>
                <div class="mt-1 flex gap-3 pl-7 opacity-0 animate-slide-up [animation-delay:2200ms] [animation-fill-mode:forwards]">
                  <span class="text-teal-400">{ "status": "success", "latency": "42ms" }</span>
                </div>
                <div class="mt-4 flex items-center gap-2">
                  <span class="text-primary-500 font-bold">➜</span>
                  <span class="h-4 w-2 bg-primary-500 animate-pulse"></span>
                </div>
              </div>
              <!-- Subtle Shimmer -->
              <div class="absolute inset-0 pointer-events-none bg-gradient-to-tr from-white/5 via-transparent to-transparent opacity-30"></div>
            </div>
          </div>
        </div>

        <!-- Floating Feature Tags -->
        <div class="mb-24 flex flex-wrap items-center justify-center gap-4 md:gap-8">
          <div class="animate-float [animation-delay:0ms] flex items-center gap-3 rounded-2xl border border-gray-200/50 bg-white/40 px-6 py-3 shadow-sm backdrop-blur-md transition-all hover:bg-white/60 dark:border-white/5 dark:bg-dark-800/40">
            <div class="flex h-8 w-8 items-center justify-center rounded-lg bg-primary-500/10 text-primary-500">
              <Icon name="swap" size="sm" />
            </div>
            <span class="text-sm font-bold tracking-tight text-gray-700 dark:text-gray-200 md:text-base">
              {{ t('home.tags.subscriptionToApi') }}
            </span>
          </div>
          <div class="animate-float [animation-delay:1000ms] flex items-center gap-3 rounded-2xl border border-gray-200/50 bg-white/40 px-6 py-3 shadow-sm backdrop-blur-md transition-all hover:bg-white/60 dark:border-white/5 dark:bg-dark-800/40">
            <div class="flex h-8 w-8 items-center justify-center rounded-lg bg-teal-500/10 text-teal-500">
              <Icon name="shield" size="sm" />
            </div>
            <span class="text-sm font-bold tracking-tight text-gray-700 dark:text-gray-200 md:text-base">
              {{ t('home.tags.stickySession') }}
            </span>
          </div>
          <div class="animate-float [animation-delay:2000ms] flex items-center gap-3 rounded-2xl border border-gray-200/50 bg-white/40 px-6 py-3 shadow-sm backdrop-blur-md transition-all hover:bg-white/60 dark:border-white/5 dark:bg-dark-800/40">
            <div class="flex h-8 w-8 items-center justify-center rounded-lg bg-indigo-500/10 text-indigo-500">
              <Icon name="chart" size="sm" />
            </div>
            <span class="text-sm font-bold tracking-tight text-gray-700 dark:text-gray-200 md:text-base">
              {{ t('home.tags.realtimeBilling') }}
            </span>
          </div>
        </div>

        <!-- Bento Features Grid -->
        <div class="mb-32 grid grid-cols-1 gap-6 md:grid-cols-12 md:grid-rows-2">
          <!-- Feature 1: Large Bento -->
          <div class="group relative overflow-hidden rounded-3xl border border-gray-200/50 bg-white/50 p-8 shadow-bento backdrop-blur-md transition-all duration-500 hover:shadow-card-hover dark:border-white/5 dark:bg-dark-900/50 md:col-span-8 md:row-span-1">
            <div class="relative z-10 flex h-full flex-col justify-between">
              <div>
                <div class="mb-6 flex h-14 w-14 items-center justify-center rounded-2xl bg-gradient-to-br from-blue-500 to-indigo-600 shadow-lg shadow-blue-500/20 transition-transform group-hover:scale-110">
                  <Icon name="server" size="lg" class="text-white" />
                </div>
                <h3 class="mb-3 font-display text-2xl font-bold text-gray-900 dark:text-white">
                  {{ t('home.features.unifiedGateway') }}
                </h3>
                <p class="max-w-md text-lg leading-relaxed text-gray-600 dark:text-gray-400">
                  {{ t('home.features.unifiedGatewayDesc') }}
                </p>
              </div>
            </div>
            <!-- Decorative Pattern -->
            <div class="absolute -right-12 -top-12 h-64 w-64 rounded-full bg-blue-500/5 blur-3xl transition-all group-hover:bg-blue-500/10"></div>
          </div>

          <!-- Feature 2: Small Bento -->
          <div class="group relative overflow-hidden rounded-3xl border border-gray-200/50 bg-white/50 p-8 shadow-bento backdrop-blur-md transition-all duration-500 hover:shadow-card-hover dark:border-white/5 dark:bg-dark-900/50 md:col-span-4 md:row-span-1">
            <div class="mb-6 flex h-14 w-14 items-center justify-center rounded-2xl bg-gradient-to-br from-primary-500 to-teal-500 shadow-lg shadow-primary-500/20 transition-transform group-hover:scale-110">
              <Icon name="users" size="lg" class="text-white" />
            </div>
            <h3 class="mb-3 font-display text-xl font-bold text-gray-900 dark:text-white">
              {{ t('home.features.multiAccount') }}
            </h3>
            <p class="text-gray-600 dark:text-gray-400">
              {{ t('home.features.multiAccountDesc') }}
            </p>
          </div>

          <!-- Feature 3: Small Bento (Bottom Left) -->
          <div class="group relative overflow-hidden rounded-3xl border border-gray-200/50 bg-white/50 p-8 shadow-bento backdrop-blur-md transition-all duration-500 hover:shadow-card-hover dark:border-white/5 dark:bg-dark-900/50 md:col-span-4 md:row-span-1">
            <div class="mb-6 flex h-14 w-14 items-center justify-center rounded-2xl bg-gradient-to-br from-purple-500 to-pink-600 shadow-lg shadow-purple-500/20 transition-transform group-hover:scale-110">
              <Icon name="chart" size="lg" class="text-white" />
            </div>
            <h3 class="mb-3 font-display text-xl font-bold text-gray-900 dark:text-white">
              {{ t('home.features.balanceQuota') }}
            </h3>
            <p class="text-gray-600 dark:text-gray-400">
              {{ t('home.features.balanceQuotaDesc') }}
            </p>
          </div>

          <!-- Feature 4: Interactive Bento (Bottom Right) -->
          <div class="group relative flex flex-col justify-center overflow-hidden rounded-3xl bg-gray-900 p-8 shadow-bento transition-all duration-500 hover:shadow-card-hover dark:bg-primary-600/20 md:col-span-8 md:row-span-1">
            <div class="relative z-10">
              <h3 class="mb-4 font-display text-3xl font-bold text-white">
                {{ t('home.heroSubtitle') }}
              </h3>
              <router-link
                :to="isAuthenticated ? dashboardPath : '/login'"
                class="inline-flex items-center gap-2 font-bold text-primary-400 transition-all hover:gap-3"
              >
                {{ t('home.getStarted') }}
                <Icon name="arrowRight" size="sm" />
              </router-link>
            </div>
            <div class="absolute right-0 top-0 h-full w-1/2 bg-gradient-to-l from-primary-500/10 to-transparent"></div>
          </div>
        </div>

        <!-- Supported Providers: Minimalist Grid -->
        <div class="mb-12 text-center">
          <h2 class="mb-3 font-display text-2xl font-bold text-gray-900 dark:text-white">
            {{ t('home.providers.title') }}
          </h2>
          <p class="text-sm text-gray-600 dark:text-gray-400">
            {{ t('home.providers.description') }}
          </p>
        </div>

        <div class="mb-24 flex flex-wrap items-center justify-center gap-4">
          <!-- Claude -->
          <div class="group flex items-center gap-3 rounded-2xl border border-gray-200/50 bg-white/40 px-6 py-3 shadow-sm backdrop-blur-md transition-all hover:scale-105 hover:border-orange-500/30 hover:bg-white/60 dark:border-white/5 dark:bg-dark-800/40">
            <div class="flex h-8 w-8 items-center justify-center rounded-lg bg-gradient-to-br from-orange-400 to-orange-600 text-xs font-bold text-white shadow-lg shadow-orange-500/20">C</div>
            <span class="font-bold tracking-tight text-gray-700 dark:text-gray-200">{{ t('home.providers.claude') }}</span>
            <span class="rounded-full bg-orange-500/10 px-2 py-0.5 text-[10px] font-bold text-orange-600 dark:text-orange-400">Live</span>
          </div>
          <!-- GPT -->
          <div class="group flex items-center gap-3 rounded-2xl border border-gray-200/50 bg-white/40 px-6 py-3 shadow-sm backdrop-blur-md transition-all hover:scale-105 hover:border-emerald-500/30 hover:bg-white/60 dark:border-white/5 dark:bg-dark-800/40">
            <div class="flex h-8 w-8 items-center justify-center rounded-lg bg-gradient-to-br from-emerald-500 to-teal-600 text-xs font-bold text-white shadow-lg shadow-emerald-500/20">G</div>
            <span class="font-bold tracking-tight text-gray-700 dark:text-gray-200">GPT-4</span>
            <span class="rounded-full bg-emerald-500/10 px-2 py-0.5 text-[10px] font-bold text-emerald-600 dark:text-emerald-400">Live</span>
          </div>
          <!-- Gemini -->
          <div class="group flex items-center gap-3 rounded-2xl border border-gray-200/50 bg-white/40 px-6 py-3 shadow-sm backdrop-blur-md transition-all hover:scale-105 hover:border-blue-500/30 hover:bg-white/60 dark:border-white/5 dark:bg-dark-800/40">
            <div class="flex h-8 w-8 items-center justify-center rounded-lg bg-gradient-to-br from-blue-500 to-indigo-600 text-xs font-bold text-white shadow-lg shadow-blue-500/20">G</div>
            <span class="font-bold tracking-tight text-gray-700 dark:text-gray-200">{{ t('home.providers.gemini') }}</span>
            <span class="rounded-full bg-blue-500/10 px-2 py-0.5 text-[10px] font-bold text-blue-600 dark:text-blue-400">Live</span>
          </div>
          <!-- Antigravity -->
          <div class="group flex items-center gap-3 rounded-2xl border border-gray-200/50 bg-white/40 px-6 py-3 shadow-sm backdrop-blur-md transition-all hover:scale-105 hover:border-rose-500/30 hover:bg-white/60 dark:border-white/5 dark:bg-dark-800/40">
            <div class="flex h-8 w-8 items-center justify-center rounded-lg bg-gradient-to-br from-rose-500 to-pink-600 text-xs font-bold text-white shadow-lg shadow-rose-500/20">A</div>
            <span class="font-bold tracking-tight text-gray-700 dark:text-gray-200">{{ t('home.providers.antigravity') }}</span>
            <span class="rounded-full bg-rose-500/10 px-2 py-0.5 text-[10px] font-bold text-rose-600 dark:text-rose-400">Live</span>
          </div>
          <!-- More -->
          <div class="group flex items-center gap-3 rounded-2xl border border-dashed border-gray-300 bg-transparent px-6 py-3 transition-all hover:border-primary-500/50 dark:border-white/10">
            <div class="flex h-8 w-8 items-center justify-center rounded-lg bg-gray-100 text-gray-400 dark:bg-dark-800">+</div>
            <span class="font-bold tracking-tight text-gray-400">{{ t('home.providers.more') }}</span>
          </div>
        </div>
      </div>
    </main>

    <!-- Footer: Minimalist Enterprise -->
    <footer class="relative z-10 border-t border-gray-200/50 px-6 py-12 dark:border-white/5">
      <div class="mx-auto max-w-6xl">
        <div class="flex flex-col items-center justify-between gap-6 md:flex-row">
          <div class="flex items-center gap-3">
            <div class="h-8 w-8 overflow-hidden rounded-lg opacity-80">
              <img :src="siteLogo || '/logo.png'" alt="Logo" class="h-full w-full object-contain" />
            </div>
            <span class="font-display text-lg font-bold tracking-tight text-gray-900 dark:text-white">
              {{ siteName }}
            </span>
          </div>
          
          <div class="flex items-center gap-8">
            <a v-if="docUrl" :href="docUrl" target="_blank" class="text-sm font-semibold text-gray-500 transition-colors hover:text-primary-500 dark:text-gray-400 dark:hover:text-primary-400">
              {{ t('home.docs') }}
            </a>
            <a :href="githubUrl" target="_blank" class="text-sm font-semibold text-gray-500 transition-colors hover:text-primary-500 dark:text-gray-400 dark:hover:text-primary-400">
              GitHub
            </a>
          </div>

          <p class="text-sm font-medium text-gray-400 dark:text-dark-500">
            &copy; {{ currentYear }} {{ siteName }}. {{ t('home.footer.allRightsReserved') }}
          </p>
        </div>
      </div>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { useAuthStore, useAppStore } from '@/stores'
import LocaleSwitcher from '@/components/common/LocaleSwitcher.vue'
import Icon from '@/components/icons/Icon.vue'

const { t } = useI18n()

const authStore = useAuthStore()
const appStore = useAppStore()

// Site settings - directly from appStore (already initialized from injected config)
const siteName = computed(() => appStore.cachedPublicSettings?.site_name || appStore.siteName || 'Sub2API')
const siteLogo = computed(() => appStore.cachedPublicSettings?.site_logo || appStore.siteLogo || '')
const siteSubtitle = computed(() => appStore.cachedPublicSettings?.site_subtitle || 'AI API Gateway Platform')
const docUrl = computed(() => appStore.cachedPublicSettings?.doc_url || appStore.docUrl || '')
const homeContent = computed(() => appStore.cachedPublicSettings?.home_content || '')

// Check if homeContent is a URL (for iframe display)
const isHomeContentUrl = computed(() => {
  const content = homeContent.value.trim()
  return content.startsWith('http://') || content.startsWith('https://')
})

// Theme
const isDark = ref(document.documentElement.classList.contains('dark'))

// GitHub URL
const githubUrl = 'https://github.com/Wei-Shaw/sub2api'

// Auth state
const isAuthenticated = computed(() => authStore.isAuthenticated)
const isAdmin = computed(() => authStore.isAdmin)
const dashboardPath = computed(() => isAdmin.value ? '/admin/dashboard' : '/dashboard')
const userInitial = computed(() => {
  const user = authStore.user
  if (!user || !user.email) return ''
  return user.email.charAt(0).toUpperCase()
})

// Current year for footer
const currentYear = computed(() => new Date().getFullYear())

// Toggle theme
function toggleTheme() {
  isDark.value = !isDark.value
  document.documentElement.classList.toggle('dark', isDark.value)
  localStorage.setItem('theme', isDark.value ? 'dark' : 'light')
}

// Initialize theme
function initTheme() {
  const savedTheme = localStorage.getItem('theme')
  if (
    savedTheme === 'dark' ||
    (!savedTheme && window.matchMedia('(prefers-color-scheme: dark)').matches)
  ) {
    isDark.value = true
    document.documentElement.classList.add('dark')
  }
}

onMounted(() => {
  initTheme()

  // Check auth state
  authStore.checkAuth()

  // Ensure public settings are loaded (will use cache if already loaded from injected config)
  if (!appStore.publicSettingsLoaded) {
    appStore.fetchPublicSettings()
  }
})
</script>

<style scoped>
/* Advanced Layout Animations */
.perspective-1000 {
  perspective: 1000px;
}

.rotate-y-\[-5deg\] {
  transform: rotateY(-5deg);
}

.hover\:rotate-y-0:hover {
  transform: rotateY(0deg);
}

@keyframes line-appear {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-slide-up {
  animation: line-appear 0.6s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
</style>
