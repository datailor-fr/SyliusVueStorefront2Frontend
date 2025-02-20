<template>
  <div>
    <div class="highlighted">
      <SfHeading
        :level="3"
        :title="$t('Order summary')"
        class="sf-heading--left sf-heading--no-underline title"
      />
    </div>
    <div class="highlighted">
      <SfProperty
        :name="$t('Products')"
        :value="totalItems"
        class="sf-property--full-width sf-property--large property"
      />
      <SfProperty
        :name="$t('Subtotal')"
        :value="$n(totals.subtotal, 'currency')"
        :class="['sf-property--full-width', 'sf-property--large property', { discounted: hasSpecialPrice }]"
      />
      <SfProperty
        v-for="discount in discounts"
        :key="discount.id"
        :name="discount.name + (discount.code && ` (${discount.code})`)"
        :value="'-' + $n(discount.value, 'currency')"
        class="sf-property--full-width sf-property--large property"
      >
        <template #value="{ props }">
          <span class="sf-property__value" style="text-align: right;">
            {{ props.value }}
            <a
              href="#"
              @click.prevent="handleCouponRemoval(discount)"
              class="text-primary"
              style="font-size: 12px;font-weight: normal;"
            >Remove</a>
          </span>
        </template>
      </SfProperty>
     <SfProperty
        v-if="hasSpecialPrice"
        :value="$n(totals.special, 'currency')"
        class="sf-property--full-width sf-property--small property special-price"
      />
      <SfProperty
        v-if="hasShipping"
        :name="$t('Shipping')"
        :value="$n(totals.shipping, 'currency')"
        class="sf-property--full-width sf-property--large property"
      />
      <SfProperty
        :name="$t('Total')"
        :value="$n(totals.total, 'currency')"
        class="sf-property--full-width sf-property--large property-total"
      />
    </div>
    <div class="highlighted promo-code">
      <SfInput
        v-model="promoCode"
        name="promoCode"
        :label="$t('Enter promo code')"
        class="sf-input--filled promo-code__input"
      />
      <SfButton class="promo-code__button" @click="submitCouponForm">{{ $t('Apply') }}</SfButton>
    </div>
    <div class="highlighted">
      <SfCharacteristic
        v-for="characteristic in characteristics"
        :key="characteristic.title"
        :title="characteristic.title"
        :description="characteristic.description"
        :icon="characteristic.icon"
        class="characteristic"
      />
    </div>
  </div>
</template>
<script>
import {
  SfHeading,
  SfButton,
  SfCollectedProduct,
  SfProperty,
  SfCharacteristic,
  SfInput,
  SfCircleIcon
} from '@storefront-ui/vue';
import { onSSR } from '@vue-storefront/core';
import { computed, ref } from '@nuxtjs/composition-api';
import { useCart, cartGetters } from '@vue-storefront/sylius';
import { useUiNotification } from '~/composables/';

export default {
  name: 'CartPreview',
  components: {
    SfHeading,
    SfButton,
    SfCollectedProduct,
    SfProperty,
    SfCharacteristic,
    SfInput,
    SfCircleIcon
  },
  setup (_, context) {
    const { cart, removeItem, updateItemQty, applyCoupon, load, removeCoupon, error } = useCart();
    const { send } = useUiNotification();
    const t = (key) => context.root.$i18n.t(key);

    const listIsHidden = ref(false);
    const promoCode = ref('');
    const showPromoCode = ref(false);

    const products = computed(() => cartGetters.getItems(cart.value));
    const totalItems = computed(() => cartGetters.getTotalItems(cart.value));
    const totals = computed(() => cartGetters.getTotals(cart.value));
    const discounts = computed(() => cartGetters.getDiscounts(cart.value));

    const submitCouponForm = async () => {
      await applyCoupon({ couponCode: promoCode.value });
      const errorKeys = Object.keys(error.value);
      errorKeys.forEach(errorKey => {
        if (error.value[errorKey] && error.value[errorKey]?.message) {
          send({ type: 'danger', message: error.value[errorKey].message });
        }
      });
    };

    const handleCouponRemoval = async (coupon) => {
      await removeCoupon({
        couponCode: coupon.code
      });
    };

    onSSR(async () => {
      if (!cart.value) await load();
    });

    return {
      discounts,
      totalItems,
      listIsHidden,
      products,
      totals,
      promoCode,
      showPromoCode,
      removeItem,
      updateItemQty,
      cartGetters,
      applyCoupon,
      handleCouponRemoval,
      submitCouponForm,
      characteristics: [
        {
          title: t('Safety'),
          description: t('It carefully packaged with a personal touch'),
          icon: 'safety'
        },
        {
          title: t('Easy shipping'),
          description:
            t('You’ll receive dispatch confirmation and an arrival date'),
          icon: 'shipping'
        },
        {
          title: t('Changed your mind?'),
          description: t('Rest assured, we offer free returns within 30 days'),
          icon: 'return'
        }
      ],
      hasSpecialPrice: computed(() => totals.value.special > 0 && totals.value.special < totals.value.subtotal),
      hasShipping: computed(() => totals.value.shipping > 0)
    };
  }
};
</script>

<style lang="scss" scoped>
.highlighted {
  box-sizing: border-box;
  width: 100%;
  background-color: var(--c-light);
  padding: var(--spacer-xl) var(--spacer-xl) 0;
  &:last-child {
    padding-bottom: var(--spacer-xl);
  }
}
.total-items {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--spacer-xl);
}
.property {
  margin-bottom: var(--spacer-base);
}
.property-total {
  margin-top: var(--spacer-xl);
  padding-top: var(--spacer-lg);
  font-size: var(--font-size-xl);
  border-top: var(--c-white) 1px solid;
  --property-name-font-weight: var(--font-weight--semibold);
  --property-name-color: var(--c-text);
}

.characteristic {
  &:not(:last-child) {
    margin-bottom: var(--spacer-base);
  }
}
.promo-code {
  display: flex;
  align-items: flex-start;
  &__button {
    --button-width: 6.3125rem;
    --button-height: var(--spacer-lg);
  }
  &__input {
    --input-background: var(--c-white);
    flex: 1;
  }
}

.discounted {
  &::v-deep .sf-property__value {
    color: var(--c-danger);
    text-decoration: line-through;
  }
}

.special-price {
  justify-content: flex-end;

  &::v-deep .sf-property__name {
    display: none;
  }
}

</style>
