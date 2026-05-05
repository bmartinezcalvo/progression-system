<script setup>
import { CdxButton, CdxIcon, CdxTextInput } from '@wikimedia/codex';
import {
  cdxIconAppearance,
  cdxIconArrowNext,
  cdxIconBell,
  cdxIconHistory,
  cdxIconLogoMediaWiki,
  cdxIconLogoWikimedia,
  cdxIconMenu,
  cdxIconSearch,
  cdxIconTray,
  cdxIconWatchlist
} from '@wikimedia/codex-icons';

const props = defineProps( {
  showFooter: {
    type: Boolean,
    default: true
  }
} );

const emit = defineEmits( [ 'go-home', 'go-article' ] );

const WIKIPEDIA_WORDMARK_EN =
  'https://en.wikipedia.org/static/images/mobile/copyright/wikipedia-wordmark-en-25.svg';
const WIKIPEDIA_TAGLINE_EN =
  'https://en.wikipedia.org/static/images/mobile/copyright/wikipedia-tagline-en-25.svg';

const desktopLinks = [
  'Privacy policy',
  'About Wikipedia',
  'Disclaimers',
  'Contact Wikipedia',
  'Legal & safety contacts',
  'Code of Conduct',
  'Developers',
  'Statistics',
  'Cookie statement',
  'Mobile view'
];

const mobileLinks = [
  'Privacy policy',
  'Contact Wikipedia',
  'Legal & safety contacts',
  'Code of Conduct',
  'Developers',
  'Statistics',
  'Cookie statement',
  'Terms of Use',
  'Desktop view'
];
</script>

