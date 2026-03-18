<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import { translations } from './i18n.js'

// 语言状态管理
const currentLang = ref(localStorage.getItem('language') || 'zh')

// 主题状态管理 - gura(蓝色) 或 chocolate(巧克力深色)
const currentTheme = ref(localStorage.getItem('theme') || 'gura')

// 切换语言
const toggleLanguage = () => {
  const newLang = currentLang.value === 'zh' ? 'en' : 'zh'
  currentLang.value = newLang
  localStorage.setItem('language', newLang)
  document.documentElement.lang = newLang === 'zh' ? 'zh-CN' : 'en'
  updatePageTitle()
}

// 切换主题
const toggleTheme = () => {
  const newTheme = currentTheme.value === 'gura' ? 'chocolate' : 'gura'
  currentTheme.value = newTheme
  localStorage.setItem('theme', newTheme)
  document.documentElement.setAttribute('data-theme', newTheme)
}

// 监听主题变化
watch(currentTheme, (newTheme) => {
  document.documentElement.setAttribute('data-theme', newTheme)
})

// 主题名称
const themeName = computed(() => {
  const isGura = currentTheme.value === 'gura'
  return currentLang.value === 'zh'
    ? (isGura ? 'Gura 蓝' : '巧克力')
    : (isGura ? 'Gura Blue' : 'Chocolate')
})

// 更新页面标题
const updatePageTitle = () => {
  document.title = currentLang.value === 'zh'
    ? '瑶光流梦 - 让游戏串流更优雅'
    : 'SDream - Make Game Streaming Greater'
}

// 当前语言的翻译内容
const t = computed(() => translations[currentLang.value])

// Star History 图表状态
const starHistoryLoaded = ref(false)
const starHistoryError = ref(false)

// 版本信息状态
const versionInfo = ref({
  current: null,
  latest: null,
  preRelease: null,
  loading: true,
  error: null,
})

// 国内镜像前缀
const MIRROR_PREFIX = 'https://mirror.ghproxy.com/'

// 下载链接
const downloadLinks = ref({
  windows: 'https://github.com/qiin2333/Sunshine-Foundation/releases/latest',
  github: 'https://github.com/qiin2333/Sunshine-Foundation/releases/',
  mirror: 'https://github.com/qiin2333/Sunshine-Foundation/releases/latest',
  latest: null,
})

// 提取资源下载链接
const extractDownloadUrl = (assets, filename) =>
  assets.find(asset => asset.name.includes(filename))?.browser_download_url

// 检查最新版本
const checkLatestVersion = async () => {
  try {
    versionInfo.value.loading = true
    versionInfo.value.error = null

    const [latestResponse, allReleasesResponse] = await Promise.all([
      fetch('https://api.github.com/repos/qiin2333/Sunshine-Foundation/releases/latest'),
      fetch('https://api.github.com/repos/qiin2333/Sunshine-Foundation/releases')
    ])

    const [latestRelease, allReleases] = await Promise.all([
      latestResponse.json(),
      allReleasesResponse.json()
    ])

    const preRelease = allReleases.find(release => release.prerelease)
    const installerFilename = 'sunshine-windows-installer.exe'

    versionInfo.value.latest = {
      version: latestRelease.tag_name,
      downloadUrl: extractDownloadUrl(latestRelease.assets, installerFilename),
      releaseUrl: latestRelease.html_url,
      body: latestRelease.body,
    }

    if (preRelease) {
      versionInfo.value.preRelease = {
        version: preRelease.tag_name,
        downloadUrl: extractDownloadUrl(preRelease.assets, installerFilename),
        releaseUrl: preRelease.html_url,
        body: preRelease.body,
      }
    }

    const latestDownloadUrl = versionInfo.value.latest.downloadUrl
    if (latestDownloadUrl) {
      downloadLinks.value.latest = latestDownloadUrl
      downloadLinks.value.windows = latestDownloadUrl
      downloadLinks.value.mirror = `${MIRROR_PREFIX}${latestDownloadUrl}`
    }
  } catch (error) {
    console.error('版本检查失败:', error)
    versionInfo.value.error = error.message
    const fallbackUrl = 'https://vip.123pan.cn/1813496318/26878949'
    downloadLinks.value.windows = fallbackUrl
    downloadLinks.value.mirror = fallbackUrl
  } finally {
    versionInfo.value.loading = false
  }
}

