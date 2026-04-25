<template>
  <!-- Custom Home Content: Full Page Mode -->
  <div v-if="homeContent" class="min-h-screen">
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
    class="noise-bg relative min-h-screen overflow-x-hidden bg-gradient-to-br from-gray-50 via-primary-50/20 to-gray-100 dark:from-dark-950 dark:via-dark-900 dark:to-dark-950 transition-colors duration-500"
  >
    <!-- Custom Cursor -->
    <div
      v-if="isFinePonter"
      class="pointer-events-none fixed z-[9999]"
      :class="cursorVisible ? 'opacity-100' : 'opacity-0'"
      :style="{ left: cursorX + 'px', top: cursorY + 'px', transform: 'translate(-50%, -50%)' }"
    >
      <div
        class="rounded-full"
        :class="cursorHovering ? 'cursor-ring-active' : 'cursor-dot-active'"
        :style="{
          width: cursorHovering ? '40px' : '20px',
          height: cursorHovering ? '40px' : '20px',
          background: cursorHovering ? 'rgba(20,184,166,0.07)' : (isDark ? '#ffffff' : '#111827'),
          border: cursorHovering ? '1.5px solid rgba(20,184,166,0.85)' : 'none',
        }"
      ></div>
    </div>
    <!-- ========== BREATHING BLOB BACKGROUND ========== -->
    <div class="pointer-events-none fixed inset-0 z-0 overflow-hidden">
      <div class="blob absolute -left-40 -top-40 h-[800px] w-[800px] rounded-full bg-primary-400/15 dark:bg-primary-500/10 blur-[130px]" style="--duration: 7s; --delay: 0s;"></div>
      <div class="blob absolute -right-48 top-1/3 h-[600px] w-[600px] rounded-full bg-teal-400/10 dark:bg-teal-500/8 blur-[110px]" style="--duration: 10s; --delay: -3s;"></div>
      <div class="blob absolute -bottom-40 left-1/4 h-[500px] w-[500px] rounded-full bg-indigo-400/8 dark:bg-indigo-500/5 blur-[90px]" style="--duration: 13s; --delay: -6s;"></div>
      <div class="blob absolute left-1/2 top-1/2 h-[450px] w-[450px] -translate-x-1/2 -translate-y-1/2 rounded-full bg-primary-500/8 dark:bg-primary-600/5 blur-[100px]" style="--duration: 9s; --delay: -2s;"></div>
    </div>

    <!-- ========== FLOATING NAV ========== -->
    <header class="fixed left-0 right-0 top-4 z-50 flex justify-center px-4 pointer-events-none">
      <nav class="nav-premium pointer-events-auto relative flex w-full max-w-4xl items-center justify-between rounded-2xl border border-black/[0.06] dark:border-white/[0.07] bg-white/80 dark:bg-dark-900/70 px-5 py-3 shadow-glass backdrop-blur-xl transition-all duration-500">
        <!-- Logo -->
        <div class="flex items-center gap-3">
          <div class="relative flex h-9 w-9 items-center justify-center rounded-xl">
            <img :src="siteLogo || '/logo.png'" alt="Logo" class="relative z-10 h-full w-full object-contain p-0.5 transition-all duration-300 invert dark:invert-0" />
          </div>
          <div class="hidden sm:block">
            <span class="nav-site-name font-display text-base font-bold tracking-tight">{{ siteName }}</span>
          </div>
        </div>

        <!-- Actions -->
        <div class="flex items-center gap-1.5 md:gap-2">
          <div class="flex items-center gap-0.5">
            <LocaleSwitcher />
            <a v-if="docUrl" :href="docUrl" target="_blank" rel="noopener noreferrer"
              class="flex h-8 w-8 items-center justify-center rounded-xl text-gray-500 transition-all hover:bg-gray-100 hover:text-gray-900 dark:text-dark-400 dark:hover:bg-dark-800 dark:hover:text-white"
              :title="t('home.viewDocs')">
              <Icon name="book" size="md" />
            </a>
            <button @click="toggleTheme"
              class="flex h-8 w-8 items-center justify-center rounded-xl text-gray-500 transition-all hover:bg-gray-100 hover:text-gray-900 dark:text-dark-400 dark:hover:bg-dark-800 dark:hover:text-white"
              :title="isDark ? t('home.switchToLight') : t('home.switchToDark')">
              <Icon v-if="isDark" name="sun" size="md" />
              <Icon v-else name="moon" size="md" />
            </button>
          </div>
          <div class="mx-1 h-4 w-px bg-gray-200 dark:bg-dark-700"></div>
          <div class="flex items-center gap-2">
            <template v-if="!isAuthenticated">
              <router-link to="/login" class="rounded-xl px-3 py-1.5 text-sm font-semibold text-gray-600 transition-all hover:bg-gray-100 hover:text-gray-900 dark:text-dark-300 dark:hover:bg-dark-800 dark:hover:text-white">
                {{ t('home.login') }}
              </router-link>
              <router-link to="/login?tab=register" class="group relative overflow-hidden rounded-xl bg-primary-500 px-4 py-1.5 text-sm font-bold text-white shadow-glow transition-all hover:scale-105 hover:bg-primary-600 active:scale-95">
                <span class="relative z-10">{{ t('home.getStarted') }}</span>
                <div class="absolute inset-0 -translate-x-full bg-gradient-to-r from-transparent via-white/25 to-transparent transition-transform duration-700 group-hover:translate-x-full"></div>
              </router-link>
            </template>
            <template v-else>
              <router-link :to="dashboardPath" class="group flex items-center gap-2 rounded-xl bg-primary-600 px-4 py-1.5 text-sm font-bold text-white transition-all hover:scale-105 active:scale-95">
                <span class="flex h-5 w-5 items-center justify-center rounded-lg bg-primary-500 dark:bg-primary-400/30 text-[10px] font-bold text-white">{{ userInitial }}</span>
                {{ t('home.dashboard') }}
                <Icon name="arrowRight" size="sm" class="transition-transform group-hover:translate-x-0.5" />
              </router-link>
            </template>
          </div>
        </div>
      </nav>
    </header>

    <!-- ========== HERO SECTION ========== -->
    <section class="relative flex min-h-screen flex-col items-center justify-center overflow-hidden px-6 pb-20 pt-28 text-center">
      <!-- Static glow center -->
      <div class="pointer-events-none absolute inset-x-0 top-1/2 -translate-y-1/2 flex justify-center">
        <div class="h-[500px] w-[700px] rounded-full bg-primary-500/8 dark:bg-primary-500/5 blur-[160px]"></div>
      </div>

      <!-- Badge -->
      <div class="relative z-10 mb-10 opacity-0 animate-fade-in [animation-delay:100ms] [animation-fill-mode:forwards]">
        <span class="inline-flex items-center gap-2 rounded-full border border-primary-300/50 dark:border-primary-500/25 bg-primary-50 dark:bg-primary-500/10 px-5 py-2 text-sm font-semibold text-primary-700 dark:text-primary-400 shadow-glass-sm backdrop-blur-sm">
          <Icon name="sparkles" size="sm" />
          AI API Gateway Platform
        </span>
      </div>

      <!-- Headline -->
      <h1 class="relative z-10 mb-6 hero-title font-display font-extrabold tracking-tight text-gray-900 dark:text-white opacity-0 animate-slide-up [animation-delay:200ms] [animation-fill-mode:forwards]">
        <span class="block">{{ siteName }}</span>
        <span class="animated-gradient mt-2 block pb-2">{{ siteSubtitle }}</span>
      </h1>

      <!-- Description -->
      <p class="relative z-10 mb-12 max-w-2xl text-lg leading-relaxed text-gray-500 dark:text-dark-400 md:text-xl opacity-0 animate-slide-up [animation-delay:370ms] [animation-fill-mode:forwards]">
        {{ t('home.heroDescription') }}
      </p>

      <!-- CTA Buttons -->
      <div class="relative z-10 mb-16 flex flex-col items-center gap-4 sm:flex-row opacity-0 animate-slide-up [animation-delay:520ms] [animation-fill-mode:forwards]">
        <router-link
          :to="isAuthenticated ? dashboardPath : '/login'"
          class="group relative flex items-center gap-2.5 overflow-hidden rounded-2xl bg-primary-500 px-9 py-4 text-base font-bold text-white shadow-glow-lg transition-all hover:scale-105 active:scale-95"
        >
          <span class="relative z-10">{{ isAuthenticated ? t('home.goToDashboard') : t('home.getStarted') }}</span>
          <Icon name="arrowRight" size="md" class="relative z-10 transition-transform group-hover:translate-x-1" />
          <div class="absolute inset-0 -translate-x-full bg-gradient-to-r from-transparent via-white/20 to-transparent transition-transform duration-700 group-hover:translate-x-full"></div>
        </router-link>
        <a v-if="docUrl" :href="docUrl" target="_blank"
          class="flex items-center gap-2 rounded-2xl border border-gray-200 dark:border-dark-700 bg-white dark:bg-dark-900 px-9 py-4 text-base font-semibold text-gray-700 dark:text-gray-300 transition-all hover:scale-105 hover:bg-gray-50 dark:hover:bg-dark-800">
          <Icon name="book" size="md" />
          {{ t('home.viewDocs') }}
        </a>
      </div>

      <!-- Scroll indicator -->
      <div class="bounce-arrow absolute bottom-10 left-1/2 -translate-x-1/2 text-gray-300 dark:text-dark-700">
        <Icon name="chevronDown" size="lg" />
      </div>
    </section>

    <!-- ========== MAIN CONTENT ========== -->
    <div class="relative z-10 px-6">
      <div class="mx-auto max-w-6xl space-y-32 pb-24">

        <!-- === FEATURE TAGS === -->
        <section class="flex flex-wrap items-center justify-center gap-4 md:gap-6">
          <div class="animate-float [animation-delay:0ms] flex items-center gap-3 rounded-2xl border border-gray-200/60 dark:border-white/[0.07] bg-white/70 dark:bg-dark-900/40 px-6 py-3.5 shadow-glass-sm backdrop-blur-md transition-all hover:scale-[1.04]">
            <div class="flex h-9 w-9 items-center justify-center rounded-xl bg-primary-500/10 text-primary-600 dark:text-primary-400">
              <Icon name="swap" size="sm" />
            </div>
            <span class="font-bold text-gray-700 dark:text-gray-200">{{ t('home.tags.subscriptionToApi') }}</span>
          </div>
          <div class="animate-float [animation-delay:800ms] flex items-center gap-3 rounded-2xl border border-gray-200/60 dark:border-white/[0.07] bg-white/70 dark:bg-dark-900/40 px-6 py-3.5 shadow-glass-sm backdrop-blur-md transition-all hover:scale-[1.04]">
            <div class="flex h-9 w-9 items-center justify-center rounded-xl bg-teal-500/10 text-teal-600 dark:text-teal-400">
              <Icon name="shield" size="sm" />
            </div>
            <span class="font-bold text-gray-700 dark:text-gray-200">{{ t('home.tags.stickySession') }}</span>
          </div>
          <div class="animate-float [animation-delay:1600ms] flex items-center gap-3 rounded-2xl border border-gray-200/60 dark:border-white/[0.07] bg-white/70 dark:bg-dark-900/40 px-6 py-3.5 shadow-glass-sm backdrop-blur-md transition-all hover:scale-[1.04]">
            <div class="flex h-9 w-9 items-center justify-center rounded-xl bg-indigo-500/10 text-indigo-600 dark:text-indigo-400">
              <Icon name="chartBar" size="sm" />
            </div>
            <span class="font-bold text-gray-700 dark:text-gray-200">{{ t('home.tags.realtimeBilling') }}</span>
          </div>
        </section>

        <!-- === BENTO FEATURES GRID (gradient border cards) === -->
        <section class="grid grid-cols-1 gap-5 md:grid-cols-12">
          <div class="glow-card group relative overflow-hidden rounded-3xl border border-gray-200/60 dark:border-white/[0.07] bg-white/60 dark:bg-white/[0.03] p-8 shadow-card backdrop-blur-md transition-all duration-500 hover:shadow-card-hover md:col-span-8">
            <div class="mb-6 flex h-14 w-14 items-center justify-center rounded-2xl bg-gradient-to-br from-blue-500 to-indigo-600 shadow-lg shadow-blue-500/25 transition-transform duration-300 group-hover:scale-110">
              <Icon name="server" size="lg" class="text-white" />
            </div>
            <h3 class="mb-3 font-display text-2xl font-bold text-gray-900 dark:text-white">{{ t('home.features.unifiedGateway') }}</h3>
            <p class="max-w-sm text-lg leading-relaxed text-gray-500 dark:text-dark-400">{{ t('home.features.unifiedGatewayDesc') }}</p>
            <div class="pointer-events-none absolute -right-12 -top-12 h-56 w-56 rounded-full bg-blue-500/5 blur-3xl transition-all group-hover:bg-blue-500/10"></div>
          </div>

          <div class="glow-card group relative overflow-hidden rounded-3xl border border-gray-200/60 dark:border-white/[0.07] bg-white/60 dark:bg-white/[0.03] p-8 shadow-card backdrop-blur-md transition-all duration-500 hover:shadow-card-hover md:col-span-4">
            <div class="mb-6 flex h-14 w-14 items-center justify-center rounded-2xl bg-gradient-to-br from-primary-500 to-teal-500 shadow-lg shadow-primary-500/25 transition-transform duration-300 group-hover:scale-110">
              <Icon name="users" size="lg" class="text-white" />
            </div>
            <h3 class="mb-3 font-display text-xl font-bold text-gray-900 dark:text-white">{{ t('home.features.multiAccount') }}</h3>
            <p class="text-gray-500 dark:text-dark-400">{{ t('home.features.multiAccountDesc') }}</p>
          </div>

          <div class="glow-card group relative overflow-hidden rounded-3xl border border-gray-200/60 dark:border-white/[0.07] bg-white/60 dark:bg-white/[0.03] p-8 shadow-card backdrop-blur-md transition-all duration-500 hover:shadow-card-hover md:col-span-4">
            <div class="mb-6 flex h-14 w-14 items-center justify-center rounded-2xl bg-gradient-to-br from-purple-500 to-pink-600 shadow-lg shadow-purple-500/25 transition-transform duration-300 group-hover:scale-110">
              <Icon name="chart" size="lg" class="text-white" />
            </div>
            <h3 class="mb-3 font-display text-xl font-bold text-gray-900 dark:text-white">{{ t('home.features.balanceQuota') }}</h3>
            <p class="text-gray-500 dark:text-dark-400">{{ t('home.features.balanceQuotaDesc') }}</p>
          </div>

          <div class="group relative flex flex-col justify-center overflow-hidden rounded-3xl bg-gray-950 p-8 shadow-card transition-all duration-500 hover:shadow-card-hover md:col-span-8">
            <div class="relative z-10">
              <h3 class="mb-4 font-display text-3xl font-bold text-white">{{ t('home.heroSubtitle') }}</h3>
              <router-link :to="isAuthenticated ? dashboardPath : '/login'"
                class="inline-flex items-center gap-2 font-bold text-primary-400 transition-all hover:text-primary-300 hover:gap-3">
                {{ t('home.getStarted') }}
                <Icon name="arrowRight" size="sm" />
              </router-link>
            </div>
            <div class="pointer-events-none absolute right-0 top-0 h-full w-1/2 bg-gradient-to-l from-primary-500/10 to-transparent"></div>
            <div class="pointer-events-none absolute -bottom-16 -right-16 h-56 w-56 rounded-full bg-primary-500/10 blur-3xl"></div>
          </div>
        </section>

        <!-- === PAIN POINTS === -->
        <section>
          <div class="mb-14 text-center">
            <h2 class="font-display text-4xl font-bold text-gray-900 dark:text-white md:text-5xl">{{ t('home.painPoints.title') }}</h2>
          </div>
          <div class="grid grid-cols-1 gap-5 sm:grid-cols-2 lg:grid-cols-4">
            <div class="glow-card group rounded-3xl border border-gray-200/60 dark:border-white/[0.07] bg-white/60 dark:bg-white/[0.03] p-7 shadow-card backdrop-blur-md transition-all duration-300 hover:shadow-card-hover hover:scale-[1.02]">
              <div class="mb-4 flex h-12 w-12 items-center justify-center rounded-2xl bg-orange-500/10 text-orange-500">
                <Icon name="dollar" size="md" />
              </div>
              <h4 class="mb-2 font-display text-lg font-bold text-gray-900 dark:text-white">{{ t('home.painPoints.items.expensive.title') }}</h4>
              <p class="text-sm leading-relaxed text-gray-500 dark:text-dark-400">{{ t('home.painPoints.items.expensive.desc') }}</p>
            </div>
            <div class="glow-card group rounded-3xl border border-gray-200/60 dark:border-white/[0.07] bg-white/60 dark:bg-white/[0.03] p-7 shadow-card backdrop-blur-md transition-all duration-300 hover:shadow-card-hover hover:scale-[1.02]">
              <div class="mb-4 flex h-12 w-12 items-center justify-center rounded-2xl bg-yellow-500/10 text-yellow-600 dark:text-yellow-400">
                <Icon name="cog" size="md" />
              </div>
              <h4 class="mb-2 font-display text-lg font-bold text-gray-900 dark:text-white">{{ t('home.painPoints.items.complex.title') }}</h4>
              <p class="text-sm leading-relaxed text-gray-500 dark:text-dark-400">{{ t('home.painPoints.items.complex.desc') }}</p>
            </div>
            <div class="glow-card group rounded-3xl border border-gray-200/60 dark:border-white/[0.07] bg-white/60 dark:bg-white/[0.03] p-7 shadow-card backdrop-blur-md transition-all duration-300 hover:shadow-card-hover hover:scale-[1.02]">
              <div class="mb-4 flex h-12 w-12 items-center justify-center rounded-2xl bg-red-500/10 text-red-500">
                <Icon name="exclamationTriangle" size="md" />
              </div>
              <h4 class="mb-2 font-display text-lg font-bold text-gray-900 dark:text-white">{{ t('home.painPoints.items.unstable.title') }}</h4>
              <p class="text-sm leading-relaxed text-gray-500 dark:text-dark-400">{{ t('home.painPoints.items.unstable.desc') }}</p>
            </div>
            <div class="glow-card group rounded-3xl border border-gray-200/60 dark:border-white/[0.07] bg-white/60 dark:bg-white/[0.03] p-7 shadow-card backdrop-blur-md transition-all duration-300 hover:shadow-card-hover hover:scale-[1.02]">
              <div class="mb-4 flex h-12 w-12 items-center justify-center rounded-2xl bg-purple-500/10 text-purple-500">
                <Icon name="ban" size="md" />
              </div>
              <h4 class="mb-2 font-display text-lg font-bold text-gray-900 dark:text-white">{{ t('home.painPoints.items.noControl.title') }}</h4>
              <p class="text-sm leading-relaxed text-gray-500 dark:text-dark-400">{{ t('home.painPoints.items.noControl.desc') }}</p>
            </div>
          </div>
        </section>

        <!-- === COMPARISON TABLE === -->
        <section>
          <div class="mb-14 text-center">
            <h2 class="font-display text-4xl font-bold text-gray-900 dark:text-white md:text-5xl">{{ t('home.comparison.title') }}</h2>
          </div>
          <div class="overflow-hidden rounded-3xl border border-gray-200/60 dark:border-white/[0.07] bg-white/60 dark:bg-white/[0.03] shadow-card backdrop-blur-md">
            <div class="grid grid-cols-3 border-b border-gray-100 dark:border-white/[0.06]">
              <div class="p-5 text-sm font-semibold uppercase tracking-wider text-gray-400 dark:text-dark-500">{{ t('home.comparison.headers.feature') }}</div>
              <div class="p-5 text-center text-sm font-semibold text-gray-500 dark:text-dark-400">{{ t('home.comparison.headers.official') }}</div>
              <div class="p-5 text-center text-sm font-bold text-primary-600 dark:text-primary-400 bg-primary-50 dark:bg-primary-500/10">{{ t('home.comparison.headers.us') }}</div>
            </div>
            <div v-for="row in comparisonRows" :key="row" class="grid grid-cols-3 border-b border-gray-100/80 dark:border-white/[0.04] last:border-0 transition-colors hover:bg-gray-50/60 dark:hover:bg-white/[0.02]">
              <div class="p-5 font-medium text-gray-700 dark:text-gray-300">{{ t(`home.comparison.items.${row}.feature`) }}</div>
              <div class="p-5 text-center text-gray-500 dark:text-dark-400">{{ t(`home.comparison.items.${row}.official`) }}</div>
              <div class="p-5 text-center font-semibold text-primary-600 dark:text-primary-400 bg-primary-50/60 dark:bg-primary-500/10">{{ t(`home.comparison.items.${row}.us`) }}</div>
            </div>
          </div>
        </section>

        <!-- === PROVIDERS === -->
        <section>
          <div class="mb-10 text-center">
            <h2 class="mb-3 font-display text-3xl font-bold text-gray-900 dark:text-white">{{ t('home.providers.title') }}</h2>
            <p class="text-gray-500 dark:text-dark-400">{{ t('home.providers.description') }}</p>
          </div>
          <div class="flex flex-wrap items-center justify-center gap-4">
            <div class="group flex items-center gap-3 rounded-2xl border border-gray-200/60 dark:border-white/[0.07] bg-white/60 dark:bg-dark-900/40 px-6 py-3 shadow-glass-sm backdrop-blur-md transition-all hover:scale-105 hover:border-orange-400/40 dark:hover:border-orange-500/30">
              <div class="flex h-8 w-8 items-center justify-center rounded-lg bg-gradient-to-br from-orange-400 to-orange-600 text-xs font-bold text-white">C</div>
              <span class="font-bold text-gray-700 dark:text-gray-200">{{ t('home.providers.claude') }}</span>
              <span class="rounded-full bg-green-500/10 px-2 py-0.5 text-[10px] font-bold text-green-600 dark:text-green-400">Live</span>
            </div>
            <div class="group flex items-center gap-3 rounded-2xl border border-gray-200/60 dark:border-white/[0.07] bg-white/60 dark:bg-dark-900/40 px-6 py-3 shadow-glass-sm backdrop-blur-md transition-all hover:scale-105 hover:border-emerald-400/40 dark:hover:border-emerald-500/30">
              <div class="flex h-8 w-8 items-center justify-center rounded-lg bg-gradient-to-br from-emerald-500 to-teal-600 text-xs font-bold text-white">G</div>
              <span class="font-bold text-gray-700 dark:text-gray-200">GPT-4</span>
              <span class="rounded-full bg-green-500/10 px-2 py-0.5 text-[10px] font-bold text-green-600 dark:text-green-400">Live</span>
            </div>
            <div class="group flex items-center gap-3 rounded-2xl border border-gray-200/60 dark:border-white/[0.07] bg-white/60 dark:bg-dark-900/40 px-6 py-3 shadow-glass-sm backdrop-blur-md transition-all hover:scale-105 hover:border-blue-400/40 dark:hover:border-blue-500/30">
              <div class="flex h-8 w-8 items-center justify-center rounded-lg bg-gradient-to-br from-blue-500 to-indigo-600 text-xs font-bold text-white">G</div>
              <span class="font-bold text-gray-700 dark:text-gray-200">{{ t('home.providers.gemini') }}</span>
              <span class="rounded-full bg-green-500/10 px-2 py-0.5 text-[10px] font-bold text-green-600 dark:text-green-400">Live</span>
            </div>
            <div class="group flex items-center gap-3 rounded-2xl border border-gray-200/60 dark:border-white/[0.07] bg-white/60 dark:bg-dark-900/40 px-6 py-3 shadow-glass-sm backdrop-blur-md transition-all hover:scale-105 hover:border-rose-400/40 dark:hover:border-rose-500/30">
              <div class="flex h-8 w-8 items-center justify-center rounded-lg bg-gradient-to-br from-rose-500 to-pink-600 text-xs font-bold text-white">A</div>
              <span class="font-bold text-gray-700 dark:text-gray-200">{{ t('home.providers.antigravity') }}</span>
              <span class="rounded-full bg-green-500/10 px-2 py-0.5 text-[10px] font-bold text-green-600 dark:text-green-400">Live</span>
            </div>
            <div class="flex items-center gap-3 rounded-2xl border border-dashed border-gray-300 dark:border-white/10 px-6 py-3 transition-all hover:border-primary-400/40 dark:hover:border-primary-500/30">
              <div class="flex h-8 w-8 items-center justify-center rounded-lg bg-gray-100 dark:bg-dark-800 text-sm font-bold text-gray-400">+</div>
              <span class="font-bold text-gray-400 dark:text-dark-500">{{ t('home.providers.more') }}</span>
            </div>
          </div>
        </section>

      </div>
    </div>

    <!-- ========== CTA SECTION ========== -->
    <section class="relative mt-8 px-6 py-32">
      <div class="relative z-10 mx-auto max-w-3xl text-center">
        <h2 class="mb-6 font-display text-4xl font-extrabold text-gray-900 dark:text-white md:text-5xl lg:text-6xl">{{ t('home.cta.title') }}</h2>
        <p class="mb-12 text-lg text-gray-500 dark:text-gray-400 md:text-xl">{{ t('home.cta.description') }}</p>
        <router-link
          :to="isAuthenticated ? dashboardPath : '/login?tab=register'"
          class="group relative inline-flex items-center gap-3 overflow-hidden rounded-2xl bg-primary-500 px-10 py-5 text-lg font-bold text-white transition-all hover:scale-105 hover:bg-primary-600 active:scale-95 shadow-glow-lg"
        >
          <span class="relative z-10">{{ t('home.cta.button') }}</span>
          <Icon name="arrowRight" size="md" class="relative z-10 transition-transform group-hover:translate-x-1" />
          <div class="absolute inset-0 -translate-x-full bg-gradient-to-r from-transparent via-white/20 to-transparent transition-transform duration-700 group-hover:translate-x-full"></div>
        </router-link>
      </div>
    </section>

    <!-- ========== FOOTER ========== -->
    <footer class="border-t border-gray-200/50 dark:border-white/[0.05] bg-white/40 dark:bg-transparent px-6 py-10 backdrop-blur-sm">
      <div class="mx-auto max-w-6xl flex flex-col items-center justify-between gap-6 md:flex-row">
        <div class="flex items-center gap-3">
          <div class="h-8 w-8 overflow-hidden rounded-lg opacity-75">
            <img :src="siteLogo || '/logo.png'" alt="Logo" class="h-full w-full object-contain transition-all duration-300 white:invert" />
          </div>
          <span class="font-display text-base font-bold text-gray-900 dark:text-white">{{ siteName }}</span>
        </div>
        <div class="flex items-center gap-8">
          <a v-if="docUrl" :href="docUrl" target="_blank" class="text-sm font-semibold text-gray-500 dark:text-dark-400 transition-colors hover:text-primary-500 dark:hover:text-primary-400">{{ t('home.docs') }}</a>
        </div>
        <p class="text-sm text-gray-400 dark:text-dark-600">&copy; {{ currentYear }} {{ siteName }}. {{ t('home.footer.allRightsReserved') }}</p>
      </div>
    </footer>

  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
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

