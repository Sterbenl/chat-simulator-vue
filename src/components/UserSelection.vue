<template>
  <div class="user-selection">
    <h2>Выберите пользователя</h2>
    <ul>
      <li
        v-for="user in users"
        :key="user.id"
        @click="$emit('user-selected', user)"
        class="user-selection-item"
      >
        <span>{{ user.name }}</span>
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
  name: 'UserSelection',
  props: {
    users: {
      type: Array as () => User[],
      required: true,
    },
  },
});
</script>