<template>
  <div class="chrome-shell">
    <header class="chrome-header chrome-header--desktop">
      <nav class="chrome-header__nav chrome-header__nav--desktop" aria-label="Site">
        <span class="chrome-header__menu-icon" aria-hidden="true">
          <CdxIcon :icon="cdxIconMenu" />
        </span>

        <a class="chrome-header__brand chrome-header__brand--desktop" href="/" aria-label="Visit the main page">
          <span class="chrome-header__wordmarks">
            <img
              class="chrome-header__wordmark-img"
              :src="WIKIPEDIA_WORDMARK_EN"
              width="120"
              height="18"
              alt="Wikipedia"
            >
            <img
              class="chrome-header__tagline-img"
              :src="WIKIPEDIA_TAGLINE_EN"
              width="120"
              height="14"
              alt=""
            >
          </span>
        </a>

        <div class="chrome-header__search-cluster">
          <CdxTextInput
            class="chrome-header__search-input"
            placeholder="Search Wikipedia"
            aria-label="Search Wikipedia"
          >
            <template #start-icon>
              <CdxIcon :icon="cdxIconSearch" />
            </template>
          </CdxTextInput>
          <CdxButton class="chrome-header__search-submit" @click="emit( 'go-article' )">
            Search
          </CdxButton>
        </div>

        <div class="chrome-header__desktop-actions">
          <CdxButton
            class="chrome-header__search-toggle"
            weight="quiet"
            aria-label="Search"
            @click="emit( 'go-article' )"
          >
            <CdxIcon :icon="cdxIconSearch" />
          </CdxButton>
          <a class="chrome-header__username-link" href="https://meta.wikimedia.org/wiki/Main_Page">
            Username
          </a>
          <CdxButton weight="quiet" aria-label="Appearance">
            <CdxIcon :icon="cdxIconAppearance" />
          </CdxButton>
          <CdxButton class="chrome-header__notify-button" weight="quiet" aria-label="Notifications">
            <CdxIcon :icon="cdxIconBell" />
            <span class="chrome-header__notify-badge" aria-hidden="true">1</span>
          </CdxButton>
          <CdxButton weight="quiet" aria-label="Notices">
            <CdxIcon :icon="cdxIconTray" />
          </CdxButton>
          <CdxButton class="chrome-header__watchlist-button" weight="quiet" aria-label="Watchlist">
            <CdxIcon :icon="cdxIconWatchlist" />
          </CdxButton>
          <CdxButton
            class="chrome-header__user-button"
            weight="quiet"
            aria-label="Homepage"
            @click="emit( 'go-home' )"
          >
            <span class="chrome-header__user-avatar">Ca</span>
          </CdxButton>
        </div>
      </nav>
    </header>

    <header class="chrome-header chrome-header--mobile">
      <nav class="chrome-header__nav chrome-header__nav--mobile" aria-label="Site">
        <CdxButton weight="quiet" size="large" aria-label="Main menu">
          <CdxIcon :icon="cdxIconMenu" />
        </CdxButton>

        <a class="chrome-header__brand chrome-header__brand--mobile" href="/" aria-label="Visit the main page">
          <img
            class="chrome-header__mobile-wordmark-img"
            :src="WIKIPEDIA_WORDMARK_EN"
            alt="Wikipedia"
          >
        </a>

        <div class="chrome-header__mobile-actions">
          <CdxButton weight="quiet" size="large" aria-label="Search" @click="emit( 'go-article' )">
            <CdxIcon :icon="cdxIconSearch" />
          </CdxButton>
          <CdxButton class="chrome-header__notify-button" weight="quiet" size="large" aria-label="Notifications">
            <CdxIcon :icon="cdxIconBell" />
            <span class="chrome-header__notify-badge" aria-hidden="true">1</span>
          </CdxButton>
          <CdxButton
            class="chrome-header__user-button"
            weight="quiet"
            size="large"
            aria-label="Homepage"
            @click="emit( 'go-home' )"
          >
            <span class="chrome-header__user-avatar">Ca</span>
          </CdxButton>
        </div>
      </nav>
    </header>

    <main class="chrome-shell__content">
      <slot />
    </main>

    <footer v-if="props.showFooter" class="chrome-footer chrome-footer--desktop">
      <div class="chrome-footer__inner">
        <p class="chrome-footer__credit">This is a prototype made with ProtoWiki.</p>
        <ul class="chrome-footer__links">
          <li v-for="link in desktopLinks" :key="link">
            <a href="#">{{ link }}</a>
          </li>
        </ul>
      </div>
    </footer>

    <footer v-if="props.showFooter" class="chrome-footer chrome-footer--mobile">
      <a class="chrome-footer__last-edited" href="#">
        <CdxIcon :icon="cdxIconHistory" size="small" />
        <span class="chrome-footer__last-edited-text">
          Last edited 1 month ago by <strong>Username</strong>
        </span>
        <CdxIcon :icon="cdxIconArrowNext" size="small" />
      </a>

      <div class="chrome-footer__mobile-body">
        <div class="chrome-footer__brand-row">
          <img
            class="chrome-footer__mobile-wordmark"
            :src="WIKIPEDIA_WORDMARK_EN"
            width="120"
            height="18"
            alt="Wikipedia"
          >

          <div class="chrome-footer__badge-cluster">
            <a class="chrome-footer__badge-btn" href="#" aria-label="Wikimedia Foundation">
              <CdxIcon :icon="cdxIconLogoWikimedia" />
            </a>
            <a class="chrome-footer__badge-btn" href="#" aria-label="MediaWiki">
              <CdxIcon :icon="cdxIconLogoMediaWiki" />
            </a>
          </div>
        </div>

        <div class="chrome-footer__rule" aria-hidden="true"></div>

        <p class="chrome-footer__credit chrome-footer__credit--mobile">
          This is a prototype made with ProtoWiki.
        </p>

        <ul class="chrome-footer__links chrome-footer__links--mobile">
          <li v-for="link in mobileLinks" :key="link">
            <a href="#">{{ link }}</a>
          </li>
        </ul>
      </div>
    </footer>
  </div>
</template>

<style scoped>
.chrome-shell {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background-color: var(--background-color-base);
  font-size: 16px;
}

.chrome-header--desktop,
.chrome-footer--desktop {
  display: none;
}

.chrome-header--mobile,
.chrome-footer--mobile {
  display: block;
}

.chrome-header--mobile {
  border-bottom: 1px solid var(--border-color-subtle);
  background-color: var(--background-color-interactive);
  box-shadow: inset 0 -1px 3px rgb(0 0 0 / 0.08);
}

.chrome-header__nav--mobile {
  display: flex;
  align-items: center;
  gap: var(--spacing-50);
  min-height: 3.375rem;
  padding: 0 var(--spacing-100) 0 var(--spacing-50);
}

