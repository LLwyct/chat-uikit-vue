<template>
  <div :class="[!isPC ? 'login-h5' : 'login']">
    <Header class="login-header">
      <template v-slot:left>
        <div class="logo">
          <a @click="openDownloadLink(Link.im)">
            <img src="../assets/image/txc-logo.svg" alt="" />
            <label class="logo-name">{{
              TUITranslateService.t("腾讯云")
            }}</label>
            <p>
              <label class="logo-name">{{
                TUITranslateService.t("即时通信IM")
              }}</label>
            </p>
          </a>
        </div>
      </template>
      <template v-slot:right>
        <el-dropdown>
          <span class="dropdown">
            <i class="icon icon-global"></i>
            <label>{{ TUITranslateService.t("当前语言") }}</label>
            <i class="icon icon-arrow-down"></i>
          </span>
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item command="zh_cn" class="language-item">简体中文</el-dropdown-item>
              <el-dropdown-item command="en" class="language-item">
                <a @click="openDownloadLink(Link.intl)" class="language-intl">
                  English(敬请期待)
                </a>
              </el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
      </template>
    </Header>
    <main class="login-main">
      <div class="login-main-content">
        <div class="login-main-adv" :class="[locale === 'en' && 'small-txt']" v-if="isPC">
          <p class="login-main-adv-introduce">
            {{ TUITranslateService.t("Login.超10亿用户的信赖") }}<br />
            {{ TUITranslateService.t("腾讯云")
            }}{{ TUITranslateService.t("即时通信") }}
          </p>
          <a class="login-sale" @click="openDownloadLink(Link.adv)">
            <label>{{ TUITranslateService.t("Home.IM首购9percent折") }},
              {{ TUITranslateService.t("Home.复购75%起") }}!
              {{ TUITranslateService.t("Home.立即选购") }}
            </label>
            <i class="icon icon-adv-arrow"></i>
          </a>
        </div>
        <el-form ref="ruleFormRef" :model="ruleForm" status-icon :rules="rules" label-width="0" class="login-form">
          <div class="login-title">
            <img src="../assets/image/logo.svg" alt="" />
            <p>{{ TUITranslateService.t("Login.免费体验") }}</p>
          </div>
          <el-form-item class="login-from-item" prop="userID">
            <el-input size="large" v-model="ruleForm.userID" :placeholder="TUITranslateService.t('Login.请输入userID')"
              class="input-with-select" :disabled="isLogin">
            </el-input>
          </el-form-item>
          <el-form-item prop="checked">
            <el-checkbox v-model="ruleForm.checked">
              <p class="checked-text">
                {{ TUITranslateService.t("Login.我已阅读并同意") }}
                <a @click="openDownloadLink(Link.privacy)">《{{
                  TUITranslateService.t(`Login.${Link.privacy.label}`)
                }}》</a>{{ TUITranslateService.t("Login.和") }}
                <a @click="openDownloadLink(Link.agreement)">《{{
                  TUITranslateService.t(`Login.${Link.agreement.label}`)
                }}》</a>
              </p>
            </el-checkbox>
          </el-form-item>
          <el-form-item class="login-btn">
            <button class="btn btn-primary" @click.prevent="submitForm(ruleFormRef)" v-if="isLogin">
              {{ TUITranslateService.t("Login.登录当前账号") }}
            </button>
            <button class="btn btn-primary" @click.prevent="submitForm(ruleFormRef)" v-else>
              {{ TUITranslateService.t("Login.登录") }}
            </button>
          </el-form-item>
          <el-form-item class="login-btn" v-if="isLogin">
            <button class="btn" @click.prevent="exitLogin">
              {{ $t("Login.切换其他账号") }}
            </button>
          </el-form-item>
          <footer class="login-form-footer">
            <a @click="openDownloadLink(Link.demo)">{{
              TUITranslateService.t(`Login.${Link.demo.label}`)
            }}</a>
            <a @click="openDownloadLink(Link.im)" v-if="!isH5">{{
              TUITranslateService.t(`Login.${Link.im.label}`)
            }}</a>
          </footer>
        </el-form>
      </div>
      <div class="login-main-middle" v-if="isPC">
        <div class="login-main-middle-box">
          <AdvList :list="qrList">
            <template #item="{ data }">
              <AdvListItem :item="data" @hoverEvent="openDownloadLink" />
            </template>
          </AdvList>
        </div>
      </div>
      <div class="login-main-footer" v-if="isPC">
        <div class="buttom-mask">
          <p class="buttom-mask-top">
            {{ TUITranslateService.t("Login.一分钟") }}
          </p>
          <p class="buttom-mask-under">
            {{ TUITranslateService.t("Login.改2行代码，1分钟跑通demo") }}
          </p>
        </div>
        <div class="buttom-mask">
          <p class="buttom-mask-top">10000+</p>
          <p class="buttom-mask-under">
            {{ TUITranslateService.t("Login.每月服务客户数超过10000家") }}
          </p>
        </div>
        <div class="buttom-mask">
          <p class="buttom-mask-top">99.99%</p>
          <p class="buttom-mask-under">
            {{ TUITranslateService.t("Login.消息收发成功率") }}
          </p>
        </div>
        <div class="buttom-mask">
          <p class="buttom-mask-top">
            {{ TUITranslateService.t("Login.10亿") }}+
          </p>
          <p class="buttom-mask-under">
            {{ TUITranslateService.t("Login.每月活跃用户数超过10亿") }}
          </p>
        </div>
      </div>
    </main>
    <footer class="login-footer" v-if="!isPC">
      <ul class="login-footer-list">
        <li class="login-footer-list-item" v-for="(item, index) in Link.advList" :key="index">
          <a @click="openDownloadLink(item)">
            <aside>
              <h1>{{ TUITranslateService.t(`Home.${item.label}`) }}</h1>
              <h1 v-if="item.subLabel" class="sub">
                {{ TUITranslateService.t(`Home.${item.subLabel}`) }}
              </h1>
            </aside>
            <span>{{TUITranslateService.t(`Home.${item.btnText}`)}}</span>
          </a>
        </li>
      </ul>
      <div class="login-footer-list-bottom">
        <div class="text-header">
          <i></i>
          <span>{{ TUITranslateService.t(`Login.更多客户端体验`) }}</span>
          <i></i>
        </div>
        <ul class="login-footer-list-bottom-image">
          <li class="platform" v-for="(item, index) in mobileList" :key="index" @click="handleJump(item)">
            <img :src="item.link" alt="" />
          </li>
        </ul>
      </div>
    </footer>
    <div class="mask" v-if="showMini" @click="showMini = false">
      <div class="mask-main">
        <img src="https://web.sdk.qcloud.com//im/assets/images/mini.png" alt="" />
        <span>{{
          TUITranslateService.t("Login.微信扫一扫，免费体验腾讯云 IM 小程序")
        }}</span>
        <span>{{
          TUITranslateService.t("Login.或者截图至相册，切换微信扫一扫识别体验")
        }}</span>
      </div>
    </div>
  </div>
