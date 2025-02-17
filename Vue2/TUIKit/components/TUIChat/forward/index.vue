<template>
  <Overlay v-if="isShowForwardPanel">
    <Transfer
      :title="'转发'"
      :isSearch="false"
      :isCustomItem="false"
      :list="customConversationList"
      :isHiddenBackIcon="isUniFrameWork"
      @cancel="closeForwardPanel"
      @submit="onSubmit"
    ></Transfer>
  </Overlay>
</template>
    
<script lang="ts" setup>
import { onUnmounted, ref } from "../../../adapter-vue";
import {
  TUIStore,
  TUIGlobal,
  StoreName,
  TUIChatService,
  TUITranslateService,
  type IConversationModel,
  type IMessageModel
} from "@tencentcloud/chat-uikit-engine";
import Overlay from "../../common/Overlay/index.vue";
import Transfer from "../../common/Transfer/index.vue";
import { Toast, TOAST_TYPE } from "../../../components/common/Toast";
import { isUniFrameWork } from "../../../utils/is-uni";

const isPC = TUIGlobal.getPlatform() === "pc";
const isShowForwardPanel = ref(false);
const customConversationList = ref();

TUIStore.watch(StoreName.CONV, {
  conversationList: (list: IConversationModel[]) => {
    customConversationList.value = list.map((conversation) => {
      return {
        // 为了实现Transfer的复用，这里用userID代替ConversationID
        userID: conversation.conversationID,
        nick: conversation.getShowName(),
        avatar: conversation.getAvatar(),
      };
    });
  },
});

TUIStore.watch(StoreName.CUSTOM, {
  singleForwardMessageID(messageID: string) {
    if (typeof messageID !== 'undefined') {
      openForwardPanel();
    }
  },
});

onUnmounted(() => {
  // 组件卸载时需要清掉数据 否则小程序会自动打开
  TUIStore.update(StoreName.CUSTOM, 'singleForwardMessageID', undefined);
});

function closeForwardPanel(): void {
  // ! 必须通过close函数关闭转发面板 singleForwardMessage必须清掉
  TUIStore.update(StoreName.CUSTOM, 'singleForwardMessageID', undefined);
  isShowForwardPanel.value = false;
}

function openForwardPanel(): void {
  isShowForwardPanel.value = true;
}

function finishSelected(selectedConvIDWrapperList: Array<{userID: string}>): void {
  /**
   * 这里传递的是 coversationID
   * 但为了实现 Transfer 的复用 这里用 userID 代替 ConversationID
   */
  const selectedConversationList = selectedConvIDWrapperList.map((convIDWrapper) => {
    const { userID: conversationID } = convIDWrapper;
    return TUIStore.getConversationModel(conversationID);
  });
  const singleForwardMessageID: string = TUIStore.getData(StoreName.CUSTOM, 'singleForwardMessageID');
  const message = TUIStore.getMessageModel(singleForwardMessageID);
  TUIChatService.sendForwardMessage(selectedConversationList, [message])
    .catch((error: {message: string, code: number}) => {
      if (error.code === 80001) {
        Toast({
          message: TUITranslateService.t('内容包含敏感词汇') || '内容包含敏感词汇',
          type: TOAST_TYPE.ERROR
        });
      } else {
        Toast({
          message: error.message as string,
          type: TOAST_TYPE.ERROR
        });
      }
    });
  closeForwardPanel();
}

function onSubmit(convIDWrapperList: Array<{userID: string}>) {
  if (convIDWrapperList?.length === 0) return;
  finishSelected(convIDWrapperList);
}
</script>