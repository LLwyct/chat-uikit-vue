<template>
  <div v-if="!isUniFrameWork" class="memeber-profile">
    <div class="memeber-profile-main">
      <img
        class="avatar"
        :src="
          userInfoManage.avatar ||
          'https://web.sdk.qcloud.com/component/TUIKit/assets/avatar_21.png'
        "
        onerror="this.src='https://web.sdk.qcloud.com/component/TUIKit/assets/avatar_21.png'"
      />
      <ul class="list">
        <h2>{{ userInfoManage.nick || userInfoManage.userID }}</h2>
        <li>
          <label>ID：</label>
          <span>{{ userInfoManage.userID }}</span>
        </li>
        <li>
          <label>{{ TUITranslateService.t("TUIContact.个性签名") }}：</label>
          <span>{{ userInfoManage.selfSignature }}</span>
        </li>
      </ul>
    </div>
    <div class="memeber-profile-footer">
      <div
        class="button"
        @click="enter(userInfoManage.userID, 'C2C')"
        v-if="showEnter()"
      >
        {{ TUITranslateService.t("TUIContact.发送消息") }}
      </div>
    </div>
  </div>
  <div v-else class="edit-h5">
    <main class="main">
      <header class="edit-h5-header">
        <aside class="left">
          <h1>{{ TUITranslateService.t(`TUIGroup.群成员`) }}</h1>
        </aside>
        <span class="close" @click="close('profile')">{{
          TUITranslateService.t(`关闭`)
        }}</span>
      </header>
      <div class="memeber-profile">
        <div class="memeber-profile-main">
          <img
            class="avatar"
            :src="
              userInfoManage.avatar ||
              'https://web.sdk.qcloud.com/component/TUIKit/assets/avatar_21.png'
            "
            onerror="this.src='https://web.sdk.qcloud.com/component/TUIKit/assets/avatar_21.png'"
          />
          <ul class="list">
            <h1>{{ userInfoManage.nick || userInfoManage.userID }}</h1>
            <li>
              <label>ID：</label>
              <span>{{ userInfoManage.userID }}</span>
            </li>
            <li>
              <label
                >{{ TUITranslateService.t("TUIContact.个性签名") }}：</label
              >
              <span>{{ userInfoManage.selfSignature }}</span>
            </li>
          </ul>
        </div>
        <div class="memeber-profile-footer">
          <div
            class="button"
            @click="enter(userInfoManage.userID, 'C2C')"
            v-if="showEnter()"
          >
            {{ TUITranslateService.t("TUIContact.发送消息") }}
          </div>
        </div>
      </div>
    </main>
  </div>
</template>
<script lang="ts" setup>
import {
  TUITranslateService,
  TUIUserService,
  TUIConversationService,
  TUIGlobal,
} from "@tencentcloud/chat-uikit-engine";
import { ref, watch, watchEffect } from "../../../adapter-vue";
import { IUserProfile } from "../../../interface";
import { isUniFrameWork } from "../../../utils/is-uni";

const props = defineProps({
  userInfo: {
    type: Object,
    default: () => ({}),
  },
});

const isFriendShip = ref(false);
const userInfoManage = ref<IUserProfile>({});

watchEffect(() => {
  userInfoManage.value = props.userInfo;
});
const emits = defineEmits([
  "handleSwitchConversation",
  "close",
  "openConversation",
]);

watch(
  () => props.userInfo,
  async (newVal: any, oldVal: any) => {
    if (newVal === oldVal) return;
    const res = await TUIUserService.getUserProfile({
      userIDList: [props.userInfo.userID],
    });
    userInfoManage.value = res?.data[0];
    // 这里需要确认是否是好友关系
    checkFriend();
  },
  {
    deep: true,
    immediate: true,
  }
);

const enter = async (ID: any, type: string) => {
  const name = `${type}${ID}`;
  TUIConversationService.getConversationProfile(name)
    .then((res: any) => {
      TUIConversationService.switchConversation(
        res.data.conversation.conversationID
      );
      if (isUniFrameWork) {
        emits("openConversation");
      } else {
        emits("handleSwitchConversation", res.data.conversation.conversationID);
      }
    })
    .catch((err: any) => {
      console.warn("获取会话资料失败", err.code, err.msg);
    });
};
const checkFriend = async () => {
  if (!(userInfoManage.value as any).userID) return;
  // 这里暂时屏蔽
  // const relation = await TUIFriendService.checkFriend(userInfo.value.userID, TUIChatEngine.TYPES.SNS_CHECK_TYPE_BOTH);
  // isFriendShip.value = (relation === TUIChatEngine.TYPES.SNS_TYPE_BOTH_WAY) ? true : false;
  if (!isUniFrameWork) {
    isFriendShip.value = true;
  }
};

const showEnter = () => {
  return isFriendShip.value || !TUIGlobal.isOfficial;
};

const close = (tabName: string) => {
  emits("close", tabName);
};
</script>
<style lang="scss" scoped>
@import url("../../../assets/styles/common.scss");

.memeber-profile {
  flex: 1;
  display: flex;
  flex-direction: column;

  &-main {
    display: flex;
    flex-direction: row;
    width: 100%;
    overflow: hidden;

    img {
      width: 60px;
      height: 60px;
      border-radius: 8px;
      margin: 20px 10px 20px 20px;
    }

    .list {
      flex: 1;
      overflow: hidden;
      margin: 20px 10px;
      font-weight: 400;

      li {
        color: #999999;
      }

      h1,
      li {
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
      }
    }
  }

  &-footer {
    border-top: 10px solid #f4f5f9;
    padding: 14px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;

    .button {
      width: 100px;
      font-size: 14px;
      cursor: pointer;
      background-color: #006eff;
      color: #ffffff;
      padding: 8px 20px;
      border-radius: 4px;
      border: none;
      font-size: 14px;
      text-align: center;
      line-height: 20px;
    }
  }
}

.edit-h5 {
  position: fixed;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: flex-end;
  z-index: 1;
  .main {
    background: #ffffff;
    flex: 1;
    padding: 18px;
    border-radius: 12px 12px 0 0;
    width: 80vw;
  }

  &-header {
    display: flex;
    align-items: center;
    justify-content: space-between;

    .close {
      font-family: PingFangSC-Regular;
      font-weight: 400;
      font-size: 18px;
      color: #3370ff;
      letter-spacing: 0;
      line-height: 27px;
    }
  }

  &-footer {
    display: flex;

    .btn {
      flex: 1;
      border: none;
      background: #147aff;
      border-radius: 5px;
      font-family: PingFangSC-Regular;
      font-weight: 400;
      font-size: 16px;
      color: #ffffff;
      letter-spacing: 0;
      line-height: 27px;
      padding: 8px 0;

      &:disabled {
        opacity: 0.3;
      }
    }
  }
}
</style>
