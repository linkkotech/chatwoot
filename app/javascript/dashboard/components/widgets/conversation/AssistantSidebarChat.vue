<script>
import { mapGetters } from 'vuex';
import AssistantHeader from './AssistantHeader.vue';
import EmptyState from './EmptyState/EmptyState.vue';
import MessagesView from './MessagesView.vue';

export default {
  name: 'AssistantSidebarChat',
  components: {
    AssistantHeader,
    EmptyState,
    MessagesView,
  },
  props: {
    inboxName: {
      type: String,
      default: 'assistente-ai', // corrigido para o nome exato da inbox
    },
  },
  props: {
    currentUserId: {
      type: Number,
      required: true,
    },
    activeConversationId: {
      type: Number,
      default: null,
    },
  },
  computed: {
    ...mapGetters({
      allChats: 'getAllConversations',
      inboxes: 'inboxes/getAllInboxes',
      currentUser: 'getCurrentUser',
    }),
    assistantInbox() {
      return this.inboxes.find(inbox => inbox.name === 'assistente-ai');
    },
    assistantChat() {
      if (!this.assistantInbox) return null;
      
      // Filtra conversas:
      // 1. Da inbox assistente-ai
      // 2. Do usuário atual
      // 3. Com metadados referenciando a conversa ativa (se houver)
      const filtered = this.allChats.filter(chat => {
        const isAssistantInbox = chat.inbox_id === this.assistantInbox.id;
        const isCurrentUser = chat.assignee_id === this.currentUserId;
        const hasActiveConversationContext = this.activeConversationId 
          ? chat.custom_attributes?.parent_conversation_id === this.activeConversationId
          : true;
        
        return isAssistantInbox && isCurrentUser && hasActiveConversationContext;
      });
      
      // Retorna a conversa mais recente que atende aos critérios
      return filtered.length 
        ? filtered.sort((a, b) => new Date(b.updated_at) - new Date(a.updated_at))[0] 
        : null;
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
    <AssistantHeader />
    <div class="flex-1 overflow-hidden">
      <template v-if="assistantChat">
        <MessagesView 
          :chat="assistantChat" 
          :current-chat="assistantChat" 
          class="h-full"
        />
      </template>
      <div v-else class="p-4 text-xs text-center text-slate-400">
        <p>Nenhuma conversa ativa com o assistente.</p>
        <p v-if="!assistantInbox" class="mt-2">
          A caixa de entrada "assistente-ai" não está configurada.
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Nenhum estilo customizado, apenas Tailwind */
</style>
