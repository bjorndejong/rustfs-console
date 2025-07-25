<script setup lang="ts">
import type { FormItemRule } from 'naive-ui';
import { useI18n } from 'vue-i18n';

const { t } = useI18n();

interface Props {
  visible: boolean;
}
const { visible } = defineProps<Props>();

const emit = defineEmits<Emits>();
const defaultFormModal = {
  current_secret_key: '',
  new_secret_key: '',
  re_new_secret_key: '',
};
const formModel = ref({ ...defaultFormModal });
// 验证规则
const rules = {
  current_secret_key: [
    {
      required: true,
      message: t('Please enter current password'),
      trigger: 'blur',
    },
  ],
  new_secret_key: [
    {
      required: true,
      message: t('Please enter new password'),
      trigger: 'blur',
    },
  ],
  re_new_secret_key: [
    {
      required: true,
      message: t('Please enter new password again'),
      trigger: 'blur',
    },
    {
      validator: validatePasswordSame,
      message: t('The two passwords are inconsistent'),
      trigger: ['blur', 'password-input'],
    },
  ],
};
// 再次输入密码的时候验证两次输入的密码是否一致
function validatePasswordSame(rule: FormItemRule, value: string): boolean {
  return value === formModel.value.new_secret_key;
}

// 输入密码时候验证与下发已经输入的重复密码是否一致
const rPasswordFormItemRef = ref();
function handlePasswordInput() {
  if (formModel.value.re_new_secret_key) {
    rPasswordFormItemRef.value?.validate({ trigger: 'password-input' });
  }
}

// 提交确认
const formRef = ref();
const { $api } = useNuxtApp();
const message = useMessage();
function submitForm(e: MouseEvent) {
  e.preventDefault();
  formRef.value?.validate(async (errors: any) => {
    if (errors) {
      return;
    }

    try {
      const res = await $api.post('/account/change-password', {
        current_secret_key: formModel.value.current_secret_key,
        new_secret_key: formModel.value.new_secret_key,
      });
      message.success(t('Updated successfully'));
      closeModal();
    } catch (error) {
      message.error(t('Update failed'));
    }
  });
}

interface Emits {
  (e: 'update:visible', visible: boolean): void;
}

const modalVisible = computed({
  get() {
    return visible;
  },
  set(visible) {
    closeModal(visible);
  },
});
function closeModal(visible = false) {
  emit('update:visible', visible);
}
</script>

<template>
  <n-modal v-model:show="modalVisible" :mask-closable="false" preset="card" :title="t('Change current account password')" class="max-w-screen-md" sizer="huge" :segmented="{
    content: true,
    action: true,
  }">
    <n-card>
      <n-form ref="formRef" label-placement="left" :model="formModel" :rules="rules" label-align="left" :label-width="130">
        <n-grid :cols="24" :x-gap="18">
          <n-form-item-grid-item :span="24" :label="t('Current Password')" path="current_secret_key">
            <n-input v-model:value="formModel.current_secret_key" show-password-on="mousedown" type="password" />
          </n-form-item-grid-item>
          <n-form-item-grid-item :span="24" :label="t('New Password')" path="new_secret_key">
            <n-input ref="nPasswordFormItemRef" v-model:value="formModel.new_secret_key" show-password-on="mousedown" type="password" @input="handlePasswordInput" />
          </n-form-item-grid-item>
          <n-form-item-grid-item ref="rPasswordFormItemRef" :span="24" :label="t('Confirm New Password')" path="re_new_secret_key">
            <n-input v-model:value="formModel.re_new_secret_key" :disabled="!formModel.new_secret_key" show-password-on="mousedown" type="password" @keydown.enter.prevent />
          </n-form-item-grid-item>
        </n-grid>
      </n-form>
    </n-card>
    <template #action>
      <n-space justify="center">
        <n-button @click="closeModal()">{{ t('Cancel') }}</n-button>
        <n-button type="primary" @click="submitForm">{{ t('Submit') }}</n-button>
      </n-space>
    </template>
  </n-modal>
</template>

<style scoped>
.n-date-picker {
  width: 100%;
}
</style>
