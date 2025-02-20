<template>
  <div>
    <SfMegaMenu
      :visible="isSearchOpen"
      :title="$t('Search results')"
      class="search"
    >
      <transition name="sf-fade" mode="out-in">
        <div v-if="products && products.length > 0" class="search__wrapper-results" key="results">
          <SfMegaMenuColumn :title="$t('Categories')" class="sf-mega-menu-column--pined-content-on-mobile search__categories">
            <template #title="{title}">
              <SfHeading :title="title" :level="4" class="search__header" />
            </template>
            <SfList>
              <SfListItem v-for="(category, key) in categories" :key="key">
                <SfMenuItem :label="category.name" :link="localePath(`/c/${category.slug}`)">
                  <template #mobile-nav-icon>
                    &#8203;
                  </template>
                </SfMenuItem>
              </SfListItem>
            </SfList>
          </SfMegaMenuColumn>
          <SfMegaMenuColumn :title="$t('Product suggestions')" class="sf-mega-menu-column--pined-content-on-mobile search__results">
            <template #title="{title}">
              <SfMenuItem :label="title" class="sf-mega-menu-column__header search__header">
                <template #mobile-nav-icon>
                  &#8203;
                </template>
              </SfMenuItem>
            </template>
            <SfScrollable class="results--desktop desktop-only" show-text="" hide-text="">
              <div class="results-listing">
                <SfProductCard
                  v-for="(product, index) in products"
                  :key="index"
                  class="result-card"
                  :regular-price="$n(productGetters.getPrice(product).regular, 'currency')"
                  :score-rating="productGetters.getAverageRating(product)"
                  :reviews-count="7"
                  wishlistIcon=""
                  isInWishlistIcon=""
                  :image="productGetters.getCoverImage(product)"
                  imageHeight="260"
                  imageWidth="260"
                  :alt="productGetters.getName(product)"
                  :title="productGetters.getName(product)"
                  :link="localePath(`/p/${productGetters.getId(product)}/${productGetters.getSlug(product)}`)"
                  :showAddToCartButton="true"
                  @click:add-to-cart="handleAddToCart({ product, quantity: 1 })"
                  :is-added-to-cart="isInCart({ product })"
                >
                  <template #image>
                    <NuxtLink :to="localePath(`/p/${productGetters.getId(product)}/${productGetters.getSlug(product)}`)">
                      <SfImage
                        class="sf-product-card__image"
                        :src="productGetters.getCoverImage(product)"
                        :alt="productGetters.getName(product)"
                        :width="260"
                        :height="260"
                        @click="$emit('close')"
                      />
                    </NuxtLink>
                  </template>
                </SfProductCard>
              </div>
            </SfScrollable>
            <div class="results--mobile smartphone-only">
              <SfProductCard
                v-for="(product, index) in products"
                :key="index"
                class="result-card"
                :regular-price="$n(productGetters.getPrice(product).regular, 'currency')"
                :score-rating="productGetters.getAverageRating(product)"
                :reviews-count="7"
                wishlistIcon=""
                isInWishlistIcon=""
                :image="productGetters.getCoverImage(product)"
                imageHeight="260"
                imageWidth="260"
                :alt="productGetters.getName(product)"
                :title="productGetters.getName(product)"
                :link="localePath(`/p/${productGetters.getId(product)}/${productGetters.getSlug(product)}`)"
                @click:add-to-cart="handleAddToCart({ product, quantity: 1 })"
                @click.native="$emit('close')"
                :is-added-to-cart="isInCart({ product })"
              />
            </div>
          </SfMegaMenuColumn>
          <div class="action-buttons smartphone-only">
            <SfButton class="action-buttons__button color-light" @click="$emit('close')">{{ $t('Cancel') }}</SfButton>
          </div>
        </div>
        <div v-else key="no-results" class="before-results">
          <SfImage src="/error/error.svg" height="240" width="240" class="before-results__picture" alt="error" loading="lazy"/>
          <p class="before-results__paragraph">{{ $t('You haven’t searched for items yet') }}</p>
          <p class="before-results__paragraph">{{ $t('Let’s start now – we’ll help you') }}</p>
          <SfButton class="before-results__button color-secondary smartphone-only" @click="$emit('close')">{{ $t('Go back') }}</SfButton>
        </div>
      </transition>
    </SfMegaMenu>
  </div>
</template>
<script>
import {
  SfMegaMenu,
  SfList,
  SfBanner,
  SfProductCard,
  SfScrollable,
  SfMenuItem,
  SfButton,
  SfHeading
} from '@storefront-ui/vue';
import SfImage from '~/components/SearchResults/SfImage.vue';
import { ref, watch, computed } from '@nuxtjs/composition-api';
import { productGetters, useCart } from '@vue-storefront/sylius';
import { useUiNotification } from '~/composables/';

