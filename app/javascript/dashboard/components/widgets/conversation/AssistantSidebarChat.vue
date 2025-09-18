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
      default: 'assistente-ai', // corrigido para o nome exato da inbox
    },
  },
  computed: {
    ...mapGetters({
      allChats: 'getAllConversations',
      inboxes: 'inboxes/getAllInboxes',
      dashboardApps: 'dashboardApps/getRecords',
    }),
    assistantInbox() {
      // eslint-disable-next-line no-console
      console.log('Todas as inboxes:', this.inboxes);
      return this.inboxes.find(inbox => inbox.name === 'assistente-ai');
    },
    assistantChat() {
      // eslint-disable-next-line no-console
      console.log('Todas as conversas:', this.allChats);
      if (!this.assistantInbox) return null;
      
      const filtered = this.allChats.filter(chat => 
        chat.inbox_id === this.assistantInbox.id
      );
      // eslint-disable-next-line no-console
      console.log('Conversas filtradas para assistente-ai:', filtered);
      
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
    <div v-else class="p-4 text-xs text-center text-n-600">
      Nenhuma conversa encontrada para a inbox <b>{{ inboxName }}</b>.<br>
      <span class="block mt-2">Verifique se o nome da inbox está correto ou se há conversas disponíveis.</span>
      <div class="mt-2 text-left">
        <b>Inboxes encontradas:</b>
        <ul>
          <li v-for="chat in allChats" :key="chat.id">
            {{ chat.inbox && chat.inbox.name }} (ID: {{ chat.inbox && chat.inbox.id }})
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Nenhum estilo customizado, apenas Tailwind */
</style>
