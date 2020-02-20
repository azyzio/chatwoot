<template>
  <div class="columns profile--settings ">
    <form @submit.prevent="updateUser">
      <div class="small-12 row profile--settings--row">
        <div class="columns small-3 ">
          <p class="section--title">
            {{ $t('PROFILE_SETTINGS.FORM.PROFILE_SECTION.TITLE') }}
          </p>
          <p>{{ $t('PROFILE_SETTINGS.FORM.PROFILE_SECTION.NOTE') }}</p>
        </div>
        <div class="columns small-9">
          <label>
            {{ $t('PROFILE_SETTINGS.FORM.PROFILE_IMAGE.LABEL') }}
            <thumbnail size="80px" :src="avatarUrl"></thumbnail>
            <input
              id="file"
              ref="file"
              type="file"
              accept="image/*"
              @change="handleImageUpload"
            />
          </label>
          <label :class="{ error: $v.name.$error }">
            {{ $t('PROFILE_SETTINGS.FORM.NAME.LABEL') }}
            <input
              v-model="name"
              type="text"
              :placeholder="$t('PROFILE_SETTINGS.FORM.NAME.PLACEHOLDER')"
              @input="$v.name.$touch"
            />
            <span v-if="$v.name.$error" class="message">
              {{ $t('PROFILE_SETTINGS.FORM.NAME.ERROR') }}
            </span>
          </label>
          <label :class="{ error: $v.email.$error }">
            {{ $t('PROFILE_SETTINGS.FORM.EMAIL.LABEL') }}
            <input
              v-model.trim="email"
              type="email"
              :placeholder="$t('PROFILE_SETTINGS.FORM.EMAIL.PLACEHOLDER')"
              @input="$v.email.$touch"
            />
            <span v-if="$v.email.$error" class="message">
              {{ $t('PROFILE_SETTINGS.FORM.EMAIL.ERROR') }}
            </span>
          </label>
        </div>
      </div>
      <div class="profile--settings--row row">
        <div class="columns small-3 ">
          <p class="section--title">
            {{ $t('PROFILE_SETTINGS.FORM.PASSWORD_SECTION.TITLE') }}
          </p>
          <p>{{ $t('PROFILE_SETTINGS.FORM.PASSWORD_SECTION.NOTE') }}</p>
        </div>
        <div class="columns small-9">
          <label :class="{ error: $v.password.$error }">
            {{ $t('PROFILE_SETTINGS.FORM.PASSWORD.LABEL') }}
            <input
              v-model.trim="password"
              type="password"
              :placeholder="$t('PROFILE_SETTINGS.FORM.PASSWORD.PLACEHOLDER')"
              @input="$v.password.$touch"
            />
            <span v-if="$v.password.$error" class="message">
              {{ $t('PROFILE_SETTINGS.FORM.PASSWORD.ERROR') }}
            </span>
          </label>
          <label :class="{ error: $v.passwordConfirmation.$error }">
            {{ $t('PROFILE_SETTINGS.FORM.PASSWORD_CONFIRMATION.LABEL') }}
            <input
              v-model.trim="passwordConfirmation"
              type="password"
              :placeholder="
                $t('PROFILE_SETTINGS.FORM.PASSWORD_CONFIRMATION.PLACEHOLDER')
              "
              @input="$v.passwordConfirmation.$touch"
            />
            <span v-if="$v.passwordConfirmation.$error" class="message">
              {{ $t('PROFILE_SETTINGS.FORM.PASSWORD_CONFIRMATION.ERROR') }}
            </span>
          </label>
        </div>
      </div>

      <woot-submit-button
        class="button nice success button--fixed-right-top"
        :button-text="$t('PROFILE_SETTINGS.BTN_TEXT')"
        :loading="isUpdating"
      >
      </woot-submit-button>
    </form>
  </div>
</template>

<script>
/* global bus */

import Thumbnail from 'dashboard/components/widgets/Thumbnail.vue';
import { required, minLength, email } from 'vuelidate/lib/validators';
import { mapGetters } from 'vuex';
import { clearCookiesOnLogout } from '../../../../store/utils/api';

export default {
  components: {
    Thumbnail,
  },
  data() {
    return {
      avatarFile: '',
      avatarUrl: '',
      name: '',
      email: '',
      password: '',
      passwordConfirmation: '',
      isUpdating: false,
    };
  },
  validations: {
    name: {
      required,
    },
    email: {
      required,
      email,
    },
    password: {
      minLength: minLength(6),
    },
    passwordConfirmation: {
      minLength: minLength(6),
      isEqPassword(value) {
        if (value !== this.password) {
          return false;
        }
        return true;
      },
    },
  },
  computed: {
    ...mapGetters({
      currentUser: 'getCurrentUser',
      currentUserId: 'getCurrentUserID',
    }),
  },
  watch: {
    currentUserId(newCurrentUserId, prevCurrentUserId) {
      if (prevCurrentUserId !== newCurrentUserId) {
        this.initializeUser();
      }
    },
  },
  mounted() {
    if (this.currentUserId) {
      this.initializeUser();
    }
  },
  methods: {
    initializeUser() {
      this.name = this.currentUser.name;
      this.email = this.currentUser.email;
      this.avatarUrl = this.currentUser.avatar_url;
    },
    async updateUser() {
      this.$v.$touch();
      if (this.$v.$invalid) {
        bus.$emit('newToastMessage', this.$t('PROFILE_SETTINGS.FORM.ERROR'));
        return;
      }
      this.isUpdating = true;
      const hasEmailChanged = this.currentUser.email !== this.email;
      try {
        await this.$store.dispatch('updateProfile', {
          name: this.name,
          email: this.email,
          avatar: this.avatarFile,
          password: this.password,
          password_confirmation: this.passwordConfirmation,
        });
        this.isUpdating = false;
        if (hasEmailChanged) {
          clearCookiesOnLogout();
          bus.$emit(
            'newToastMessage',
            this.$t('PROFILE_SETTINGS.AFTER_EMAIL_CHANGED')
          );
        }
      } catch (error) {
        this.isUpdating = false;
      }
    },
    handleImageUpload(event) {
      const [file] = event.target.files;
      this.avatarFile = file;
      this.avatarUrl = URL.createObjectURL(file);
    },
  },
};
</script>

<style lang="scss" scoped>
@import '~dashboard/assets/scss/variables.scss';
@import '~dashboard/assets/scss/mixins.scss';

.profile--settings {
  padding: 24px;
  overflow: auto;
}

.profile--settings--row {
  @include border-normal-bottom;
  padding: 16px;
  .small-3 {
    padding: 16px 16px 16px 0;
  }
  .small-9 {
    padding: 16px;
  }
}

.section--title {
  color: $color-woot;
}
</style>