onMounted(() => {
  document.documentElement.setAttribute('data-theme', currentTheme.value)
  document.documentElement.lang = currentLang.value === 'zh' ? 'zh-CN' : 'en'
  updatePageTitle()

  const img = new Image()
  img.onload = () => { starHistoryLoaded.value = true }
  img.onerror = () => { starHistoryError.value = true }
  img.src = 'https://api.star-history.com/svg?repos=qiin2333/Sunshine-Foundation&type=Date&width=800&height=400'

  checkLatestVersion()
})

// 客户端推荐
const clients = [
  {
    name: 'Moonlight-PC',
    platform: 'Windows/macOS/Linux',
    link: 'https://github.com/qiin2333/moonlight-qt',
  },
  {
    name: 'VPLUS Moonlight-Android',
    platform: 'Android',
    link: 'https://github.com/qiin2333/moonlight-vplus',
  },
  {
    name: '王冠版 Moonlight-Android',
    platform: 'Android',
    link: 'https://github.com/WACrown/moonlight-android',
  },
  {
    name: 'VoidLink (Moonlight-iOS)',
    platform: 'iOS',
    link: 'https://apps.apple.com/cn/app/voidlink/id6747717070',
  },
]
</script>

<template>
  <div class="website" :data-theme="currentTheme">
    <!-- 头部导航 -->
    <header class="header">
      <div class="container">
        <nav class="nav">
          <a href="#" class="logo">
            <span class="logo-name">{{ t.title }}</span>
            <span class="logo-badge">Beta</span>
          </a>

          <div class="nav-center">
            <a href="#features" class="nav-link">{{ t.nav.features }}</a>
            <a href="#download" class="nav-link">{{ t.nav.download }}</a>
            <a href="#clients" class="nav-link">{{ t.nav.clients }}</a>
            <a href="#stats" class="nav-link">{{ t.nav.stats }}</a>
            <a href="#docs" class="nav-link">{{ t.nav.docs }}</a>
          </div>

          <div class="nav-controls">
            <button @click="toggleTheme" class="theme-toggle" :title="themeName">
              <svg v-if="currentTheme === 'gura'" xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="5"/><line x1="12" y1="1" x2="12" y2="3"/><line x1="12" y1="21" x2="12" y2="23"/><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/><line x1="1" y1="12" x2="3" y2="12"/><line x1="21" y1="12" x2="23" y2="12"/><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/></svg>
              <svg v-else xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/></svg>
            </button>
            <button @click="toggleLanguage" class="lang-toggle">
              {{ currentLang === 'zh' ? 'EN' : '中文' }}
            </button>
          </div>
        </nav>
      </div>
    </header>

    <!-- 主横幅 -->
    <section class="hero">
      <div class="container">
        <div class="hero-content">
          <p class="hero-badge">Game Streaming Platform</p>
          <h1 class="hero-title">{{ t.tagline }}</h1>
          <p class="hero-subtitle">{{ t.subtitle }}</p>
          <div class="hero-actions">
            <a :href="downloadLinks.windows" class="btn btn-primary">
              {{ t.hero.download }}
            </a>
            <a :href="downloadLinks.github" class="btn btn-outline">
              {{ t.hero.github }}
            </a>
            <a :href="downloadLinks.mirror" class="btn btn-outline">
              {{ t.hero.mirror }}
            </a>
          </div>
          <div class="hero-stats">
            <span class="stat-item">Low Latency</span>
            <span class="stat-divider"></span>
            <span class="stat-item">HDR Support</span>
            <span class="stat-divider"></span>
            <span class="stat-item">Virtual Display</span>
          </div>
        </div>
      </div>
    </section>

    <!-- 核心特性 -->
    <section id="features" class="section">
      <div class="container">
        <div class="section-header">
          <h2 class="section-title">{{ t.features.title }}</h2>
          <div class="section-line"></div>
        </div>
        <div class="features-grid">
          <div
            v-for="(feature, index) in t.features.items"
            :key="feature.title"
            class="feature-card"
          >
            <span class="feature-number">{{ String(index + 1).padStart(2, '0') }}</span>
            <h3 class="feature-title">{{ feature.title }}</h3>
            <p class="feature-desc">{{ feature.description }}</p>
          </div>
        </div>
      </div>
    </section>

    <!-- 下载区域 -->
    <section id="download" class="section section-alt">
      <div class="container">
        <div class="section-header">
          <h2 class="section-title">{{ t.download.title }}</h2>
          <div class="section-line"></div>
        </div>

        <!-- 版本信息 -->
        <div v-if="versionInfo.latest" class="version-info">
          <div class="version-badge">
            <span class="version-dot"></span>
            <span class="version-label">{{ t.download.latestVersion }}</span>
            <span class="version-number">{{ versionInfo.latest.version }}</span>
          </div>
          <button
            @click="checkLatestVersion"
            class="btn-text"
            :disabled="versionInfo.loading"
          >
            {{ t.download.checkUpdate }}
          </button>
        </div>

        <!-- 加载状态 -->
        <div v-if="versionInfo.loading" class="loading-state">
          <div class="loading-spinner"></div>
          <p>{{ t.download.checking }}</p>
        </div>

        <!-- 错误状态 -->
        <div v-if="versionInfo.error" class="error-state">
          <p>{{ t.download.error }}</p>
          <button @click="checkLatestVersion" class="btn btn-outline">
            {{ t.download.retry }}
          </button>
        </div>

        <div class="download-content">
          <div class="download-info">
            <h3>{{ t.download.requirements }}</h3>
            <ul class="requirements-list">
              <li
                v-for="(req, index) in t.download.requirementsList"
                :key="index"
                v-html="req"
              ></li>
            </ul>
          </div>
          <div class="download-actions">
            <a :href="downloadLinks.windows" class="download-btn primary">
              <span class="download-text">
                <strong>{{ t.download.windowsLatest }}</strong>
                <small>{{ versionInfo.latest?.version || t.download.recommended }}</small>
              </span>
              <span class="download-arrow">&rarr;</span>
            </a>
            <a :href="downloadLinks.github" class="download-btn">
              <span class="download-text">
                <strong>{{ t.download.allVersions }}</strong>
                <small>{{ t.download.githubReleases }}</small>
              </span>
              <span class="download-arrow">&rarr;</span>
            </a>
            <a :href="downloadLinks.mirror" class="download-btn">
              <span class="download-text">
                <strong>{{ t.download.mirrorDownload }}</strong>
                <small>{{ t.download.domesticSpeed }}</small>
              </span>
              <span class="download-arrow">&rarr;</span>
            </a>
          </div>
        </div>

        <!-- 预发布版本 -->
        <div v-if="versionInfo.preRelease" class="prerelease-alert">
          <div class="alert-content">
            <h4>{{ t.download.prerelease }}</h4>
            <p>
              {{ t.download.prereleaseFound }}
              <strong>{{ versionInfo.preRelease.version }}</strong>
            </p>
          </div>
          <a
            :href="versionInfo.preRelease.releaseUrl"
            class="btn btn-outline"
            target="_blank"
          >
            {{ t.download.viewPrerelease }}
          </a>
        </div>
      </div>
    </section>

    <!-- 推荐客户端 -->
    <section id="clients" class="section">
      <div class="container">
        <div class="section-header">
          <h2 class="section-title">{{ t.clients.title }}</h2>
          <p class="section-subtitle">{{ t.clients.subtitle }}</p>
          <div class="section-line"></div>
        </div>
        <div class="clients-grid">
          <div v-for="client in clients" :key="client.name" class="client-card">
            <div class="client-info">
              <h3 class="client-name">{{ client.name }}</h3>
              <p class="client-platform">{{ client.platform }}</p>
            </div>
            <a :href="client.link" class="client-link" target="_blank" rel="noopener">
              {{ t.clients.downloadBtn }}
            </a>
          </div>
        </div>
      </div>
    </section>

    <!-- Star History -->
    <section id="stats" class="section section-alt">
      <div class="container">
        <div class="section-header">
          <h2 class="section-title">{{ t.stats.title }}</h2>
          <p class="section-subtitle">{{ t.stats.subtitle }}</p>
          <div class="section-line"></div>
        </div>
        <div class="star-history-container">
          <div v-if="!starHistoryLoaded && !starHistoryError" class="loading-state">
            <div class="loading-spinner"></div>
            <p>{{ t.stats.loading }}</p>
          </div>
          <div v-else-if="starHistoryError" class="error-state">
            <p>{{ t.stats.error }}</p>
            <a
              href="https://star-history.com/#qiin2333/Sunshine-Foundation&Date"
              target="_blank"
              class="btn btn-outline"
            >
              {{ t.stats.viewManually }}
            </a>
          </div>
          <img
            v-else
            src="https://api.star-history.com/svg?repos=qiin2333/Sunshine-Foundation&type=Date&width=800&height=400"
            :alt="`${t.title} Star History`"
            class="star-history-chart"
            loading="lazy"
          />
        </div>
        <div class="stats-actions">
          <a
            href="https://github.com/qiin2333/Sunshine-Foundation"
            class="btn btn-primary"
            target="_blank"
          >
            {{ t.stats.giveStar }}
          </a>
          <a
            href="https://star-history.com/#qiin2333/Sunshine-Foundation&Date"
            class="btn btn-outline"
            target="_blank"
          >
            {{ t.stats.viewStats }}
          </a>
        </div>
      </div>
    </section>

    <!-- 文档链接 -->
    <section id="docs" class="section">
      <div class="container">
        <div class="section-header">
          <h2 class="section-title">{{ t.docs.title }}</h2>
          <div class="section-line"></div>
        </div>
        <div class="docs-grid">
          <a
            href="https://docs.qq.com/aio/DSGdQc3htbFJjSFdO?p=YTpMj5JNNdB5hEKJhhqlSB"
            class="doc-card"
            target="_blank"
          >
            <h3>{{ t.docs.userGuide }}</h3>
            <p>{{ t.docs.userGuideDesc }}</p>
          </a>
          <a
            href="https://docs.lizardbyte.dev/projects/sunshine/latest/"
            class="doc-card"
            target="_blank"
          >
            <h3>{{ t.docs.officialDocs }}</h3>
            <p>{{ t.docs.officialDocsDesc }}</p>
          </a>
          <a
            href="https://qm.qq.com/cgi-bin/qm/qr?k=5qnkzSaLIrIaU4FvumftZH_6Hg7fUuLD&jump_from=webapi"
            class="doc-card"
            target="_blank"
          >
            <h3>{{ t.docs.qqGroup }}</h3>
            <p>{{ t.docs.qqGroupDesc }}</p>
          </a>
        </div>
      </div>
    </section>

    <!-- 页脚 -->
    <footer class="footer">
      <div class="container">
        <div class="footer-content">
          <div class="footer-section footer-brand">
            <h4 class="footer-logo-text">{{ t.footer.title }}</h4>
            <p>{{ t.footer.subtitle }}</p>
          </div>
          <div class="footer-section">
            <h4>{{ t.footer.links }}</h4>
            <ul>
              <li>
                <a href="https://github.com/qiin2333/Sunshine-Foundation" target="_blank">GitHub</a>
              </li>
              <li>
                <a href="https://github.com/LizardByte/awesome-sunshine" target="_blank">
                  awesome-sunshine
                </a>
              </li>
            </ul>
          </div>
        </div>
        <div class="footer-bottom">
          <p>{{ t.footer.copyright }}</p>
          <div class="beian-row">
            <span class="beian-item">
              <img src="/police-badge.webp" :alt="t.footer.policeBadgeAlt" class="beian-badge" />
              <span>{{ t.footer.policePlaceholder }}</span>
            </span>
            <a :href="t.footer.icpLink" class="beian-item" target="_blank" rel="noopener">
              {{ t.footer.icp }}
            </a>
          </div>
        </div>
      </div>
    </footer>
  </div>
