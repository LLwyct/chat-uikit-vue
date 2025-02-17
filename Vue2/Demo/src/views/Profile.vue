<template>
  <div :class="['TUI-profile', !isPC && 'TUI-profile-h5']">
    <div
      v-if="displayType !== 'setting'"
      :class="['TUI-profile-basic', !isPC && 'TUI-profile-h5-basic']"
    >
      <img
        :class="[
          'TUI-profile-basic-avatar',
          !isPC && 'TUI-profile-h5-basic-avatar',
        ]"
        :src="
          userProfile.avatar ||
          'https://web.sdk.qcloud.com/component/TUIKit/assets/avatar_21.png'
        "
      />
      <div
        :class="[
          'TUI-profile-basic-info',
          !isPC && 'TUI-profile-h5-basic-info',
        ]"
      >
        <div
          :class="[
            'TUI-profile-basic-info-nick',
            !isPC && 'TUI-profile-h5-basic-info-nick',
          ]"
        >
          {{ userProfile.nick || "-" }}
        </div>
        <div
          :class="[
            'TUI-profile-basic-info-id',
            !isPC && 'TUI-profile-h5-basic-info-id',
          ]"
        >
          <label
            :class="[
              'TUI-profile-basic-info-id-label',
              !isPC && 'TUI-profile-h5-basic-info-id-label',
            ]"
            >{{ TUITranslateService.t("TUIProfile.用户ID") }}:</label
          >
          <div
            :class="[
              'TUI-profile-basic-info-id-value',
              !isPC && 'TUI-profile-h5-basic-info-id-value',
            ]"
          >
            {{ userProfile.userID }}
          </div>
        </div>
      </div>
    </div>
    <div
      v-if="displayType !== 'profile' && (!isPC || showSetting)"
      ref="settingDomRef"
      :class="['TUI-profile-setting', !isPC && 'TUI-profile-h5-setting']"
      @click.stop
    >
      <div
        v-for="item in settingList"
        :key="item.value"
        :class="[
          'TUI-profile-setting-item',
          !isPC && 'TUI-profile-h5-setting-item',
          item.value === 'exit' && 'TUI-profile-h5-setting-item-exit',
        ]"
      >
        <div
          @click="handleSettingListItemOnClick(item)"
          :class="[
            'TUI-profile-setting-item-label',
            !isPC && 'TUI-profile-h5-setting-item-label',
          ]"
        >
          <div :class="['label-left']">
            <div :class="['label-title']">
              {{ item.label }}
            </div>
            <div
              :class="['label-desc']"
              v-if="
                item.children && !isPC && item.childrenShowType === 'switch'
              "
            >
              {{ item.value }}
            </div>
          </div>
          <div :class="['label-right']">
            <div
              v-if="
                !isPC &&
                item.children &&
                item.selectedChild &&
                item.childrenShowType === 'bottomPopup'
              "
              :class="[
                'TUI-profile-setting-item-label-value',
                !isPC && 'TUI-profile-h5-setting-item-label-value',
              ]"
            >
              {{ item?.children[item.selectedChild]?.label }}
            </div>
            <Icon
              v-if="item.children"
              :file="rightArrowIcon"
              width="14px"
              height="14px"
            ></Icon>
          </div>
        </div>
        <div
          v-if="item.children && isPC"
          :class="[
            'TUI-profile-setting-item-children',
            !isPC && 'TUI-profile-h5-setting-item-children',
          ]"
        >
          <div
            :class="[
              'TUI-profile-setting-item-children-item',
              !isPC && 'TUI-profile-h5-setting-item-children-item',
            ]"
            v-for="child in item.children"
            :key="child.value"
            @click="handleSettingListItemOnClick(child)"
          >
            <div
              :class="[
                'TUI-profile-setting-item-children-item-label',
                !isPC && 'TUI-profile-h5-setting-item-children-item-label',
              ]"
            >
              {{ child.label }}
            </div>
            <Icon
              v-if="item.selectedChild === child.value"
              :file="selectedIcon"
              width="14px"
              height="14px"
            ></Icon>
          </div>
        </div>
        <!-- 移动端 children显示，分多个类型 -->
        <BottomPopup
          v-if="
            item.children && !isPC && item.childrenShowType === 'bottomPopup'
          "
          :show="item.showChildren"
          @onClose="item.showChildren = false"
        >
          <div
            :class="[
              'TUI-profile-setting-item-bottom-popup',
              !isPC && 'TUI-profile-h5-setting-item-bottom-popup',
            ]"
            v-for="child in item.children"
            :key="child.value"
            @click="handleSettingListItemOnClick(child)"
          >
            {{ child.label }}
          </div>
        </BottomPopup>
      </div>
    </div>
  </div>
