<template>
  <div id="product">
    <SfBreadcrumbs
      class="breadcrumbs desktop-only"
      :breadcrumbs="breadcrumbs"
    />
    <div class="product">
      <LazyHydrate when-idle>
        <SfGallery
          v-if="productGallery"
          :images="productGallery"
          imageWidth="550"
          imageHeight="412"
          thumbWidth="260"
          thumbHeight="260"
          class="product__gallery"
        />
      </LazyHydrate>

      <div class="product__info" v-if="product">
        <div class="product__header">
          <SfHeading
            :title="productGetters.getName(product)"
            :level="3"
            class="sf-heading--no-underline sf-heading--left"
          />
          <SfIcon
            icon="drag"
            size="xxl"
            color="var(--c-text-disabled)"
            class="product__drag-icon smartphone-only"
          />
        </div>
        <div class="product__price-and-rating">
          <SfPrice
            :regular="$n(productGetters.getPrice(product).regular, 'currency')"
            :special="productGetters.getPrice(product).special && $n(productGetters.getPrice(product).special, 'currency')"
          />
          <div>
            <div class="product__rating">
              <SfRating
                :score="averageRating"
                :max="5"
              />
              <a v-if="!!totalReviews" href="#" class="product__count">
                ({{ totalReviews }})
              </a>
            </div>
          </div>
        </div>
        <div>
          <p class="product__description desktop-only">
            {{ product.shortDescription }}
          </p>
          <SfSelect
            v-for="(item, key) in Object.keys(options)"
            :key="key"
            :value="configuration[item]"
            @input="value => updateFilter({ value, filter: item })"
            :label="options[item].label"
            class="sf-select--underlined product__select-size"
            :required="true"
          >
            <SfSelectOption
              v-for="size in options[item].value"
              :key="size.value"
              :value="size.value"
            >
              {{size.label}}
            </SfSelectOption>
          </SfSelect>

          <div v-if="options.color && options.color.length > 1" class="product__colors desktop-only">
            <p class="product__color-label">{{ $t('Color') }}:</p>
            <SfColor
              v-for="(color, i) in options.color"
              :key="i"
              :color="color.value"
              class="product__color"
              @click="updateFilter({color})"
            />
          </div>
          <SfAddToCart
            v-e2e="'product_add-to-cart'"
            :stock="product.selectedVariant.onHand"
            v-model="qty"
            :disabled="loading || !product.selectedVariant.inStock"
            class="product__add-to-cart"
            @click="handleAddToCart({ product, quantity: parseInt(qty) })"
          />
        </div>

        <LazyHydrate when-idle>
          <SfTabs :open-tab="1" class="product__tabs">
            <SfTab :title="$t('Description')" key="description">
              <div class="product__description">
                  {{ product.description }}
              </div>
              <SfProperty
                v-for="(property, i) in properties"
                :key="i"
                :name="property.name"
                :value="property.value"
                class="product__property"
              >
                <template v-if="property.name === 'Category'" #value>
                  <SfButton class="product__property__button sf-button--text">
                    {{ property.value }}
                  </SfButton>
                </template>
              </SfProperty>
            </SfTab>
            <SfTab
              v-if="Array.isArray(reviews) && reviews.length"
              :title="$t('Read reviews')"
              key="read_reviews"
            >
              <SfReview
                v-for="review in reviews"
                :key="reviewGetters.getReviewId(review)"
                :author="reviewGetters.getReviewAuthor(review)"
                :date="reviewGetters.getReviewDate(review)"
                :message="reviewGetters.getReviewMessage(review)"
                :max-rating="5"
                :rating="reviewGetters.getReviewRating(review)"
                :char-limit="250"
                read-more-text="Read more"
                hide-full-text="Read less"
                class="product__review"
              />
            </SfTab>
            <SfTab
              v-if="isAuthenticated"
              :title="$t('Add review')"
              key="add_review"
            >
              <add-review-form
                :product-id="product.id"
                @submit="handleReviewSubmit"
              />
            </SfTab>
          </SfTabs>
        </LazyHydrate>
      </div>
    </div>

    <LazyHydrate when-visible>
      <InstagramFeed />
    </LazyHydrate>

    <LazyHydrate when-visible>
      <MobileStoreBanner />
    </LazyHydrate>

  </div>
</template>
<script>
import {
  SfProperty,
  SfHeading,
  SfPrice,
  SfRating,
  SfSelect,
  SfAddToCart,
  SfTabs,
  SfGallery,
  SfIcon,
  SfImage,
  SfBanner,
  SfAlert,
  SfSticky,
  SfReview,
  SfBreadcrumbs,
  SfButton,
  SfColor
} from '@storefront-ui/vue';