</template>
<script lang="ts" setup>
import { ref } from "../TUIKit/adapter-vue";
import { TUILogin } from "@tencentcloud/tui-core";
import {
  TUIGlobal,
  TUITranslateService,
} from "@tencentcloud/chat-uikit-engine";
import { SDKAppID, secretKey } from "../main";
import router from "../router/index";
import { Message } from "element-ui";
import Header from "../components/Header.vue";
import AdvList from "../components/advList.vue";
import AdvListItem from "../components/advListItem.vue";
import { Link, qrList, mobileList } from "../utils/link";
import { genTestUserSig } from "../TUIKit";

const locale = ref("zh");
const isLogin = ref(false);
const showMini = ref(false);
const isH5 = ref(TUIGlobal.getPlatform() === "h5");
const isPC = ref(TUIGlobal.getPlatform() === "pc");

const ruleFormRef = ref();

const validateUserID = (rule: any, value: any, callback: any) => {
  if (!rule.required) {
    callback();
  } else if (!value) {
    callback(new Error(TUITranslateService.t("Login.请输入userID")));
  } else {
    callback();
  }
};

const validateChecked = (rule: any, value: any, callback: any) => {
  if (!value) {
    callback(new Error(TUITranslateService.t("Login.请先勾选用户协议")));
  } else {
    callback();
  }
};

const ruleForm = ref({
  checked: false,
  userID: "",
});

const rules = ref({
  checked: [{ required: true, validator: validateChecked, trigger: "change" }],
  userID: [{ required: true, validator: validateUserID, trigger: "blur" }],
});


if (localStorage.getItem("TUIKit-userInfo")) {
  const storgeUserInfo = localStorage.getItem("TUIKit-userInfo") || "";
  const userInfo = JSON.parse(storgeUserInfo);
  if (new Date(userInfo?.expire) > new Date()) {
    isLogin.value = true;
    ruleForm.value.userID = userInfo.userID || "";
  }
}

const openDownloadLink = (type: any) => {
  type.download && openFullPlatformLink(type.download);
  !type.download && type.url && openFullPlatformLink(type.url);
};

const openFullPlatformLink = (link: string) => {
  window.open(link);
};

const submitForm = (formEl: any) => {
  if (!formEl) return;
  formEl.validate((valid: any) => {
    if (valid) {
      const options = genTestUserSig({
        SDKAppID,
        secretKey,
        userID: ruleForm.value.userID,
      });
      const loginInfo = {
        SDKAppID,
        userID: ruleForm.value.userID,
        userSig: options.userSig,
        useUploadPlugin: true,
        useProfanityFilterPlugin: true
      };
      login(loginInfo);
    } else {
      return false;
    }
  });
};

const login = (loginInfo: any) => {
  TUILogin.login(loginInfo)
    .then((res: any) => {
      router.push({ path: "home" });
    })
    .catch((error: any) => {
      Message.error({
        message: error,
      });
    });
};

const exitLogin = async () => {
  localStorage.removeItem("TUIKit-userInfo");
  ruleForm.value.userID = "";
  isLogin.value = false;
};

const handleJump = (item: any) => {
  switch (item.type) {
    case "android":
      openFullPlatformLink(item.url);
      break;
    case "iphone":
      openFullPlatformLink(item.url);
      break;
    case "miniprogram":
      showMini.value = true;
      break;
  }
};
</script>

<style scoped lang="scss">
@import "../styles/login.scss";
@import "../styles/icon.scss";

.login-sale {
  font-weight: 400;
  color: #ffffff;
  background: url("../assets/image/adv-bg.svg") no-repeat;
  background-size: cover;
  background-position: center;
}

.btn-primary,
.btn {
  width: 100%;
}
</style>