// Comparison table rows
const comparisonRows = ['pricing', 'models', 'management', 'stability', 'control']

// Custom cursor
const cursorX = ref(-200)
const cursorY = ref(-200)
const cursorHovering = ref(false)
const cursorVisible = ref(false)
const isFinePonter = ref(false)

function onCursorMove(e: MouseEvent) {
  cursorX.value = e.clientX
  cursorY.value = e.clientY
  cursorVisible.value = true
}

function onCursorOver(e: MouseEvent) {
  const target = e.target as HTMLElement
  cursorHovering.value = !!target.closest('a, button, [role="button"], input, select, textarea, label, [tabindex="0"]')
}

function onCursorLeave() {
  cursorVisible.value = false
}

onMounted(() => {
  initTheme()
  authStore.checkAuth()
  if (!appStore.publicSettingsLoaded) {
    appStore.fetchPublicSettings()
  }

  // Custom cursor - only on mouse devices
  isFinePonter.value = window.matchMedia('(pointer: fine)').matches
  if (isFinePonter.value) {
    document.documentElement.classList.add('home-cursor')
    document.addEventListener('mousemove', onCursorMove)
    document.addEventListener('mouseover', onCursorOver)
    document.addEventListener('mouseleave', onCursorLeave)
  }
})

onUnmounted(() => {
  document.documentElement.classList.remove('home-cursor')
  document.removeEventListener('mousemove', onCursorMove)
  document.removeEventListener('mouseover', onCursorOver)
  document.removeEventListener('mouseleave', onCursorLeave)
})
</script>

