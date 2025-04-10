<script setup lang="ts">
import {
  ref,
  computed,
  onMounted,
  onUnmounted,
  watch,
  onWatcherCleanup,
} from 'vue';

const { isPip, pipPort } = defineProps<{
  isPiP: boolean;
  pipPort?: MessagePort;
}>();

const messagesFromMain = ref<string[]>([]);

watch(
  () => pipPort,
  (port) => {
    if (!port) return;
    port.onmessage = (event) => messagesFromMain.value.push(event.data);
    port.onmessageerror = (err) => {
      console.error('Error in message channel:', err);
    };

    onWatcherCleanup(() => {
      port.onmessage = null;
      port.onmessageerror = null;
      messagesFromMain.value = [];
    });
  },
  { immediate: true }
);

const sendMessageToMain = () => {
  if (pipPort) {
    pipPort.postMessage(`Hello from PIP. Count = ${count.value}`);
  }
};

const count = ref(0);
const currentTime = ref(new Date());

const formattedTime = computed(() => {
  return currentTime.value.toLocaleTimeString();
});

let timerInterval;
onMounted(() => {
  timerInterval = setInterval(() => {
    currentTime.value = new Date();
  }, 1000);
});

onUnmounted(() => {
  clearInterval(timerInterval);
});
</script>

<template>
  <div style="position: absolute; bottom: 0; right: 0">
    <div class="pip-component">
      <div class="controls">
        <button @click="count++">Count: {{ count }}</button>
      </div>
      <div class="time">Current Time: {{ formattedTime }}</div>

      <button v-if="isPiP" @click="sendMessageToMain">
        Send message from to main
      </button>

      <ul v-if="isPiP">
        <li v-for="message in messagesFromMain">{{ message }}</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.source-info {
  margin: 1em 0;
  font-size: 0.9em;
  color: #666;
}

.controls {
  margin: 1em 0;
}

.time {
  font-size: 0.9em;
  margin-top: 0.5em;
  font-weight: bold;
}
</style>
