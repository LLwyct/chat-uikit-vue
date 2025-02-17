<template>
  <ToolbarItemContainer
    :iconFile="evaluateIcon"
    title="评价"
    :needBottomPopup="true"
    :iconWidth="isUniFrameWork ? '26px' : '20px'"
    :iconHeight="isUniFrameWork ? '26px' : '20px'"
    @onDialogClose="onDialogClose"
    ref="container"
  >
    <div :class="['evaluate', !isPC && 'evaluate-h5']">
      <div :class="['evaluate-header', !isPC && 'evaluate-h5-header']">
        <div
          :class="[
            'evaluate-header-content',
            !isPC && 'evaluate-h5-header-content',
          ]"
        >
          {{ TUITranslateService.t("Evaluate.请对本次服务进行评价") }}
        </div>
        <div
          v-if="!isPC"
          :class="[
            'evaluate-header-close',
            !isPC && 'evaluate-h5-header-close',
          ]"
          @click.stop="closeDialog"
        >
          {{ TUITranslateService.t("关闭") }}
        </div>
      </div>
      <div :class="['evaluate-content', !isPC && 'evaluate-h5-content']">
        <ul
          :class="[
            'evaluate-content-list',
            !isPC && 'evaluate-h5-content-list',
          ]"
        >
          <li
            :class="[
              'evaluate-content-list-item',
              !isPC && 'evaluate-h5-content-list-item',
            ]"
            v-for="(item, index) in starList"
            :key="index"
            @click.stop="selectStar(index)"
          >
            <Icon
              v-if="index <= currentStarIndex"
              :file="starLightIcon"
              :width="isPC ? '20px' : '30px'"
              :height="isPC ? '20px' : '30px'"
            ></Icon>
            <Icon
              v-else
              :file="starIcon"
              :width="isPC ? '20px' : '30px'"
              :height="isPC ? '20px' : '30px'"
            ></Icon>
          </li>
        </ul>
        <textarea
          :class="[
            'evaluate-content-text',
            !isPC && 'evaluate-h5-content-text',
          ]"
          v-model="comment"
        ></textarea>
        <div
          :class="[
            'evaluate-content-button',
            !isPC && 'evaluate-h5-content-button',
          ]"
        >
          <button class="btn" @click="submitEvaluate">
            {{ TUITranslateService.t("Evaluate.提交评价") }}
          </button>
        </div>
      </div>
      <div :class="['evaluate-adv', !isPC && 'evaluate-h5-adv']">
        {{ TUITranslateService.t("Evaluate.服务评价工具") }}
        {{ "(" + TUITranslateService.t("Evaluate.使用") }}
        <a @click="openLink(Link.customMessage)">
          {{ TUITranslateService.t(`Evaluate.${Link.customMessage.label}`) }}
        </a>
        {{ TUITranslateService.t("Evaluate.搭建") + ")" }}
      </div>
    </div>
  </ToolbarItemContainer>
</template>
<script setup lang="ts">
import {
  TUIGlobal,
  TUITranslateService,
  TUIStore,
  StoreName,
  IConversationModel,
  SendMessageParams,
  TUIChatService,
} from "@tencentcloud/chat-uikit-engine";
import { ref } from "../../../../adapter-vue";
import ToolbarItemContainer from "../toolbar-item-container/index.vue";
import evaluateIcon from "../../../../assets/icon/evaluate.svg";
import Link from "../../../../utils/documentLink";
import Icon from "../../../common/Icon.vue";
import starIcon from "../../../../assets/icon/star.png";
import starLightIcon from "../../../../assets/icon/star-light.png";
import { CHAT_MSG_CUSTOM_TYPE } from "../../../../constant";
import { isUniFrameWork } from "../../../../utils/is-uni";

const props = defineProps({
  starTotal: {
    type: Number,
    default: 5,
  },
});

const isPC = ref(TUIGlobal.getPlatform() === "pc");
const isH5 = ref(TUIGlobal.getPlatform() === "h5");
const isApp = ref(TUIGlobal.getPlatform() === "app");
const container = ref();

const starList = ref<number>(props.starTotal);
const currentStarIndex = ref<number>(-1);
const comment = ref("");
const currentConversation = ref<typeof IConversationModel>();

TUIStore.watch(StoreName.CONV, {
  currentConversation: (conversation: typeof IConversationModel) => {
    currentConversation.value = conversation;
  },
});

const onDialogClose = () => {
  resetEvaluate();
};

const openLink = (link: any) => {
  if (isPC.value || isH5.value) {
    window.open(Link?.customMessage?.url);
  }
};

const closeDialog = () => {
  container?.value?.toggleDialogDisplay(false);
};

const resetEvaluate = () => {
  currentStarIndex.value = -1;
  comment.value = "";
};

const selectStar = (starIndex?: any) => {
  if (currentStarIndex.value === starIndex) {
    currentStarIndex.value = currentStarIndex.value - 1;
  } else {
    currentStarIndex.value = starIndex;
  }
};

const submitEvaluate = () => {
  if (currentStarIndex.value < 0) {
    return;
  }
  const options = {
    to:
      currentConversation?.value?.groupProfile?.groupID ||
      currentConversation?.value?.userProfile?.userID,
    conversationType: currentConversation?.value?.type,
    payload: {
      data: JSON.stringify({
        businessID: CHAT_MSG_CUSTOM_TYPE.EVALUATE,
        version: 1,
        score: currentStarIndex.value + 1,
        comment: comment.value,
      }),
      description: "对本次的服务评价",
      extension: "对本次的服务评价",
    },
  };
  TUIChatService.sendCustomMessage(options);
  // 提交后关闭 dialog
  // close dialog after submit evaluate
  container?.value?.toggleDialogDisplay(false);
};
</script>
<style scoped lang="scss" src="./style/index.scss"></style>
