<script setup>
import DefaultTheme from "vitepress/theme";
import TranslationNotice from "./TranslationNotice.vue";
</script>

<template>
  <DefaultTheme.Layout>
    <template #doc-top>
      <div class="doc-top-wrapper">
        <div class="doc-top-content">
          <TranslationNotice />
        </div>
        <div class="doc-top-aside-spacer"></div>
      </div>
    </template>
  </DefaultTheme.Layout>
</template>

<style>
/*
 * The #doc-top slot is a direct child of .VPDoc, OUTSIDE .container.
 * We replicate .container's flex layout (content + aside) to align
 * the notice with the actual content area below.
 */
.doc-top-wrapper {
  max-width: var(--vp-layout-max-width, 1480px);
  margin: 0 auto;
  display: flex;
}

.doc-top-content {
  flex: 1;
  min-width: 0;
  display: flex;
  justify-content: center;
}

.doc-top-content > * {
  width: 100%;
  max-width: 744px;
}

.doc-top-aside-spacer {
  width: 320px;
  flex-shrink: 0;
}

/* On pages without aside (no outline), don't reserve aside space */
.VPDoc:not(.has-aside) .doc-top-aside-spacer {
  display: none;
}

/* On mobile, aside is hidden */
@media (max-width: 1024px) {
  .doc-top-aside-spacer {
    display: none;
  }
}
</style>