.chrome-header__brand {
  color: inherit;
  text-decoration: none;
}

.chrome-header__brand:hover {
  color: inherit;
  text-decoration: none;
}

.chrome-header__brand--mobile {
  min-width: 0;
}

.chrome-header__mobile-wordmark-img {
  display: block;
  width: auto;
  height: 21px;
  opacity: 0.67;
}

.chrome-header__mobile-actions {
  display: flex;
  align-items: center;
  gap: 0;
  margin-inline-start: auto;
}

.chrome-header__mobile-actions :deep(.cdx-button) {
  width: 44px;
  min-width: 44px;
  height: 44px;
  padding: 0;
}

.chrome-header__notify-button {
  position: relative;
}

.chrome-header__notify-badge {
  position: absolute;
  right: 0;
  bottom: 2px;
  min-width: 16px;
  padding: 0 3px;
  border: 1px solid var(--color-inverted-fixed);
  border-radius: var(--border-radius-base);
  background-color: var(--background-color-progressive);
  color: var(--color-inverted-fixed);
  font-size: var(--font-size-x-small);
  font-weight: var(--font-weight-bold);
  line-height: 1;
  text-align: center;
}

.chrome-shell__content {
  flex: 1 1 auto;
  min-height: 0;
}

.chrome-footer__credit {
  margin: 0;
}

.chrome-footer--mobile {
  margin-top: 0;
}

