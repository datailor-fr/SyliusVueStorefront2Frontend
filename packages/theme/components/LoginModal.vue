<template>
  <SfModal
    v-e2e="'login-modal'"
    :visible="isLoginModalOpen"
    :persistent="true"
    class="modal"
    @close="closeModal"
  >
    <template #modal-bar>
      <SfBar
        class="sf-modal__bar smartphone-only"
        :close="true"
        :title="$t(barTitle)"
        @click:close="closeModal"
      />
    </template>
    <transition name="sf-fade" mode="out-in">
      <div v-if="isLogin">
        <ValidationObserver v-slot="{ handleSubmit }" key="log-in">
          <form class="form" @submit.prevent="handleSubmit(handleLogin)">
            <div v-show="error.login" class="login-error">
              <SfAlert :message="error.login" type="danger"/>
            </div>
            <ValidationProvider rules="required|email" v-slot="{ errors }">
              <SfInput
                v-e2e="'login-modal-email'"
                v-model="form.username"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="email"
                :label="$t('Your e-mail')"
                class="form__element"
              />
            </ValidationProvider>
            <ValidationProvider rules="required" v-slot="{ errors }">
              <SfInput
                v-e2e="'login-modal-password'"
                v-model="form.password"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="password"
                :label="$t('Password')"
                type="password"
                class="form__element"
              />
            </ValidationProvider>
            <SfCheckbox
              v-e2e="'login-modal-remember-me'"
              v-model="rememberMe"
              name="remember-me"
              :label="$t('Remember me')"
              class="form__element checkbox remember-me-checkbox"
            />
            <SfButton v-e2e="'login-modal-submit'"
              type="submit"
              class="sf-button--full-width form__button"
              :disabled="loading"
            >
              <SfLoader :class="{ loader: loading }" :loading="loading">
                <div>{{ $t('Login') }}</div>
              </SfLoader>
            </SfButton>
          </form>
        </ValidationObserver>
<!--        <div class="action">-->
<!--          <SfButton class="sf-button&#45;&#45;text" @click="setIsForgottenValue(true)">-->
<!--            {{ $t('Forgotten password?') }}-->
<!--          </SfButton>-->
<!--        </div>-->
        <div class="bottom">
          <hr class="light-line login-register-separator" />
          <p class="bottom__paragraph">{{ $t('No account') }}</p>
          <SfButton class="register-today-button sf-button--full-width form__button color-light" @click="setIsLoginValue(false)">
            {{ $t('Register today') }}
          </SfButton>
        </div>
      </div>
      <div v-else-if="isForgotten">
        <p>{{ $t('Forgot Password') }}</p>
        <ValidationObserver v-slot="{ handleSubmit }" key="log-in">
          <form class="form" @submit.prevent="handleSubmit(handleForgotten)">
            <ValidationProvider rules="required|email" v-slot="{ errors }">
              <SfInput
                v-e2e="'forgot-modal-email'"
                v-model="form.username"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="email"
                :label="$t('Forgot Password Modal Email')"
                class="form__element"
              />
            </ValidationProvider>
            <div v-if="forgotPasswordError.request">
              {{ forgotPasswordError.request.message }}
            </div>
            <SfButton
              v-e2e="'forgot-modal-submit'"
              type="submit"
              class="sf-button--full-width form__button"
              :disabled="forgotPasswordLoading"
            >
              <SfLoader :class="{ loader: forgotPasswordLoading }" :loading="forgotPasswordLoading">
                <div>{{ $t('Reset Password') }}</div>
              </SfLoader>
            </SfButton>
          </form>
        </ValidationObserver>
      </div>
      <div v-else-if="isThankYouAfterForgotten" class="thank-you">
        <i18n tag="p" class="thank-you__paragraph" path="forgotPasswordConfirmation">
          <span class="thank-you__paragraph--bold">{{ userEmail }}</span>
        </i18n>
        <p class="thank-you__paragraph">{{ $t('Thank You Inbox') }}</p>
      </div>
      <div v-else-if="isVerifyUser" class="form">
          <p class="verify__paragraph">{{ $t('Please check your email to verify your account') }}</p>

          <SfButton type="submit" class="verify__button" @click="closeModal()">
            {{ $t('Go back') }}
          </SfButton>
      </div>
      <div v-else class="form">
        <ValidationObserver v-slot="{ handleSubmit }" key="sign-up">
          <form class="form" @submit.prevent="handleSubmit(handleRegister)" autocomplete="off">
            <ValidationProvider rules="required|email" v-slot="{ errors }">
              <SfInput
                v-e2e="'login-modal-email'"
                v-model="form.email"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="email"
                :label="$t('Your e-mail')"
                class="form__element"
              />
            </ValidationProvider>
            <ValidationProvider rules="required" v-slot="{ errors }">
              <SfInput
                v-e2e="'login-modal-firstName'"
                v-model="form.firstName"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="firstName"
                :label="$t('First name')"
                class="form__element"
              />
            </ValidationProvider>
            <ValidationProvider rules="required" v-slot="{ errors }">
              <SfInput
                v-e2e="'login-modal-lastName'"
                v-model="form.lastName"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="lastName"
                :label="$t('Last name')"
                class="form__element"
              />
            </ValidationProvider>
            <ValidationProvider rules="required" v-slot="{ errors }">
              <SfInput
                v-e2e="'login-modal-password'"
                v-model="form.password"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="password"
                :label="$t('Password')"
                type="password"
                class="form__element"
              />
            </ValidationProvider>
            <ValidationProvider :rules="{ required: { allowFalse: false } }" v-slot="{ errors }">
              <SfCheckbox
                v-e2e="'login-modal-create-account'"
                v-model="createAccount"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="createAccount"
                :label="$t('I accept the terms & conditions')"
                class="form__element"
              />
            </ValidationProvider>
            <div v-if="error.register">
              {{ error.register }}
            </div>
            <SfButton
              v-e2e="'login-modal-submit'"
              type="submit"
              class="sf-button--full-width form__button"
              :disabled="loading"
            >
              <SfLoader :class="{ loader: loading }" :loading="loading">
                <div>{{ $t('Create an account') }}</div>
              </SfLoader>
            </SfButton>
          </form>
        </ValidationObserver>
        <div class="action">
          {{ $t('or') }}
          <SfButton v-e2e="'login-modal-login-to-your-account'" class="sf-button--text" @click="setIsLoginValue(true)">
            {{ $t('login in to your account') }}
          </SfButton>
        </div>
      </div>
    </transition>
  </SfModal>
