<script>
import { mapGetters } from 'vuex';
import ConversationHeader from './ConversationHeader.vue';
import DashboardAppFrame from '../DashboardApp/Frame.vue';
import EmptyState from './EmptyState/EmptyState.vue';
import MessagesView from './MessagesView.vue';

export default {
  name: 'AssistantSidebarChat',
  components: {
    ConversationHeader,
    DashboardAppFrame,
    EmptyState,
    MessagesView,
  },
  props: {
    inboxName: {
      type: String,
      default: 'assistente-ia',
    },
  },
  computed: {
    ...mapGetters({
      allChats: 'getAllConversations',
      dashboardApps: 'dashboardApps/getRecords',
    }),
    assistantChat() {
      // Filtra a conversa da inbox "assistente-ia" (API)
      // Se houver mais de uma, pega a mais recente (pode ser ajustado conforme regra de negócio)
      const filtered = this.allChats.filter(chat => chat.inbox && chat.inbox.name === this.inboxName);
      // Ordena por updated_at (ou outro critério, se necessário)
      return filtered.length ? filtered.sort((a, b) => new Date(b.updated_at) - new Date(a.updated_at))[0] : null;
    },
    dashboardAppTabs() {
      return [
        {
          key: 'messages',
          index: 0,
          name: this.$t('CONVERSATION.DASHBOARD_APP_TAB_MESSAGES'),
        },
        ...this.dashboardApps.map((dashboardApp, index) => ({
          key: `dashboard-${dashboardApp.id}`,
          index: index + 1,
          name: dashboardApp.title,
        })),
      ];
    },
  },
};
</script>

<template>
  <div class="h-full w-full flex flex-col">
    <template v-if="assistantChat">
      <ConversationHeader :chat="assistantChat" />
      <MessagesView :chat="assistantChat" :current-chat="assistantChat" />
    </template>
    <EmptyState v-else />
  </div>
</template>

<style scoped>
/* Nenhum estilo customizado, apenas Tailwind */
</style>