import InstagramFeed from '~/components/InstagramFeed.vue';
import AddReviewForm from '~/components/Product/AddReviewForm.vue';
import { ref, computed, onUpdated } from '@nuxtjs/composition-api';
import { useProduct, useCart, productGetters, useReview, reviewGetters, useUser } from '@vue-storefront/sylius';
import { onSSR } from '@vue-storefront/core';
import MobileStoreBanner from '~/components/MobileStoreBanner.vue';
import LazyHydrate from 'vue-lazy-hydration';
import { useUiNotification } from '~/composables';

export default {
  name: 'Product',
  transition: 'fade',
  setup(props, context) {
    const t = (key) => context.root.$i18n.t(key);
    const qty = ref(1);
    const { id, slug } = context.root.$route.params;
    const { isAuthenticated } = useUser();
    const { products, search } = useProduct('products');
    const { send } = useUiNotification();

    const { addItem, loading, error } = useCart();
    const { reviews: productReviews, search: searchReviews, addReview } = useReview('productReviews');

    onSSR(async () => {
      await search({ slug, query: context.root.$route.query});
      await searchReviews({ productId: id });
    });
    const product = computed(() => products.value.products && productGetters.getFiltered(products.value.products, { master: true, attributes: context.root.$route.query })[0]);

    const options = computed(() => productGetters.getAttributes(products.value?.products, ['color', 'size'])) || [];

    const configuration = computed(() => product?.value ? productGetters.getAttributes(product?.value, ['color', 'size']) : []);
    const categories = computed(() => productGetters.getCategoryIds(product?.value)) || [];

    const reviews = computed(() => {
      return productReviews?.value ? reviewGetters.getItems(productReviews?.value) : [];
    });
    const totalReviewsCount = computed(() => {
      return productReviews?.value ? reviewGetters.getTotalReviews(productReviews.value) : 0;
    });

    // TODO: Breadcrumbs are temporary disabled because productGetters return undefined. We have a mocks in data
    // const breadcrumbs = computed(() => productGetters.getBreadcrumbs ? productGetters.getBreadcrumbs(product.value) : props.fallbackBreadcrumbs);

    const productGallery = computed(() => product?.value && productGetters.getGallery(product?.value).map(img => ({
      mobile: { url: img.small },
      desktop: { url: img.normal },
      big: { url: img.small },
      alt: product?.value?.name
    })));

    const handleAddToCart = async (params) => {
      await addItem(params);

      const cartError = Object.values(error.value).find(err => err !== null);

      if (cartError) {
        send({ type: 'danger', message: cartError.message });

        return;
      }

      send({ type: 'success', message: t('Product has been added to the cart') });
    };

    const updateFilter = (item) => {
      if (Array.isArray(item)) {
        const filterObj = item.reduce((prev, curr) => {
          const record = {};
          record[curr.filter] = curr.value;

          return {
            ...prev,
            ...record
          };
        }, {});

        context.root.$router.replace({
          path: context.root.$route.path,
          query: {
            ...configuration.value,
            ...filterObj
          }
        });

        return;
      }

      const filterObj = {};
      filterObj[item.filter] = item.value;

      context.root.$router.replace({
        path: context.root.$route.path,
        query: {
          ...configuration.value,
          ...filterObj
        }
      });
    };
    const handleReviewSubmit = async ({form, onComplete, onError}) => {
      try {
        form.value.productId = parseInt(id);
        await addReview(form.value);
        onComplete();
      } catch (e) {
        onError(e);
      }
    };

    const properties = computed(() => {
      return product.value?.attributes?.map(item => ({
        value: item.stringValue,
        name: item.name
      })) || [];
    });

    onUpdated(() => {
      if (
        !Object.keys(context.root.$route.query).length &&
        Object.keys(options.value).length
      ) {
        const filter = Object
          .keys(options.value)
          .map(key => ({ value: options.value[key].value[0].value, filter: key }));

        updateFilter(filter);
      }
    });

    return {
      updateFilter,
      configuration,
      product,
      categories,
      properties,
      reviews,
      reviewGetters,
      price: computed(() => productGetters.getPrice(product.value)),
      averageRating: computed(() => productGetters.getAverageRating(product.value)),
      totalReviews: totalReviewsCount,
      options,
      qty,
      handleAddToCart,
      loading,
      productGetters,
      productGallery,
      isAuthenticated,
      handleReviewSubmit
    };
  },
  components: {
    SfAlert,
    SfColor,
    SfProperty,
    SfHeading,
    SfPrice,
    SfRating,
    SfSelect,
    SfAddToCart,
    SfTabs,
    SfGallery,
    SfIcon,
    SfImage,
    SfBanner,
    SfSticky,
    SfReview,
    SfBreadcrumbs,
    SfButton,
    InstagramFeed,
    MobileStoreBanner,
    LazyHydrate,
    AddReviewForm
  },
  data() {
    return {
      detailsIsActive: false,
      breadcrumbs: []
    };
  }
};
</script>