</template>
<script lang="ts" setup>
import TUIChatEngine, {
  TUIGlobal,
  TUITranslateService,
  TUIUserService,
  TUIStore,
  StoreName,
} from "@tencentcloud/chat-uikit-engine";
import { TUILogin } from "@tencentcloud/tui-core";
import {
  ref,
  watch,
  nextTick,
  defineProps,
  defineEmits,
  onMounted,
} from "../TUIKit/adapter-vue";
import { Toast, TOAST_TYPE } from "../TUIKit/components/common/Toast/index";
import BottomPopup from "../TUIKit/components/common/BottomPopup/index.vue";
import Icon from "../TUIKit/components/common/Icon.vue";
import rightArrowIcon from "../TUIKit/assets/icon/right-icon.svg";
import selectedIcon from "../TUIKit/assets/icon/selected.svg";
import { IUserProfile } from "../TUIKit/interface";
import router from "../router/index";

const props = defineProps({
  displayType: {
    type: String,
    default: "profile", // "profile"/"setting"/"all"
  },
  showSetting: {
    type: Boolean,
    default: false,
  },
});
const emits = defineEmits(["update:showSetting"]);

const settingDomRef = ref();
const userProfile = ref<IUserProfile>({});
const isPC = ref(TUIGlobal.getPlatform() === "pc");
const settingList = ref<{
  [propsName: string]: {
    value: string;
    label: string;
    onClick?: any;
    // children相关
    selectedChild?: string;
    childrenShowType?: string; // "bottomPopup"/"switch"
    showChildren?: boolean;
    children?: {
      [propsName: string]: {
        value: string;
        label: string;
        onClick?: any;
      };
    };
  };
}>({
  editProfile: {
    value: "editProfile",
    label: "编辑资料（暂未开放）",
    onClick: (item: any) => {
      console.warn("编辑资料功能努力开发中，敬请期待");
    },
  },
  allowType: {
    value: "allowType",
    label: "加我为好友时",
    selectedChild: "",
    childrenShowType: "bottomPopup",
    showChildren: false,
    onClick: (item: any) => {
      if (!isPC.value) {
        item.showChildren = true;
      }
    },
    children: {
      [TUIChatEngine.TYPES.ALLOW_TYPE_ALLOW_ANY]: {
        value: TUIChatEngine.TYPES.ALLOW_TYPE_ALLOW_ANY,
        label: "同意任何用户加好友",
        onClick: (item: any) => {
          updateMyProfile({ allowType: item.value });
        },
      },
      [TUIChatEngine.TYPES.ALLOW_TYPE_NEED_CONFIRM]: {
        value: TUIChatEngine.TYPES.ALLOW_TYPE_NEED_CONFIRM,
        label: "需要验证",
        onClick: (item: any) => {
          updateMyProfile({ allowType: item.value });
        },
      },
      [TUIChatEngine.TYPES.ALLOW_TYPE_DENY_ANY]: {
        value: TUIChatEngine.TYPES.ALLOW_TYPE_DENY_ANY,
        label: "拒绝任何人加好友",
        onClick: (item: any) => {
          updateMyProfile({ allowType: item.value });
        },
      },
    },
  },
  exit: {
    value: "exit",
    label: "退出登录",
    onClick: (item: any) => {
      TUILogin.logout().then(() => {
        router.push({ path: "/" });
      });
    },
  },
});

const handleSettingListItemOnClick = (item: any) => {
  if (item?.onClick && typeof item?.onClick === "function") {
    item.onClick(item);
  }
};

const updateMyProfile = (props: object) => {
  TUIUserService.updateMyProfile(props)
    .then((res: any) => {
      Toast({
        message: "更新用户资料成功",
        type: TOAST_TYPE.SUCCESS,
      });
    })
    .catch((err: any) => {
      console.warn("更新用户资料失败", err);
      Toast({
        message: "更新用户资料失败",
        type: TOAST_TYPE.ERROR,
      });
    });
};

TUIStore.watch(StoreName.USER, {
  userProfile: (userProfileData: IUserProfile) => {
    userProfile.value = userProfileData;
    if (userProfile?.value?.allowType) {
      settingList.value.allowType.selectedChild = userProfile?.value?.allowType;
    }
  },
});

// pc端
// 处理 setting 点击外部关闭
// click outside
let clickOutside = false;
let clickInner = false;
const onClickOutside = (component: any) => {
  document.addEventListener("mousedown", onClickDocument);
  component?.addEventListener &&
    component?.addEventListener("mousedown", onClickTarget);
};
const onClickDocument = () => {
  clickOutside = true;
  if (!clickInner && clickOutside) {
    emits("update:showSetting", false);
    removeClickListener(settingDomRef.value);
  }
  clickOutside = false;
  clickInner = false;
};
const onClickTarget = () => {
  clickInner = true;
};
const removeClickListener = (component: any) => {
  document.removeEventListener("mousedown", onClickDocument);
  component?.removeEventListener &&
    component?.removeEventListener("mousedown", onClickTarget);
};

watch(
  () => props.showSetting,
  (newVal: boolean, oldVal: boolean) => {
    if (isPC.value && newVal && !oldVal) {
      nextTick(() => {
        onClickOutside(settingDomRef.value);
      });
    }
  },
  {
    immediate: true,
  }
);

// 规避TUIStore.watch userProfile 登录后暂时不能及时触发更新
onMounted(() => {
  // 查询自己的资料
  TUIUserService.getUserProfile().then((res: any) => {
    userProfile.value = res.data;
  });
});
</script>

<style lang="scss" scoped src="../styles/profile/index.scss"></style>