</template>
<script>
import { ref, watch, reactive, computed } from '@nuxtjs/composition-api';
import { SfModal, SfInput, SfButton, SfCheckbox, SfLoader, SfAlert, SfBar } from '@storefront-ui/vue';
import { ValidationProvider, ValidationObserver } from 'vee-validate';
import { useUser, useForgotPassword } from '@vue-storefront/sylius';
import { useUiState, useUiNotification } from '~/composables';
import { useVSFContext } from '@vue-storefront/core';

export default {
  name: 'LoginModal',
  components: {
    SfModal,
    SfInput,
    SfButton,
    SfCheckbox,
    SfLoader,
    SfAlert,
    ValidationProvider,
    ValidationObserver,
    SfBar
  },
  setup(props, context) {
    const t = (key) => context.root.$i18n.t(key);
    const { isLoginModalOpen, toggleLoginModal } = useUiState();
    const form = ref({});
    const isLogin = ref(true);
    const isForgotten = ref(false);
    const isThankYouAfterForgotten = ref(false);
    const isVerifyUser = ref(false);
    const userEmail = ref('');
    const createAccount = ref(false);
    const rememberMe = ref(false);
    const { register, login, logout, loading, error: userError } = useUser();
    const { request, error: forgotPasswordError, loading: forgotPasswordLoading } = useForgotPassword();
    const { $router } = context.root;
    const { send } = useUiNotification();
    const { $sylius } = useVSFContext();

    const error = reactive({
      login: null,
      register: null
    });

    const resetErrorValues = () => {
      error.login = null;
      error.register = null;
    };

    const barTitle = computed(() => {
      if (isLogin.value) {
        return 'Sign in';
      } else if (isForgotten.value || isThankYouAfterForgotten.value) {
        return 'Reset Password';
      } else {
        return 'Register';
      }
    });

    watch(isLoginModalOpen, () => {
      if (isLoginModalOpen) {
        form.value = {};
        resetErrorValues();
      }
    });

    const setIsLoginValue = (value) => {
      resetErrorValues();
      isLogin.value = value;
    };

    const setIsForgottenValue = (value) => {
      resetErrorValues();
      isForgotten.value = value;
      isLogin.value = !value;
    };

    const setIsVerifyUser = (value) => {
      resetErrorValues();
      isForgotten.value = false;
      isLogin.value = !value;
      isVerifyUser.value = value;
    };

    const handleForm = (fn) => async () => {
      resetErrorValues();
      await fn({ user: form.value });

      const hasUserErrors = userError.value.register || userError.value.login;
      if (hasUserErrors) {
        error.login = userError.value.login?.message;
        error.register = userError.value.register?.message;

        if (error.login === t('Can\'t authenticate, user not verified')) {
          setIsVerifyUser(true);
        } else if (error.login === 'Can\'t authenticate, user not verified') {
          setIsVerifyUser(true);
        }

        return;
      }

      if (fn === register) {
        send({ type: 'info', message: t('Your account has been registered') });

        await login({user: {username: form.value.email, password: form.value.password }});
        if ($sylius?.config?.state?.getCustomerToken() === undefined) {
          setIsVerifyUser(true);
          await logout();

          return;
        }

        setIsLoginValue(false);
      }

      send({ type: 'info', message: t('Login successful') });
      $router.push('/my-account');

      toggleLoginModal();
    };

    const closeModal = () => {
      setIsForgottenValue(false);
      setIsVerifyUser(false);
      toggleLoginModal();
    };

    const handleRegister = async () => handleForm(register)();

    const handleLogin = async () => handleForm(login)();

    const handleForgotten = async () => {
      userEmail.value = form.value.username;
      await request({ email: userEmail.value });

      if (!forgotPasswordError.value.request) {
        isThankYouAfterForgotten.value = true;
        isForgotten.value = false;
      }
    };

    return {
      form,
      error,
      userError,
      loading,
      isLogin,
      createAccount,
      rememberMe,
      isLoginModalOpen,
      toggleLoginModal,
      handleLogin,
      handleRegister,
      setIsLoginValue,
      isForgotten,
      setIsForgottenValue,
      forgotPasswordError,
      forgotPasswordLoading,
      handleForgotten,
      isVerifyUser,
      closeModal,
      isThankYouAfterForgotten,
      userEmail,
      barTitle
    };
  }
};
</script>

