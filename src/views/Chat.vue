<template>
  <div class="h-screen overflow-y-hidden relative bg-transparent">
    <div class="absolute top-0 left-0 w-full h-full -z-20" :class="classes" />
    <div class="absolute top-0 left-0 w-full h-full bg-cover bg-center -z-10" :class="bgClass" :style="style" />

    <Navbar backroute="/chats" class="mb-0">
      <template #backbutton>
        <Button variant="ghost" size="icon" class="text-primary-600 !bg-transparent">
          <ChevronLeft class="w-6 h-6" />
        </Button>
      </template>
      <template #title>
        <div class="flex items-center flex-row gap-3">
          <Avatar class="bg-neutral-950 h-10 w-10 rounded-full flex items-center justify-center">
              <AvatarImage src="https://github.com/radix-vue.png" alt="@radix-vue" />
              <AvatarFallback><i v-html="fallbackImage" /></AvatarFallback>
          </Avatar>
          <p class="text-xl font-medium">BubbleChat</p>
        </div>
      </template>
    </Navbar>

    <div
      ref="scrollContainer"
      class="flex flex-col items-center justify-start overflow-y-scroll transition-all duration-300"
      :class="isEmojiPickerOpen === false ? 'h-[calc(100dvh_-_143px)]' : 'h-[calc(100dvh_-_475px)]'"
    >
      <div class="max-w-md flex flex-col container px-4 mx-auto gap-y-3 mt-3">
        <!-- Add day for each  -->
        <span v-for="(message, index) in messages" :key="message.id" class="text-xs text-gray-500 dark:text-gray-400 last:mb-3">
          <p class="text-center" v-if="checkLastTime(index)" v-html="formattedDate(message.time)"></p>
          <Message :key="message.id" :message="message" />
        </span>
      </div>
    </div>
    <MessageInput @openEmojiPicker="(e) => openEmojiPicker(e)" />
  </div>
</template>

<script setup>
  import api from '@/api/index.js';
  import Navbar from '@/components/Navbar.vue';
  import Message from '@/components/Message.vue';
  import MessageInput from '@/components/MessageInput.vue';
  import Button from '@/components/ui/button/Button.vue';
  import { ChevronLeft, UserRound } from 'lucide-vue-next'
  import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar'
  import fallbackImage from '@/assets/user-fallback.svg?raw'
</script>

<script>
export default {
  name: 'ChatList',
  data() {
    return {
      messages: [],
      isEmojiPickerOpen: false,
    }
  },
  async created() {
    // Start polling when the component is created
    await this.startPolling();
  },
  methods: {
    async startPolling() {
      // Fetch data initially
      await this.getMessages();
      this.scrollToBottom();
    },
    async getMessages() {
      const token = JSON.parse(localStorage.getItem('token'));
      const result = await api.getmessages({token: token.token});
      this.messages = result.messages ?? 'No messages';
      setTimeout(this.getMessages, 5000); // Poll every 5 seconds
    },
    checkLastTime(index) {
      return index === 0 || this.formattedDate(this.messages[index].time) !== this.formattedDate(this.messages[index - 1].time);
    },
    formattedDate(time) {
        const dateParts = time.split('_');
        const date = new Date(dateParts[0].replace(/-/g, '/') + ' ' + dateParts[1].replace(/-/g, ':'));
        const today = new Date();
        const yesterday = new Date(today);
        yesterday.setDate(yesterday.getDate() - 1);

        // If the date is today, display only hour and minute
        if (date.toDateString() === today.toDateString()) {
          const options = { hour: 'numeric', minute: 'numeric' };
          return date.toLocaleTimeString(navigator.language, options);
        }

        // If the date is yesterday, display "yesterday"
        if (date.toDateString() === yesterday.toDateString()) {
          return 'Yesterday';
        }

        // If the date is within the last week, display weekday
        const diffInDays = Math.ceil((today - date) / (1000 * 60 * 60 * 24));
        if (diffInDays <= 7) {
          const options = { weekday: 'long', hour: 'numeric', minute: 'numeric' };
          return date.toLocaleDateString(navigator.language, options);
        }

        // Otherwise, display the full date
        const options = { year: '2-digit', month: '2-digit', day: '2-digit' };
        return date.toLocaleDateString(navigator.language, options);
    },
    scrollToBottom() {
      // Scroll to the bottom of the scroll container
      this.$refs.scrollContainer.scrollTop = this.$refs.scrollContainer.scrollHeight;
    },
    openEmojiPicker(e) {
      this.isEmojiPickerOpen = e;
      // scroll after 300ms
      setTimeout(() => this.scrollToBottom(), 400);
    }
  },
  computed: {
    style() {
      const imageUrl = JSON.parse(localStorage.getItem('imageUrl'));
      return `background-image: url(${imageUrl.backgroundImage})`;
    },
    classes() {
      const imageUrl = JSON.parse(localStorage.getItem('imageUrl'));
      return imageUrl.type === 'pattern' ? 'bg-opacity-10 bg-yellow-800 dark:bg-neutral-950' : '';
    },
    bgClass() {
      const imageUrl = JSON.parse(localStorage.getItem('imageUrl'));
      return imageUrl.type === 'pattern' ? 'opacity-50 dark:opacity-10 invert' : '';
    },
  },
}
</script>