<style scoped>
/* ===== CUSTOM CURSOR ===== */
.cursor-dot-active {
  transition: width 0.18s ease, height 0.18s ease, background 0.18s ease, border 0.18s ease;
}

.cursor-ring-active {
  animation: cursor-ring-pop 0.4s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}

@keyframes cursor-ring-pop {
  0%   { transform: scale(0.2); opacity: 0; }
  60%  { transform: scale(1.18); opacity: 1; }
  100% { transform: scale(1);   opacity: 1; }
}

/* ===== GLOBAL CURSOR HIDE ===== */
:global(.home-cursor),
:global(.home-cursor *) {
  cursor: none !important;
}

/* ===== NOISE TEXTURE OVERLAY ===== */
.noise-bg::after {
  content: '';
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 999;
  opacity: 0.022;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
  background-repeat: repeat;
  background-size: 128px 128px;
}

/* ===== HERO TITLE ===== */
.hero-title {
  font-size: clamp(3rem, 8.5vw, 6.5rem);
  line-height: 1.04;
}

/* ===== ANIMATED GRADIENT TEXT ===== */
.animated-gradient {
  background: linear-gradient(270deg, #14b8a6, #6366f1, #2dd4bf, #a78bfa, #14b8a6);
  background-size: 300% auto;
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradient-flow 8s ease infinite;
}

@keyframes gradient-flow {
  0% { background-position: 0% center; }
  50% { background-position: 100% center; }
  100% { background-position: 0% center; }
}

/* ===== BREATHING BLOBS ===== */
@keyframes breathe {
  0%, 100% {
    transform: scale(1) translateY(0px);
    opacity: 0.7;
  }
  50% {
    transform: scale(1.14) translateY(-20px);
    opacity: 1;
  }
}

.blob {
  animation: breathe var(--duration, 7s) ease-in-out infinite;
  animation-delay: var(--delay, 0s);
}

/* ===== FLOATING BADGE ===== */
@keyframes badge-float-anim {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-6px); }
}

