<script setup lang="ts">
import {
  ref,
  shallowRef,
  onMounted,
  onBeforeUnmount,
  computed,
  nextTick,
} from 'vue';
import PipComponent from './components/PipComponent.vue';

const channel = shallowRef<MessageChannel | null>(null);
const mainPort = shallowRef<MessagePort | null>(null);
const pipPort = shallowRef<MessagePort | null>(null);
const pipWindow = shallowRef<PictureInPictureWindow | null>(null);

const isPiP = computed(() => pipWindow.value !== null);

const messagesFromPiP = ref<string[]>([]);

// Open PiP window
const openPiP = async () => {
  if (!document.pictureInPictureEnabled) {
    alert('Picture-in-Picture API is not supported in your browser');
    return;
  }

  try {
    // Create a message channel for communication
    channel.value = new MessageChannel();
    mainPort.value = channel.value.port1;
    pipPort.value = channel.value.port2;

    // Request the PiP window
    pipWindow.value = await documentPictureInPicture.requestWindow({
      width: 400,
      height: 300,
    });

    // Set PiP the window title
    pipWindow.value.document.title = 'PiP';

    // Copy style sheets over from the main document
    [...document.styleSheets].forEach((styleSheet) => {
      try {
        const cssRules = [...styleSheet.cssRules]
          .map((rule) => rule.cssText)
          .join('');
        const style = document.createElement('style');

        style.textContent = cssRules;
        pipWindow.value.document.head.appendChild(style);
      } catch (e) {
        const link = document.createElement('link');

        link.rel = 'stylesheet';
        link.type = styleSheet.type;
        link.media = styleSheet.media;
        link.href = styleSheet.href;
        pipWindow.value.document.head.appendChild(link);
      }
    });

    // Set up message handling
    mainPort.value.onmessage = (event) => {
      messagesFromPiP.value.push(event.data);
    };

    // Handle PiP window close
    pipWindow.value.addEventListener('pagehide', closePiP);
  } catch (error) {
    console.error('Failed to open PiP window:', error);
    alert(`Failed to open PiP: ${error.message}`);
  }
};

// Close PiP window
const closePiP = () => {
  if (pipWindow.value) {
    pipWindow.value.close();
    pipWindow.value = null;
    channel.value = null;
    mainPort.value = null;
    pipPort.value = null;
    messagesFromPiP.value = [];
    messageCount.value = 0;
  }
};

const messageCount = ref(0);

const sendMessageToPIP = () => {
  if (mainPort.value) {
    mainPort.value.postMessage(`Hello from main ${++messageCount.value}`);
  }
};

// Clean up resources when component is unmounted
onBeforeUnmount(closePiP);
</script>

<template>
  <div class="container">
    <h1>Vue Teleport to PiP Demo</h1>

    <button @click="openPiP" :disabled="pipWindow">
      Open Picture-in-Picture
    </button>

    <button @click="closePiP" :disabled="!pipWindow">
      Close Picture-in-Picture
    </button>

    <div class="main-content">
      <h2>Main Window Content</h2>
      <p>This content stays in the main window.</p>
    </div>

    <button v-if="isPiP" @click="sendMessageToPIP">Send message to PIP</button>

    <ul v-if="isPiP">
      <li v-for="message in messagesFromPiP">{{ message }}</li>
    </ul>

    <!-- Component to teleport to PiP and back -->
    <Teleport :to="pipWindow ? pipWindow.document.body : 'body'">
      <PipComponent :pipPort :isPiP />
    </Teleport>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.main-content {
  margin-top: 2em;
  padding: 1em;
  border: 1px solid #ddd;
  border-radius: 8px;
  background-color: #f9f9f9;
  max-width: 400px;
}
</style>
