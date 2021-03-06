<script>
import {
  GlAlert,
  GlButton,
  GlSprintf,
  GlLink,
  GlIcon,
  GlFormGroup,
  GlFormInputGroup,
  GlToggle,
  GlModal,
  GlModalDirective,
} from '@gitlab/ui';
import { isEqual } from 'lodash';
import ClipboardButton from '~/vue_shared/components/clipboard_button.vue';
import { I18N_PAGERDUTY_SETTINGS_FORM, CONFIGURE_PAGERDUTY_WEBHOOK_DOCS_LINK } from '../constants';

export default {
  components: {
    GlAlert,
    GlButton,
    GlSprintf,
    GlLink,
    GlIcon,
    GlFormGroup,
    GlFormInputGroup,
    GlToggle,
    GlModal,
    ClipboardButton,
  },
  directives: {
    'gl-modal': GlModalDirective,
  },
  inject: ['service', 'pagerDutySettings'],
  data() {
    return {
      active: this.pagerDutySettings.active,
      webhookUrl: this.pagerDutySettings.webhookUrl,
      loading: false,
      resettingWebhook: false,
      webhookUpdateFailed: false,
      showAlert: false,
    };
  },
  i18n: I18N_PAGERDUTY_SETTINGS_FORM,
  CONFIGURE_PAGERDUTY_WEBHOOK_DOCS_LINK,
  computed: {
    formData() {
      return {
        pagerduty_active: this.active,
      };
    },
    isFormUpdated() {
      return isEqual(this.pagerDutySettings, {
        active: this.active,
        webhookUrl: this.webhookUrl,
      });
    },
    isSaveDisabled() {
      return this.isFormUpdated || this.loading || this.resettingWebhook;
    },
    webhookUpdateAlertMsg() {
      return this.webhookUpdateFailed
        ? this.$options.i18n.webhookUrl.updateErrMsg
        : this.$options.i18n.webhookUrl.updateSuccessMsg;
    },
    webhookUpdateAlertVariant() {
      return this.webhookUpdateFailed ? 'danger' : 'success';
    },
  },
  methods: {
    updatePagerDutyIntegrationSettings() {
      this.loading = true;

      this.service.updateSettings(this.formData).catch(() => {
        this.loading = false;
      });
    },
    resetWebhookUrl() {
      this.resettingWebhook = true;

      this.service
        .resetWebhookUrl()
        .then(({ data: { pagerduty_webhook_url: url } }) => {
          this.webhookUrl = url;
          this.showAlert = true;
          this.webhookUpdateFailed = false;
        })
        .catch(() => {
          this.showAlert = true;
          this.webhookUpdateFailed = true;
        })
        .finally(() => {
          this.resettingWebhook = false;
        });
    },
  },
};
</script>

<template>
  <div>
    <gl-alert
      v-if="showAlert"
      class="gl-mb-3"
      :variant="webhookUpdateAlertVariant"
      @dismiss="showAlert = false"
    >
      {{ webhookUpdateAlertMsg }}
    </gl-alert>

    <p>{{ $options.i18n.introText }}</p>
    <form ref="settingsForm" @submit.prevent="updatePagerDutyIntegrationSettings">
      <gl-form-group class="col-8 col-md-9 gl-p-0">
        <gl-toggle
          id="active"
          v-model="active"
          :is-loading="loading"
          :label="$options.i18n.activeToggle.label"
        />
      </gl-form-group>

      <gl-form-group
        class="col-8 col-md-9 gl-p-0"
        :label="$options.i18n.webhookUrl.label"
        label-for="url"
      >
        <gl-form-input-group id="url" data-testid="webhook-url" readonly :value="webhookUrl">
          <template #append>
            <clipboard-button
              :text="webhookUrl"
              :title="$options.i18n.webhookUrl.copyToClipboard"
            />
          </template>
        </gl-form-input-group>

        <div class="gl-text-gray-200 gl-pt-2">
          <gl-sprintf :message="$options.i18n.webhookUrl.helpText">
            <template #docsLink>
              <gl-link
                :href="$options.CONFIGURE_PAGERDUTY_WEBHOOK_DOCS_LINK"
                target="_blank"
                class="gl-display-inline-flex"
              >
                <span>{{ $options.i18n.webhookUrl.helpDocsLink }}</span>
                <gl-icon name="external-link" />
              </gl-link>
            </template>
          </gl-sprintf>
        </div>
        <gl-button
          v-gl-modal.resetWebhookModal
          class="gl-mt-3"
          :disabled="loading"
          :loading="resettingWebhook"
          data-testid="webhook-reset-btn"
        >
          {{ $options.i18n.webhookUrl.resetWebhookUrl }}
        </gl-button>
        <gl-modal
          modal-id="resetWebhookModal"
          :title="$options.i18n.webhookUrl.resetWebhookUrl"
          :ok-title="$options.i18n.webhookUrl.resetWebhookUrl"
          ok-variant="danger"
          @ok="resetWebhookUrl"
        >
          {{ $options.i18n.webhookUrl.restKeyInfo }}
        </gl-modal>
      </gl-form-group>
      <gl-button
        ref="submitBtn"
        :disabled="isSaveDisabled"
        variant="success"
        type="submit"
        class="js-no-auto-disable"
      >
        {{ $options.i18n.saveBtnLabel }}
      </gl-button>
    </form>
  </div>
</template>