export default {
  name: 'SearchResults',
  components: {
    SfMegaMenu,
    SfList,
    SfBanner,
    SfProductCard,
    SfScrollable,
    SfMenuItem,
    SfButton,
    SfImage,
    SfHeading
  },
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    result: {
      type: Object
    }
  },
  setup(props, { emit, root }) {
    const t = (key) => root.$i18n.t(key);
    const { addItem: addItemToCart, isInCart, error } = useCart();
    const { send } = useUiNotification();
    const isSearchOpen = ref(props.visible);
    const products = computed(() => props.result?.products);
    const categories = computed(() => props.result?.categories);

    const handleAddToCart = async (params) => {
      await addItemToCart(params);

      const cartError = Object.values(error.value).find(err => err !== null);

      if (cartError) {
        send({ type: 'danger', message: cartError.message });

        return;
      }

      send({ type: 'success', message: t('Product has been added to the cart') });
    };

    watch(() => props.visible, (newVal) => {
      isSearchOpen.value = newVal;
      if (isSearchOpen.value) {
        document.body.classList.add('no-scroll');
      } else {
        document.body.classList.remove('no-scroll');
        emit('removeSearchResults');
      }
    });

    return {
      isSearchOpen,
      productGetters,
      products,
      categories,
      handleAddToCart,
      isInCart
    };
  }
};
</script>
<style lang="scss" scoped>
.search {
  position: absolute;
  right: 0;
  left: 0;
  z-index: 3;
  --mega-menu-column-header-margin: var(--spacer-sm) 0 var(--spacer-xl);
  --mega-menu-content-padding: 0 0 var(--spacer-2xl) 0;
  --mega-menu-height: auto;
  @include for-desktop {
    --mega-menu-content-padding: var(--spacer-xl) 0;
  }
  &__wrapper-results {
    width: 100%;
    display: flex;
    flex-direction: column;
    @include for-desktop {
      flex-direction: row;
      flex: 1;
    }
  }
  &__categories {
    flex: 0 0 220px;
    margin: var(--spacer-sm);
    @include for-mobile {
      width: 95%;
      margin: auto;
      flex: 0 0 auto;
    }
  }
  &__results {
    flex: 1;
  }
  &__header {
    text-align: left;
  }
  ::v-deep .sf-bar {
    display: none;
  }
  ::v-deep .sf-mega-menu {
    &__menu {
      align-items: center;
    }
    &-column__header {
      display: none;
      @include for-desktop {
        display: flex;
      }
    }
  }
}
.results {
  &--desktop {
    --scrollable-max-height: 35vh;
    --product-card-add-button-bottom: var(--spacer-base);
  }
  &--mobile {
    padding: var(--spacer-base) var(--spacer-sm);
    display: grid;
    grid-gap: var(--spacer-sm);
    grid-template-columns: repeat(auto-fit, 10rem);
    justify-content: center;
    background: var(--c-white);
  }
}
.see-all {
  padding: var(--spacer-xl) 0 0 var(--spacer-sm);
}
.action-buttons {
  padding: var(--spacer-xl) var(--spacer-sm);
  background: var(--c-white);
  width: 100%;
  &__button {
    width: calc(100% - 32px);
  }
}
.results-listing {
  display: flex;
  flex-wrap: wrap;
  margin-left: var(--spacer-2xs);
}
.result-card {
  margin: var(--spacer-sm) 0;
  @include for-desktop {
    margin: var(--spacer-2xs) 0;
  }
}

.before-results {
  box-sizing: border-box;
  padding: var(--spacer-base) var(--spacer-sm) var(--spacer-2xl);
  width: 100%;
  text-align: center;
  @include for-desktop {
    padding: 0;
  }
  &__picture {
    --image-width: 230px;
    margin-top: var(--spacer-2xl);
    @include for-desktop {
      --image-width: 18.75rem;
      margin-top: var(--spacer-base);
    }
  }
  &__paragraph {
    font-family: var(--font-family--primary);
    font-weight: var(--font-weight--normal);
    font-size: var(--font-size--base);
    color: var(--c-text-muted);
    margin: 0;
    @include for-desktop {
      font-size: var(--font-size--lg);
    }
    &:first-of-type {
      margin: var(--spacer-xl) auto var(--spacer-xs);
    }
  }
  &__button {
    margin: var(--spacer-xl) auto;
    width: 100%;
  }
}
</style>
