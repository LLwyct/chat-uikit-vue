<template>
  <div class="tui-conversation" @click="handleClickConv">
    <ConversationHeader ref="headerRef" />
    <ConversationNetwork />
    <ConversationList
      @handleSwitchConversation="handleSwitchConversation"
    />
  </div>
</template>
<script lang="ts" setup>
import {
  TUIStore,
  StoreName,
} from "@tencentcloud/chat-uikit-engine";
import { ref } from "../../adapter-vue";
import ConversationList from "./conversation-list/index.vue";
import ConversationHeader from "./conversation-header/index.vue";
import ConversationNetwork from "./conversation-network/index.vue";


const emits = defineEmits(["handleSwitchConversation"]);

const totalUnreadCount = ref(0);
const headerRef = ref<HTMLElement | undefined>();

TUIStore.watch(StoreName.CONV, {
  totalUnreadCount: (count: number) => {
    totalUnreadCount.value = count;
  },
});

const handleSwitchConversation = (conversationID: string) => {
  emits("handleSwitchConversation", conversationID);
};

const handleClickConv = () => {
  headerRef?.value?.closeChildren();
}

</script>

<style lang="scss" scoped src="./style/index.scss"></style>