</template>

<style lang="less" scoped>
@import './styles/variables.less';

// ============================================
// 朴素简洁风格 - 参考 Open WebUI
// ============================================

.website {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  line-height: 1.6;
  color: var(--text-primary);
  background: var(--background-primary);
  min-height: 100vh;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1.5rem;

  @media (max-width: 640px) {
    padding: 0 1rem;
  }
}

// ============================================
// Header
// ============================================

.header {
  position: sticky;
  top: 0;
  z-index: 100;
  background: var(--background-primary);
  border-bottom: 1px solid var(--border-color);
  transition: background-color 0.3s ease;
}

.nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.75rem 0;
  gap: 1.5rem;
}

.logo {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  text-decoration: none;
  flex-shrink: 0;
}

.logo-name {
  font-size: 1.125rem;
  font-weight: 700;
  color: var(--primary-color);
}

.logo-badge {
  font-size: 0.625rem;
  font-weight: 600;
  background: var(--primary-color);
  color: var(--text-inverse);
  padding: 1px 6px;
  border-radius: 4px;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.nav-center {
  display: flex;
  align-items: center;
  gap: 0.25rem;
}

.nav-link {
  text-decoration: none;
  color: var(--text-secondary);
  font-weight: 500;
  font-size: 0.875rem;
  padding: 0.5rem 0.75rem;
  border-radius: 8px;
  transition: color 0.2s, background-color 0.2s;

  &:hover {
    color: var(--primary-color);
    background: var(--background-secondary);
  }
}

.nav-controls {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  flex-shrink: 0;
}

.theme-toggle {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  border-radius: 8px;
  background: var(--background-secondary);
  border: 1px solid var(--border-color);
  color: var(--text-secondary);
  cursor: pointer;
  transition: all 0.2s;

  &:hover {
    color: var(--primary-color);
    border-color: var(--primary-color);
  }
}

.lang-toggle {
  background: var(--primary-color);
  color: var(--text-inverse);
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 8px;
  font-weight: 600;
  font-size: 0.875rem;
  cursor: pointer;
  transition: opacity 0.2s;

  &:hover {
    opacity: 0.9;
  }
}

// ============================================
// Hero
// ============================================

.hero {
  padding: 6rem 0 4rem;
  text-align: center;
  background: var(--background-primary);
}

.hero-content {
  max-width: 720px;
  margin: 0 auto;
}

.hero-badge {
  display: inline-block;
  font-size: 0.8rem;
  font-weight: 500;
  color: var(--primary-color);
  padding: 0.25rem 0.75rem;
  border: 1px solid var(--primary-color);
  border-radius: 999px;
  margin-bottom: 1.5rem;
  opacity: 0.85;
}

.hero-title {
  font-size: clamp(2.25rem, 6vw, 3.5rem);
  font-weight: 800;
  line-height: 1.15;
  margin: 0 0 1.25rem;
  color: var(--text-primary);
}

.hero-subtitle {
  font-size: 1rem;
  color: var(--text-secondary);
  line-height: 1.7;
  margin: 0 0 2rem;
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
}

.hero-actions {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  flex-wrap: wrap;
  margin-bottom: 2.5rem;
}

.hero-stats {
  display: inline-flex;
  align-items: center;
  gap: 1.25rem;
  color: var(--text-muted);
  font-size: 0.85rem;
  font-weight: 500;
}

.stat-divider {
  width: 1px;
  height: 16px;
  background: var(--border-color);
}

// ============================================
// Buttons
// ============================================

.btn {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.625rem 1.5rem;
  border-radius: 8px;
  font-weight: 600;
  font-size: 0.875rem;
  text-decoration: none;
  cursor: pointer;
  border: none;
  transition: all 0.2s;
}

.btn-primary {
  background: var(--primary-color);
  color: var(--text-inverse);

  &:hover {
    opacity: 0.9;
    transform: translateY(-1px);
  }
}

.btn-outline {
  background: transparent;
  color: var(--text-primary);
  border: 1px solid var(--border-color);

  &:hover {
    border-color: var(--primary-color);
    color: var(--primary-color);
  }
}

.btn-text {
  background: none;
  border: none;
  color: var(--primary-color);
  font-weight: 600;
  font-size: 0.875rem;
  cursor: pointer;
  padding: 0.5rem 0.75rem;
  border-radius: 6px;
  transition: background-color 0.2s;

  &:hover:not(:disabled) {
    background: var(--background-secondary);
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}

// ============================================
// Sections
// ============================================

.section {
  padding: 5rem 0;
}

.section-alt {
  background: var(--background-secondary);
}

.section-header {
  text-align: center;
  margin-bottom: 3rem;
}

.section-title {
  font-size: 1.75rem;
  font-weight: 700;
  margin: 0 0 0.5rem;
  color: var(--text-primary);
}

.section-subtitle {
  color: var(--text-secondary);
  font-size: 1rem;
  margin: 0 0 0.75rem;
}

.section-line {
  width: 48px;
  height: 3px;
  background: var(--primary-color);
  margin: 0 auto;
  border-radius: 999px;
}

// ============================================
// Features Grid
// ============================================

.features-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1.5rem;

  @media (max-width: 1024px) {
    grid-template-columns: repeat(2, 1fr);
  }

  @media (max-width: 640px) {
    grid-template-columns: 1fr;
  }
}

.feature-card {
  background: var(--background-secondary);
  border: 1px solid var(--border-color);
  border-radius: 12px;
  padding: 1.75rem;
  transition: border-color 0.2s, box-shadow 0.2s;

  &:hover {
    border-color: var(--primary-color);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
  }

  [data-theme="chocolate"] & {
    background: var(--background-tertiary);
  }
}

.feature-number {
  display: inline-block;
  font-size: 0.75rem;
  font-weight: 700;
  color: var(--primary-color);
  margin-bottom: 0.75rem;
  letter-spacing: 0.05em;
}

.feature-title {
  font-size: 1.125rem;
  font-weight: 700;
  margin: 0 0 0.5rem;
  color: var(--text-primary);
}

.feature-desc {
  font-size: 0.875rem;
  color: var(--text-secondary);
  margin: 0;
  line-height: 1.6;
}

// ============================================
// Download
// ============================================

.version-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: var(--background-primary);
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 1rem 1.25rem;
  margin-bottom: 2rem;

  [data-theme="chocolate"] & {
    background: var(--background-tertiary);
  }
}

