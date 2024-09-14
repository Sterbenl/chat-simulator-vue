<template>
  <div class="chat-list">
    <h2>Чаты</h2>
    <ul>
      <li
        v-for="user in sortedUsers"
        :key="user.id"
        @click="selectChat(user)"
        class="chat-list-item"
      >
        <span>{{ user.name }}</span>
        <!-- Показываем уведомления только у получателя -->
        <div v-if="newMessages && newMessages[user.id]" class="unread-count">
          {{ newMessages[user.id] }}
        </div>
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

interface User {
  id: string;
  name: string;
}

export default defineComponent({
  name: 'ChatList',
  props: {
    users: {
      type: Array as () => User[],
      required: true,
    },
    activeUserId: {
      type: String,
      required: true,
    },
    newMessages: {
      type: Object,
      required: true,
    },
  },
  emits: ['chat-selected'],
  computed: {
    sortedUsers() {
      return this.users
        .filter((user: User) => user.id !== this.activeUserId)
        .sort((a: User, b: User) => {
          const newMessagesA = this.newMessages[a.id] || 0;
          const newMessagesB = this.newMessages[b.id] || 0;
          return newMessagesB - newMessagesA;
        });
    },
  },
  methods: {
    selectChat(user: User) {
      this.$emit('chat-selected', user);
    },
  },
});
</script>
