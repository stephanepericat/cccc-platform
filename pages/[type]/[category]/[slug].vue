<template>
  <div class="article-page" :class="{ pending }">
    <ILoader v-if="pending" class="article-page__loader" />
    <template v-else-if="!pending && article">
      <h1 v-text="article.title" class="article-page__title" />

      <IMedia class="article-page__author">
        <template #image>
          <SanityImage
            :asset-id="article.author.avatar"
            auto="format"
            :h="50"
            :w="50"
          />
        </template>
        <h5 class="article-page__author--name">
          <NuxtLink :text="article.author.nickname" :to="localePath(`/${AUTHOR}/${article.author.slug}`)" />
        </h5>
        <p class="article-page__info">
          <span class="article-page__info--category">{{ articleType }} / {{ article.category }} / {{ format(new Date(article.published), DEFAULT_DATE_FORMAT) }} ({{ $t("article.updated") }}: {{ format(new Date(article.updated), DEFAULT_DATE_FORMAT) }})</span>
        </p>
      </IMedia>

      <section class="article-page__body">
        <SanityContent :blocks="article.body" />
        <!-- community content -->
        <template v-if="type === COMMUNITY && article.info">
          <h4 class="article-page__body--info" v-text="$t('article.contactInfo')" />
          <div class="article-page__body--info-address">
            <div v-text="article.info.street1" />
            <div v-text="article.info.street2" />
            <div v-text="article.info.city" />
            <div v-text="article.info.zipCode" />
            <div v-text="article.info.country" />
          </div>
          <div class="article-page__body--info-contact">
            <IButton v-if="article.info.phone" class="article-page__body--info-contact-button" :to="`tel:${article.info.phone}`">
              <Icon class="article-page__body--info-contact-icon" name="material-symbols:add-call-rounded" />
              {{ $t("article.call") }}
            </IButton>
            <IButton v-if="article.info.email" class="article-page__body--info-contact-button" :to="`mailto:${article.info.email}`">
              <Icon class="article-page__body--info-contact-icon" name="material-symbols:alternate-email-rounded" />
              {{ $t("article.email") }}
            </IButton>
            <IButton v-if="article.info.website" class="article-page__body--info-contact-button" :to="article.info.website" target="_blank">
              <Icon class="article-page__body--info-contact-icon" name="material-symbols:web-sharp" />
              {{ $t("article.website") }}
            </IButton>
          </div>
        </template>
        <!-- product content -->
        <template v-if="type === PRODUCT">
          <div class="article-page__body--info-product">
            <IButton v-if="article.link" class="article-page__body--info-product-button" :to="article.link" target="_blank">
              <Icon class="article-page__body--info-product-icon" name="material-symbols:info-outline-rounded" />
              {{ $t("article.moreInfo") }}
            </IButton>
          </div>
        </template>
      </section>
    </template>
  </div>
</template>
<script setup>
  import { format } from "date-fns";
  import { AUTHOR, COMMUNITY, PRODUCT } from "~/assets/constants/types";
  import publicationQuery from "~/sanity/publication.sanity";
  import { DEFAULT_DATE_FORMAT } from "~/assets/constants/date-formats";

  const { locale, t } = useI18n();
  const localePath = useLocalePath();
  const { params } = useRoute();
  const { type, category, slug } = params;

  const { data: article, pending } = useLazySanityQuery(publicationQuery, {
    category,
    locale,
    slug,
    type,
  });

  const pageTitle = computed(() => t("article.title", { title: article?.value?.title || "" }));
  const articleType = computed(() => t(`layout.${type}`));

  useHead({
    meta: [
      { name: "description", content: t("home.description") },
    ],
    title: pageTitle.value
  });
</script>
<style lang="scss" scoped>
@import "~/assets/sass/mixins.scss";

.article-page {
  &.pending {
    @include pending();
  }

  &__loader {
    @include loader();
  }

  &__title {
    @include title();
    margin-bottom: 40px;
  }

  &__info--category,
  &__author--name {
    @include eyebrow();

    font-weight: 500;
  }

  &__author {
    margin-bottom: 40px;
  }

  &__info {
    &--category {
      font-weight: 200;
    }
  }

  &__body {
    font-size: 1.25rem;
    font-weight: 300;

    :deep(strong) {
      font-weight: 500;
    }

    &--info {
      @include eyebrow();

      font-weight: 400;
      margin-top: 40px;

      &-address {
        font-size: 1.1rem;
      }

      &-contact,
      &-product {
        margin-top: 20px;

        &-button {
          margin-right: 10px;
        }

        &-icon {
          margin-right: 5px;
        }
      }

      &-product {
        margin-top: 40px;
      }
    }
  }
}
</style>