.version-badge {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.version-dot {
  width: 8px;
  height: 8px;
  background: var(--primary-color);
  border-radius: 50%;
}

.version-label {
  color: var(--text-secondary);
  font-size: 0.875rem;
}

.version-number {
  background: var(--primary-color);
  color: var(--text-inverse);
  padding: 0.2rem 0.6rem;
  border-radius: 6px;
  font-weight: 700;
  font-size: 0.8rem;
}

.download-content {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  align-items: start;

  @media (max-width: 768px) {
    grid-template-columns: 1fr;
  }
}

.download-info {
  h3 {
    margin: 0 0 1rem;
    font-weight: 700;
    color: var(--text-primary);
  }
}

.requirements-list {
  list-style: none;
  padding: 0;
  margin: 0;

  li {
    padding: 0.6rem 0;
    border-bottom: 1px solid var(--border-color);
    color: var(--text-secondary);
    font-size: 0.875rem;
    display: flex;
    align-items: flex-start;
    gap: 0.5rem;

    &::before {
      content: '\2713';
      color: var(--primary-color);
      font-weight: 700;
      flex-shrink: 0;
    }

    a {
      color: var(--primary-color);
      &:hover { text-decoration: underline; }
    }
  }
}

.download-actions {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.download-btn {
  display: flex;
  align-items: center;
  padding: 1rem 1.25rem;
  background: var(--background-primary);
  border: 1px solid var(--border-color);
  border-radius: 10px;
  text-decoration: none;
  color: var(--text-primary);
  transition: border-color 0.2s, transform 0.2s;

  [data-theme="chocolate"] & {
    background: var(--background-tertiary);
  }

  &:hover {
    border-color: var(--primary-color);
    transform: translateX(4px);

    .download-arrow {
      transform: translateX(4px);
    }
  }

  &.primary {
    background: var(--primary-color);
    color: var(--text-inverse);
    border-color: transparent;
  }

  .download-text {
    flex: 1;
    display: flex;
    flex-direction: column;

    strong { font-size: 0.95rem; }
    small {
      opacity: 0.7;
      font-size: 0.8rem;
      margin-top: 2px;
    }
  }

  .download-arrow {
    font-size: 1.1rem;
    transition: transform 0.2s;
  }
}

.prerelease-alert {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  background: var(--primary-color);
  border-radius: 10px;
  padding: 1.25rem;
  margin-top: 2rem;
  color: var(--text-inverse);

  .alert-content {
    flex: 1;

    h4 {
      margin: 0 0 0.25rem;
      font-weight: 700;
      color: var(--text-inverse);
    }
    p {
      margin: 0;
      opacity: 0.9;
      font-size: 0.875rem;
    }
  }

  .btn-outline {
    color: var(--text-inverse);
    border-color: rgba(255, 255, 255, 0.4);

    &:hover {
      background: rgba(255, 255, 255, 0.15);
      border-color: rgba(255, 255, 255, 0.6);
      color: var(--text-inverse);
    }
  }

  @media (max-width: 640px) {
    flex-direction: column;
    text-align: center;
  }
}

// ============================================
// Clients
// ============================================

.clients-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1rem;
}

.client-card {
  background: var(--background-secondary);
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 1.25rem;
  display: flex;
  align-items: center;
  gap: 1rem;
  transition: border-color 0.2s;

  [data-theme="chocolate"] & {
    background: var(--background-tertiary);
  }

  &:hover {
    border-color: var(--primary-color);
  }

  .client-info { flex: 1; }

  .client-name {
    margin: 0 0 0.25rem;
    font-size: 1rem;
    font-weight: 600;
    color: var(--text-primary);
  }

  .client-platform {
    margin: 0;
    color: var(--text-muted);
    font-size: 0.8rem;
  }

  .client-link {
    color: var(--primary-color);
    text-decoration: none;
    font-weight: 600;
    font-size: 0.85rem;
    padding: 0.4rem 0.75rem;
    border: 1px solid var(--primary-color);
    border-radius: 6px;
    transition: all 0.2s;
    white-space: nowrap;

    &:hover {
      background: var(--primary-color);
      color: var(--text-inverse);
    }
  }
}

// ============================================
// Star History
// ============================================

.stats-actions {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  flex-wrap: wrap;
}

.star-history-container {
  background: var(--background-primary);
  border: 1px solid var(--border-color);
  border-radius: 12px;
  padding: 2rem;
  margin-bottom: 2rem;
  min-height: 300px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;

  [data-theme="chocolate"] & {
    background: var(--background-tertiary);
  }
}

.star-history-chart {
  width: 100%;
  max-width: 800px;
  height: auto;
  border-radius: 8px;
}

.loading-state,
.error-state {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  gap: 1rem;
  padding: 2rem;
  color: var(--text-secondary);
}

.loading-spinner {
  width: 36px;
  height: 36px;
  border: 3px solid var(--border-color);
  border-top-color: var(--primary-color);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

// ============================================
// Docs
// ============================================

.docs-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1rem;
}

.doc-card {
  background: var(--background-secondary);
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 1.5rem;
  text-decoration: none;
  transition: border-color 0.2s;

  [data-theme="chocolate"] & {
    background: var(--background-tertiary);
  }

  &:hover {
    border-color: var(--primary-color);
  }

  h3 {
    margin: 0 0 0.5rem;
    font-weight: 700;
    font-size: 1rem;
    color: var(--text-primary);
  }

  p {
    margin: 0;
    color: var(--text-secondary);
    font-size: 0.85rem;
  }
}

// ============================================
// Footer
// ============================================

.footer {
  background: var(--background-dark);
  color: var(--text-inverse);
  padding: 3rem 0 1.5rem;
}

.footer-content {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 2rem;
  margin-bottom: 2rem;
}

.footer-section {
  h4 {
    margin: 0 0 1rem;
    font-weight: 600;
    color: var(--text-inverse);
    font-size: 0.95rem;
  }

  p {
    margin: 0;
    color: var(--text-muted);
    font-size: 0.875rem;
  }

  ul {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  li { margin-bottom: 0.5rem; }

  a {
    color: var(--text-muted);
    text-decoration: none;
    font-size: 0.875rem;
    transition: color 0.2s;

    &:hover { color: var(--primary-color); }
  }
}

.footer-logo-text {
  color: var(--primary-color) !important;
}

.footer-bottom {
  border-top: 1px solid var(--border-dark);
  padding-top: 1rem;
  text-align: center;

  p {
    margin: 0;
    color: var(--text-muted);
    font-size: 0.8rem;
  }
}

.beian-row {
  margin-top: 0.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.beian-item {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  color: var(--text-muted);
  font-size: 0.8rem;
  text-decoration: none;

  &:hover { color: var(--primary-color); }
}

.beian-badge {
  width: 16px;
  height: 16px;
}

// ============================================
// Responsive
// ============================================

@media (max-width: 768px) {
  .nav {
    flex-wrap: wrap;
    gap: 0.75rem;
  }

  .nav-center {
    order: 3;
    width: 100%;
    justify-content: center;
    flex-wrap: wrap;
    gap: 2px;
  }

  .nav-link {
    font-size: 0.75rem;
    padding: 0.375rem 0.5rem;
  }

  .logo-badge {
    display: none;
  }

  .hero {
    padding: 4rem 0 3rem;
  }

  .hero-actions {
    flex-direction: column;
    align-items: center;

    .btn {
      width: 100%;
      max-width: 260px;
      justify-content: center;
    }
  }

  .hero-stats {
    flex-direction: column;
    gap: 0.5rem;

    .stat-divider { display: none; }
  }

  .section {
    padding: 3rem 0;
  }

  .section-title {
    font-size: 1.5rem;
  }

  .version-info {
    flex-direction: column;
    gap: 0.75rem;
    text-align: center;
  }

  .prerelease-alert {
    flex-direction: column;
    text-align: center;
  }
}
</style>
