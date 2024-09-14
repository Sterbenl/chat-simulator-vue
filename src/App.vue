<template>
  <div id="app">
    <div v-if="!activeUser">
      <UserSelection :users="users" @user-selected="onUserSelected" />
    </div>
    <div v-else>
      <div class="chat-container">
        <ChatList
          :users="users"
          :newMessages="newMessages[activeUser.id]"
          :activeUserId="activeUser.id"
          @chat-selected="onChatSelected"
        />
        <ChatWindow
          v-if="chattingWith"
          :activeUser="activeUser"
          :chattingWith="chattingWith"
          :messages="filteredMessages"
          @send-message="sendMessage"
        />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import UserSelection from './components/UserSelection.vue';
import ChatList from './components/ChatList.vue';
import ChatWindow from './components/ChatWindow.vue';

interface User {
  id: string;
  name: string;
}

interface Message {
  id: string;
  senderId: string;
  receiverId: string;
  content: string;
  timestamp: number;
}

interface NewMessages {
  [userId: string]: {
    [chatWithUserId: string]: number;
  };
}

export default defineComponent({
  name: 'App',
  components: {
    UserSelection,
    ChatList,
    ChatWindow,
  },
  data() {
    return {
      users: [
        { id: '1', name: 'Алиса' },
        { id: '2', name: 'Влад' },
        { id: '3', name: 'Ксюша' },
        { id: '4', name: 'Данил' },

      ] as User[],
      activeUser: null as User | null,
      chattingWith: null as User | null,
      messages: [] as Message[],
      newMessages: {} as NewMessages, // структура хранит уведомления по пользователю
    };
  },
  created() {
    this.loadMessages();
    window.addEventListener('storage', this.syncMessagesAcrossTabs);
  },
  beforeUnmount() {
    window.removeEventListener('storage', this.syncMessagesAcrossTabs);
  },
  computed: {
    filteredMessages() {
      if (this.activeUser && this.chattingWith) {
        return this.messages.filter(
          (msg) =>
            (msg.senderId === this.activeUser!.id && msg.receiverId === this.chattingWith!.id) ||
            (msg.senderId === this.chattingWith!.id && msg.receiverId === this.activeUser!.id)
        );
      }
      return [];
    },
  },
  methods: {
    saveMessages() {
      localStorage.setItem('messages', JSON.stringify(this.messages));
      localStorage.setItem('newMessages', JSON.stringify(this.newMessages));
    },

  loadMessages() {
  const storedMessages = localStorage.getItem('messages');
  const storedNewMessages = localStorage.getItem('newMessages');
  
  if (storedMessages) {
    this.messages = JSON.parse(storedMessages);
  }

  if (storedNewMessages) {
    this.newMessages = JSON.parse(storedNewMessages);
  } else {
    // Инициализируем newMessages для всех пользователей, если данных нет
    this.users.forEach(user => {
      if (!this.newMessages[user.id]) {
        this.newMessages[user.id] = {};
      }
    });
  }
},


    syncMessagesAcrossTabs(event: StorageEvent) {
      if (event.key === 'messages') {
        const updatedMessages = localStorage.getItem('messages');
        if (updatedMessages) {
          this.messages = JSON.parse(updatedMessages);
          this.forceUpdateChat();
        }
      }
      if (event.key === 'newMessages') {
        const updatedNewMessages = localStorage.getItem('newMessages');
        if (updatedNewMessages) {
          this.newMessages = JSON.parse(updatedNewMessages);
        }
      }
    },

    onUserSelected(user: User) {
      this.activeUser = user;
    },

    onChatSelected(user: User) {
      this.chattingWith = user;
      this.resetNewMessages(user.id);
    },

    sendMessage(content: string) {
      if (this.activeUser && this.chattingWith) {
        const newMessage: Message = {
          id: Date.now().toString(),
          senderId: this.activeUser.id,
          receiverId: this.chattingWith.id,
          content,
          timestamp: Date.now(),
        };
        this.messages.push(newMessage);

        // Сохраняем сообщения после их добавления
        this.saveMessages();

        // Увеличиваем счётчик только для получателя
        this.incrementNewMessages(this.chattingWith.id, this.activeUser.id);

        this.forceUpdateChat(); // Принудительно обновляем чат
      }
    },

    forceUpdateChat() {
      this.messages = [...this.messages];
    },

    // Увеличиваем счётчик новых сообщений только для конкретного получателя
    incrementNewMessages(receiverId: string, senderId: string) {
      if (!this.newMessages[receiverId]) {
        this.newMessages[receiverId] = {};
      }
      if (!this.newMessages[receiverId][senderId]) {
        this.newMessages[receiverId][senderId] = 0;
      }
      this.newMessages[receiverId][senderId]++;
      this.saveMessages();
    },

    resetNewMessages(receiverId: string) {
      if (this.newMessages[this.activeUser!.id] && this.newMessages[this.activeUser!.id][receiverId]) {
        this.newMessages[this.activeUser!.id][receiverId] = 0;
        this.saveMessages();
      }
    },
  },
});
</script>