.badge-float {
  animation: badge-float-anim 3.5s ease-in-out infinite;
}

/* ===== SCROLL INDICATOR ===== */
@keyframes bounce-down {
  0%, 100% { transform: translateY(0) translateX(-50%); }
  50% { transform: translateY(9px) translateX(-50%); }
}

.bounce-arrow {
  animation: bounce-down 1.8s ease-in-out infinite;
}

/* ===== MARQUEE ===== */
@keyframes marquee-scroll {
  from { transform: translateX(0); }
  to { transform: translateX(-50%); }
}

.marquee-track {
  animation: marquee-scroll 30s linear infinite;
  width: max-content;
}

.marquee-track:hover {
  animation-play-state: paused;
}

/* ===== NAV PREMIUM ===== */

.nav-logo-ring {
  box-shadow: 0 0 0 1.5px rgba(20, 184, 166, 0.35), 0 0 12px rgba(20, 184, 166, 0.15);
}

.nav-site-name {
  background: linear-gradient(135deg, #111827 0%, #374151 100%);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.dark .nav-site-name {
  background: linear-gradient(135deg, #f9fafb 0%, #d1d5db 100%);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

/* ===== GRADIENT BORDER CARDS ===== */
.glow-card {
  position: relative;
}

.glow-card::before {
  content: '';
  position: absolute;
  inset: 0;
  border-radius: inherit;
  padding: 1px;
  background: linear-gradient(
    135deg,
    rgba(20, 184, 166, 0.5) 0%,
    transparent 40%,
    transparent 60%,
    rgba(99, 102, 241, 0.5) 100%
  );
  -webkit-mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0);
  mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;
  opacity: 0;
  transition: opacity 0.4s ease;
  pointer-events: none;
  z-index: 1;
}

.glow-card:hover::before {
  opacity: 1;
}

/* ===== CTA BUTTON PULSING GLOW ===== */
@keyframes cta-pulse {
  0%, 100% { box-shadow: 0 0 20px rgba(20, 184, 166, 0.4), 0 0 40px rgba(20, 184, 166, 0.2); }
  50% { box-shadow: 0 0 30px rgba(20, 184, 166, 0.6), 0 0 60px rgba(20, 184, 166, 0.3); }
}

.cta-glow {
  animation: cta-pulse 3s ease-in-out infinite;
}

/* ===== PREMIUM SLIDE UP (override tailwind) ===== */
.animate-slide-up {
  animation: slide-up-in 0.8s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}

@keyframes slide-up-in {
  from { opacity: 0; transform: translateY(24px); }
  to { opacity: 1; transform: translateY(0); }
}

.animate-fade-in {
  animation: fade-in-anim 0.6s ease-out forwards;
}

@keyframes fade-in-anim {
  from { opacity: 0; }
  to { opacity: 1; }
}
</style>