.chrome-footer__last-edited {
  display: flex;
  align-items: center;
  gap: var(--spacing-50);
  padding: var(--spacing-50) var(--spacing-100);
  border-block: 1px solid var(--border-color-muted, #dadde3);
  background-color: var(--background-color-neutral-subtle, #f8f9fa);
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 22px;
  text-decoration: none;
}

.chrome-footer__last-edited-text {
  flex: 1 1 auto;
  min-width: 0;
}

.chrome-footer__last-edited-text strong {
  color: var(--color-emphasized);
}

.chrome-footer__mobile-body {
  padding: var(--spacing-100);
  background-color: var(--background-color-neutral-subtle, #f8f9fa);
}

.chrome-footer__brand-row {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: var(--spacing-75);
}

.chrome-footer__mobile-wordmark {
  display: block;
  width: auto;
  height: 18px;
  opacity: 0.85;
}

.chrome-footer__badge-cluster {
  display: flex;
  align-items: center;
  gap: var(--spacing-50);
}

.chrome-footer__badge-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 44px;
  height: 44px;
  border: 1px solid var(--border-color-muted, #dadde3);
  border-radius: var(--border-radius-base);
  background-color: var(--background-color-base);
  color: var(--color-base);
}

.chrome-footer__rule {
  height: 1px;
  margin: var(--spacing-100) 0;
  background-color: var(--border-color-muted, #dadde3);
}

.chrome-footer__credit--mobile {
  margin-bottom: var(--spacing-75);
  font-size: 14px;
  line-height: 22px;
}

.chrome-footer__links {
  margin: 0;
  padding: 0;
  list-style: none;
  font-size: 14px;
  line-height: 22px;
}

.chrome-footer__links li {
  display: inline;
}

.chrome-footer__links a {
  color: var(--color-progressive);
  font-size: 14px;
  line-height: 22px;
  text-decoration: none;
}

.chrome-footer__links a:hover {
  text-decoration: underline;
}

.chrome-footer__links--mobile {
  font-size: var(--font-size-large);
  line-height: 1.7;
}

.chrome-footer__links--mobile li::after {
  content: ' · ';
  color: var(--color-subtle);
}

.chrome-footer__links--mobile li:last-child::after {
  content: '';
}

@media (min-width: 640px) {
  .chrome-header--desktop,
  .chrome-footer--desktop {
    display: block;
  }

  .chrome-header--mobile,
  .chrome-footer--mobile {
    display: none;
  }

  .chrome-shell {
    font-size: 14px;
  }

  .chrome-header--desktop {
    background-color: var(--background-color-base);
  }

  .chrome-header__nav--desktop {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: var(--spacing-100);
    min-height: 66px;
    padding: var(--spacing-50) var(--spacing-100);
  }

  .chrome-header__menu-icon {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    min-width: 32px;
    min-height: 32px;
    color: var(--color-base);
    line-height: 0;
  }

  .chrome-header__brand--desktop {
    display: flex;
    align-items: center;
    flex-shrink: 0;
  }

  .chrome-header__wordmarks {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    gap: 2px;
    width: 152px;
    min-height: 44px;
    margin-inline-start: var(--spacing-50);
    padding-inline-start: var(--spacing-75);
    padding-block: 3px;
  }

  .chrome-header__wordmark-img,
  .chrome-header__tagline-img {
    display: block;
    width: auto;
    max-width: 100%;
  }

  .chrome-header__search-cluster {
    display: flex;
    flex: 1 1 auto;
    align-items: stretch;
    max-width: 474px;
    padding-inline-start: var(--spacing-150);
  }

  .chrome-header__search-input {
    flex: 1 1 auto;
    min-width: 0;
  }

  .chrome-header__search-submit:deep(.cdx-button) {
    align-self: stretch;
    margin-inline-start: -1px;
    border-radius: 0 var(--border-radius-base) var(--border-radius-base) 0;
  }

  .chrome-header__desktop-actions {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: 0;
    margin-inline-start: auto;
  }

  .chrome-header__search-toggle {
    display: none;
  }

  .chrome-header__username-link {
    margin-inline: var(--spacing-50);
    color: var(--color-progressive);
    font-size: var(--font-size-medium);
    text-decoration: none;
  }

  .chrome-header__username-link:hover {
    text-decoration: underline;
  }

  .chrome-header__desktop-actions :deep(.cdx-button) {
    width: 44px;
    min-width: 44px;
    height: 44px;
    padding: 0;
  }

  .chrome-header__dropdown-chevron {
    display: inline-block;
    width: 20px;
    height: 20px;
    transform: scale(0.5);
    background-color: var(--color-base);
    mask-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 20 20" fill="%23000"><path d="m17.5 4.75-7.5 7.5-7.5-7.5L1 6.25l9 9 9-9z"/></svg>');
    mask-repeat: no-repeat;
    mask-position: center;
  }

  .chrome-header__user-button:deep(.cdx-button) {
    gap: 0;
    overflow: visible;
  }

  .chrome-header__user-avatar {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 32px;
    height: 32px;
    border: 1px solid var(--border-color-subtle);
    border-radius: 50%;
    background: #edf3ff;
    color: var(--color-emphasized);
    font-size: 16px;
    line-height: 22px;
    font-weight: var(--font-weight-regular);
  }

  .chrome-footer--desktop {
    margin-top: var(--spacing-200);
    margin-inline: var(--spacing-100);
    background-color: var(--background-color-base);
    color: var(--color-base);
    font-size: var(--font-size-x-small);
    line-height: var(--line-height-x-small);
  }

  .chrome-footer__inner {
    max-width: 99.75rem;
    margin: 0 auto;
    padding: var(--spacing-150) 0;
    border-top: 1px solid var(--border-color-subtle);
  }

  .chrome-footer--desktop .chrome-footer__credit {
    margin-bottom: var(--spacing-100);
    font-size: 14px;
    line-height: 22px;
  }

  .chrome-footer--desktop .chrome-footer__links li {
    display: inline-block;
    margin-right: var(--spacing-50);
  }

  .chrome-footer--desktop .chrome-footer__links li::after {
    content: '•';
    padding-inline-start: var(--spacing-50);
    color: var(--color-subtle);
  }

  .chrome-footer--desktop .chrome-footer__links li:last-child::after {
    content: none;
  }
}

@media (max-width: 1119px) and (min-width: 640px) {
  .chrome-header__search-cluster {
    display: none;
  }

  .chrome-header__search-toggle {
    display: inline-flex;
  }

  .chrome-header__desktop-actions :deep(.cdx-button:not(.chrome-header__user-button)) {
    width: 44px;
    min-width: 44px;
    height: 44px;
    padding: 0;
  }

  .chrome-header__user-button:deep(.cdx-button) {
    width: 44px;
    min-width: 44px;
    height: 44px;
    padding: 0;
  }

  .chrome-header__notify-badge {
    right: 4px;
    bottom: 8px;
  }
}

@media (max-width: 767px) and (min-width: 640px) {
  .chrome-header__watchlist-button {
    display: none;
  }
}
</style>