<style lang="scss" scoped>
#product {
  box-sizing: border-box;
  @include for-desktop {
    max-width: 1272px;
    margin: 0 auto;
  }
}
.product {
  @include for-desktop {
    display: flex;
    padding: 0 var(--spacer-sm);
  }
  &__info {
    margin: var(--spacer-sm) auto;
    flex-grow: 1;
    @include for-desktop {
      max-width: 32.625rem;
      margin: 0 auto;
      padding-left: var(--spacer-sm);
    }
  }
  &__header {
    --heading-title-color: var(--c-link);
    --heading-title-font-weight: var(--font-weight--bold);
    --heading-padding: 0;
    margin: 0 var(--spacer-sm);
    display: flex;
    justify-content: space-between;
    @include for-desktop {
      --heading-title-font-weight: var(--font-weight--semibold);
      margin: 0 auto;
    }
  }
  &__drag-icon {
    animation: moveicon 1s ease-in-out infinite;
  }
  &__price-and-rating {
    margin: 0 var(--spacer-sm) var(--spacer-base);
    align-items: center;
    @include for-desktop {
      display: flex;
      justify-content: space-between;
      margin: var(--spacer-sm) 0 var(--spacer-lg) 0;
    }
  }
  &__rating {
    display: flex;
    align-items: center;
    justify-content: flex-end;
    margin: var(--spacer-xs) 0 var(--spacer-xs);
  }
  &__count {
    @include font(
      --count-font,
      var(--font-weight--normal),
      var(--font-size--sm),
      1.4,
      var(--font-family--secondary)
    );
    color: var(--c-text);
    text-decoration: none;
    margin: 0 0 0 var(--spacer-xs);
  }
  &__description {
    @include font(
      --product-description-font,
      var(--font-weight--light),
      var(--font-size--base),
      1.6,
      var(--font-family--primary)
    );
  }
  &__select-size {
    margin: 0 var(--spacer-sm);
    @include for-desktop {
      margin: 0;
    }
  }
  &__colors {
    @include font(
      --product-color-font,
      var(--font-weight--normal),
      var(--font-size--lg),
      1.6,
      var(--font-family--secondary)
    );
    display: flex;
    align-items: center;
    margin-top: var(--spacer-xl);
  }
  &__color-label {
    margin: 0 var(--spacer-lg) 0 0;
  }
  &__color {
    margin: 0 var(--spacer-2xs);
  }
  &__add-to-cart {
    margin: var(--spacer-base) var(--spacer-sm) 0;
    @include for-desktop {
      margin-top: var(--spacer-2xl);
    }
  }
  &__guide,
  &__compare,
  &__save {
    display: block;
    margin: var(--spacer-xl) 0 var(--spacer-base) auto;
  }
  &__compare {
    margin-top: 0;
  }
  &__tabs {
    --tabs-title-z-index: 0;
    --tabs-title-margin: 0 var(--spacer-sm) 0 0;
    margin: var(--spacer-lg) auto var(--spacer-2xl);
    --tabs-title-font-size: var(--font-size--lg);
    @include for-desktop {
      margin-top: var(--spacer-2xl);
    }
  }
  &__property {
    margin: var(--spacer-base) 0;
    &__button {
      --button-font-size: var(--font-size--base);
    }
  }
  &__review {
    padding-bottom: 24px;
    border-bottom: var(--c-light) solid 1px;
    margin-bottom: var(--spacer-base);

    @include for-mobile {
      max-width: 100%;
    }
  }
  &__additional-info {
    color: var(--c-link);
    @include font(
      --additional-info-font,
      var(--font-weight--light),
      var(--font-size--sm),
      1.6,
      var(--font-family--primary)
    );
    &__title {
      font-weight: var(--font-weight--normal);
      font-size: var(--font-size--base);
      margin: 0 0 var(--spacer-sm);
      &:not(:first-child) {
        margin-top: 3.5rem;
      }
    }
    &__paragraph {
      margin: 0;
    }
  }
  ::v-deep .sf-gallery {
    flex: 1;

    @include for-mobile {
      &__stage {
        max-width: 100%;
      }
      &__big-image {
        width: 100%;
      }
      .sf-image {
        width: 100%;
        height: auto;
      }
    }
  }
}
.breadcrumbs {
  margin: var(--spacer-base) auto var(--spacer-lg);
}
@keyframes moveicon {
  0% {
    transform: translate3d(0, 0, 0);
  }
  50% {
    transform: translate3d(0, 30%, 0);
  }
  100% {
    transform: translate3d(0, 0, 0);
  }
}
</style>