<style lang="scss" scoped>

.modal {
  --modal-index: 3;
  --overlay-z-index: 3;
}
.form {
  margin-top: var(--spacer-sm);
  &__element {
    margin: 0 0 var(--spacer-xl) 0;
  }
}
.action {
  display: flex;
  align-items: center;
  justify-content: center;
  margin: var(--spacer-xl) 0 var(--spacer-xl) 0;
  font: var(--font-weight--light) var(--font-size--base) / 1.6 var(--font-family--secondary);
  & > * {
    margin: 0 0 0 var(--spacer-xs);
  }
}
.action {
  margin: var(--spacer-xl) 0 var(--spacer-xl) 0;
}
.checkbox {
  margin-bottom: var(--spacer-2xl);
}
.bottom {
  text-align: center;
  margin-bottom: var(--spacer-lg);
  font-size: var(--h3-font-size);
  font-weight: var(--font-weight--semibold);
  font-family: var(--font-family--secondary);
  &__paragraph {
    color: var(--c-primary);
    margin: 0 0 var(--spacer-base) 0;
    @include for-desktop {
      margin: 0;
    }
  }
}
.thank-you {
  &__paragraph {
    &--bold {
      font-weight: var(--font-weight--semibold);
    }
  }
}
.verify {
  &__button {
    width: 100%;
    margin-top: var(--spacer-xl);
  }

  &__paragraph {
    margin: var(--spacer-lg) 0;
    text-align: center;
  }
}

.login-error {
  text-align: center;
  background: red;
  margin: 0px;
  margin-bottom: 40px;
  padding: 15px;

  .sf-alert {
    color: white;
    --icon-color: white;
  }
}

.light-line {
  border: none;
  border-bottom: solid #ddd 1px;
}

.login-register-separator {
  margin-top: 20px;
  margin-bottom: 20px;
}

.register-today-button {
  margin-top: 20px;
}

.remember-me-checkbox {
  height: 0px;
}
</style>
