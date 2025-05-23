<template>
  <app-dialog v-model="model" width="344px" is-centered>
    <div class="rename-account-form">
      <h3>Rename account</h3>

      <div
        class="rename-account-form__input"
        :class="{ focus: isFocus && isValidName, error: !isValidName }"
      >
        <img
          :src="network.identicon(network.displayAddress(account.address))"
        />
        <input
          ref="renameAccountInput"
          v-model="accountName"
          type="text"
          placeholder="Account name"
          autocomplete="off"
          autofocus
          @focus="changeFocus"
          @blur="changeFocus"
        />
      </div>

      <p>
        Name your account something that makes sense to you! Main account, dapp
        account, yolo account, etc.
      </p>

      <div class="rename-account-form__buttons">
        <div class="rename-account-form__buttons-cancel">
          <base-button
            title="Cancel"
            :click="closeWindow"
            :no-background="true"
          />
        </div>
        <div class="rename-account-form__buttons-send">
          <base-button
            title="Rename account"
            :click="renameAccount"
            :disabled="!isValidName || isProcessing"
          />
        </div>
      </div>
    </div>
  </app-dialog>
</template>

<script setup lang="ts">
import { PropType, ref, onMounted, computed } from 'vue';
import AppDialog from '@action/components/app-dialog/index.vue';
import BaseButton from '@action/components/base-button/index.vue';
import { NodeType } from '@/types/provider';
import { EnkryptAccount } from '@enkryptcom/types';
import KeyRing from '@/libs/keyring/keyring';
import BackupState from '@/libs/backup-state';

const model = defineModel<boolean>();
const closeWindow = () => {
  model.value = false;
};
const isFocus = ref(false);
const accountName = ref('');
const renameAccountInput = ref(null);
const isProcessing = ref(false);
defineExpose({ renameAccountInput });

const emit = defineEmits<{
  (e: 'update:init'): void;
}>();
const props = defineProps({
  network: {
    type: Object as PropType<NodeType>,
    default: () => ({}),
  },
  account: {
    type: Object as PropType<EnkryptAccount>,
    default: () => ({}),
  },
});
const currentNames: string[] = [];
const keyring = new KeyRing();

const isValidName = computed(() => {
  if (accountName.value === props.account.name) return true;
  if (accountName.value.length < 3) return false;
  if (currentNames.includes(accountName.value)) return false;
  return true;
});

onMounted(() => {
  if (renameAccountInput.value) {
    (renameAccountInput.value as HTMLInputElement).focus();
  }
  accountName.value = props.account.name;
  keyring.getKeysArray().then(accounts => {
    accounts.forEach(acc => currentNames.push(acc.name));
  });
});
const changeFocus = () => {
  isFocus.value = !isFocus.value;
};
const renameAccount = () => {
  isProcessing.value = true;
  keyring.renameAccount(props.account.address, accountName.value).then(() => {
    const backupState = new BackupState();
    backupState.backup(false).catch(() => {
      console.error('Failed to backup');
    });
    closeWindow();
    emit('update:init');
  });
};
</script>

<style lang="less" scoped>
@import '@action/styles/theme.less';
.rename-account-form {
  padding: 16px;

  h3 {
    font-style: normal;
    font-weight: 700;
    font-size: 24px;
    line-height: 32px;
    color: @primaryLabel;
    margin: 0 0 16px 0;
  }
  p {
    font-style: normal;
    font-weight: 400;
    font-size: 12px;
    line-height: 16px;
    letter-spacing: 0.5px;
    color: @tertiaryLabel;
    margin: 0 0 20px 0;
  }
  &__input {
    width: 312px;
    height: 56px;
    border: 2px solid @gray02;
    border-radius: 10px;
    padding: 12px;
    display: flex;
    box-sizing: border-box;
    justify-content: flex-start;
    align-items: center;
    flex-direction: row;
    margin-bottom: 16px;
    &.error {
      border: 2px solid @error;
      line-height: 38px;
    }
    &.focus {
      border: 2px solid @primary;
    }
    img {
      width: 32px;
      height: 32px;
      margin-right: 12px;
      border-radius: 50%;
    }
    input {
      width: 100%;
      font-style: normal;
      font-weight: normal;
      font-size: 16px;
      line-height: 24px;
      color: @primaryLabel;
      border: 0 none;
      outline: none;
    }
  }
  &__buttons {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-direction: row;
    width: 100%;
    &-cancel {
      width: 132px;
    }
    &-send {
      width: 172px;
    }
  }
}
</style>
