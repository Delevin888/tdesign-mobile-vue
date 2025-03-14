<template>
  <t-popup
    :visible="visible"
    placement="center"
    :show-overlay="showOverlay"
    :overlay-props="overlayProps"
    :prevent-scroll-through="preventScrollThrough"
    :destroy-on-close="destroyOnClose"
    @close="handleOverlayClick"
    @closed="handleClosed"
  >
    <div :class="`${name}`" :style="rootStyles">
      <slot name="top" />
      <div v-if="closeBtn" :class="`${name}__close-btn`">
        <close-icon @click="handleClose" />
      </div>
      <div :class="`${name}__content`">
        <div v-if="titleNode" :class="`${name}__header`">
          <t-node :content="titleNode" />
        </div>
        <div v-if="contentNode" :class="`${name}__body`">
          <div :class="`${name}__body-text`">
            <t-node :content="contentNode" />
          </div>
        </div>
      </div>
      <slot name="middle" />
      <div :class="footerClass">
        <slot name="actions">
          <template v-if="actionsBtnProps">
            <t-button
              v-for="(item, index) in actionsBtnProps"
              :key="index"
              v-bind="item"
              :class="buttonClass"
              @click="handleCancel"
            />
          </template>
        </slot>
        <slot name="cancelBtn">
          <template v-if="!actions && cancelBtn">
            <t-button v-bind="cancelBtnProps" :class="buttonClass" @click="handleCancel" />
          </template>
        </slot>
        <slot name="confirmBtn">
          <template v-if="!actions && confirmBtn">
            <t-button v-bind="confirmBtnProps" :class="buttonClass" @click="handleConfirm" />
          </template>
        </slot>
      </div>
    </div>
  </t-popup>
</template>
<script lang="ts">
import { CloseIcon } from 'tdesign-icons-vue-next';
import { computed, toRefs, defineComponent, getCurrentInstance } from 'vue';
import get from 'lodash/get';
import isString from 'lodash/isString';

import TButton, { ButtonProps } from '../button';
import TPopup from '../popup';
import config from '../config';
import DialogProps from './props';
import { renderContent, renderTNode, TNode } from '../shared';

const { prefix } = config;
const name = `${prefix}-dialog`;

export default defineComponent({
  name,
  components: { TPopup, TNode, TButton, CloseIcon },
  props: DialogProps,
  emits: ['update:visible', 'confirm', 'overlay-click', 'cancel', 'close', 'closed'],
  setup(props, context) {
    const internalInstance = getCurrentInstance();
    const contentNode = computed(() => renderContent(internalInstance, 'default', 'content'));
    const titleNode = computed(() => renderTNode(internalInstance, 'title'));
    const isTextStyleBtn = computed(() =>
      [props?.confirmBtn, props?.cancelBtn, ...(props?.actions || [])].some((item) => get(item, 'variant') === 'text'),
    );

    const footerClass = computed(() => [
      `${name}__footer`,
      {
        [`${name}__footer--column`]: props.buttonLayout === 'vertical',
        [`${name}__footer--full`]: isTextStyleBtn.value && get(props.actions, 'length', 0) === 0,
      },
    ]);

    const buttonClass = computed(() => [
      `${name}__button`,
      {
        [`${name}__button--${props.buttonLayout}`]: !isTextStyleBtn.value,
        [`${name}__button--text`]: isTextStyleBtn.value,
      },
    ]);

    const rootStyles = computed(() => ({
      zIndex: props.zIndex,
      width: isString(props.width) ? props.width : `${props.width}px`,
    }));

    const handleClose = (args: { e: MouseEvent }) => {
      const { e } = args;
      context.emit('update:visible', false);
      context.emit('close', { e, trigger: 'close-btn' });
    };

    const handleClosed = () => {
      context.emit('closed');
    };

    const handleConfirm = (e: MouseEvent) => {
      context.emit('update:visible', false);
      context.emit?.('confirm', { e });
    };

    const handleCancel = (e: MouseEvent) => {
      context.emit('update:visible', false);
      context.emit('close', { e, trigger: 'cancel' });
      context.emit('cancel', { e });
    };

    const handleOverlayClick = (args: { e: MouseEvent }) => {
      const { e } = args;
      if (!props.closeOnOverlayClick) {
        return;
      }
      context.emit('update:visible', false);
      context.emit('close', { e, trigger: 'overlay' });
      context.emit('overlay-click', { e });
    };

    const calcBtn = (btn: any) => (isString(btn) ? { content: btn } : btn);
    const confirmBtnProps = computed(() => ({
      theme: 'primary',
      ...calcBtn(props.confirmBtn),
    }));
    const cancelBtnProps = computed<ButtonProps>(() => ({
      theme: isTextStyleBtn.value ? 'default' : 'light',
      ...calcBtn(props.cancelBtn),
    }));
    const actionsBtnProps = computed(() => props.actions?.map((item) => calcBtn(item)));

    return {
      name,
      footerClass,
      buttonClass,
      contentNode,
      titleNode,
      confirmBtnProps,
      cancelBtnProps,
      actionsBtnProps,
      handleClose,
      handleClosed,
      handleConfirm,
      handleCancel,
      handleOverlayClick,
      rootStyles,
      ...toRefs(props),
    };
  },
});
</script>
