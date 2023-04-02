<template>
  <div class="chat">
    <div v-if="!joined" class="welcome-div">
      <h1>Welcome!</h1>
      <div class="form-div">
        <form @submit.prevent="join">
          <label>Enter your name: </label>
          <input v-model="name" />
          <button type="submit">Join! 🚀</button>
        </form>
      </div>
      <h2>[#{{ name }}]</h2>
    </div>
    <div v-else class="chat-container">
      <div class="messages-container">
        <div v-for="message in messages">
          [#{{ message.name }}]: {{ message.text }}
        </div>
      </div>
      <div v-if="typingDisplay">{{ typingDisplay }}</div>
      <hr />
      <div class="typing-area">
        <form @submit.prevent="sendMessage">
          <label>Message:</label>
          <input v-model="messageText" @input="emitTyping" />
          <button type="submit">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { io } from "socket.io-client";
import { onBeforeMount, ref } from "vue";
const socket = io("http://localhost:3001");

const messages = ref([]);
const messageText = ref("");
const joined = ref(false);
const name = ref("");
const typingDisplay = "";

onBeforeMount(() => {
  socket.emit("findAllMessages", {}, (response) => {
    messages.value = response;
  });

  socket.on("message", (message) => {
    messages.value.push(message);
  });

  socket.on("typing", ({ name, isTyping }) => {
    if (isTyping) {
      typingDisplay.value = `${name} is typing...`;
    } else {
      typingDisplay.value = "";
    }
  });
});

const join = () => {
  socket.emit("join", { name: name.value }, () => {
    joined.value = true;
  });
};

const sendMessage = () => {
  socket.emit(
    "createMessage",
    { name: name.value, text: messageText.value },
    () => {
      messageText.value = "";
    }
  );
};

let timeout;
const emitTyping = () => {
  socket.emit("typing", { isTyping: true });

  timeout = setTimeout(() => {
    socket.emit("typing", { isTyping: false });
  }, 2000);
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
}

.welcome-div {
  font-family: "Roboto Mono", monospace;
  display: flex;
  flex-direction: column;
}

.chat {
  width: 100vw;
  height: 100vh;
  background-image: linear-gradient(
    to right top,
    #051937,
    #004d7a,
    #008793,
    #00bf72,
    #a8eb12
  );
}

.welcome-div h1 {
  color: white;
  margin-top: 2vh;
  font-size: 4rem;
  text-align: center;
}

.welcome-div .form-div {
  width: 30vw;
  margin: 5vh auto;
}

.welcome-div form {
  height: 50vh;
  border: 1px solid white;
  border-radius: 2rem;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 90%;
  margin: 0 auto;
}

.welcome-div form label {
  font-size: 2rem;
  color: white;
}

.welcome-div form input {
  margin: 2vh;
  padding: 1vh 1vw;
  font-size: 1.3rem;
  border-radius: 1rem;
  font-family: "Roboto Mono", monospace;
}

.welcome-div form button {
  font-size: 1.3rem;
  padding: 1vh 1vw;
  border-radius: 1rem;
}

.welcome-div form button:hover {
  cursor: pointer;
}

.welcome-div h2 {
  text-align: center;
  color: white;
  font-size: 3rem;
}
</style>
