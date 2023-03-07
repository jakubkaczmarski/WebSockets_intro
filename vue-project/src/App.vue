<script setup>
import HelloWorld from './components/HelloWorld.vue'
import TheWelcome from './components/TheWelcome.vue'
import {io} from 'socket.io-client';
import {onBeforeMount, ref} from 'vue';

const messages = ref([]);
const messageText = ref('');
const joined = ref(false);
const name = ref('');
const isTyping = ref(false);
const typingDisplay = ref('');
onBeforeMount(() =>
{
  socket.emit('findAllMessages', {}, (response) =>
  {
    messages.value = response;
    
  });
  socket.on('message', (message) => 
  {
    
    messages.value.push(message);
    console.log(message);
  });

  socket.on('typing', ({name, isTyping}) => 
  {
    if(isTyping)
    {
      
      typingDisplay.value = `${name} is typing...`;

    }else{
      typingDisplay.value = '';
    }
  })

})
let timeout;
const emitTyping = () =>
{
  socket.emit('typing', {isTyping: true});
  timeout = setTimeout(() => 
  {
    socket.emit('typing', {isTyping: false});
  }, 2000);
}
const join = () =>
{
  socket.emit('join', {name: name.value}, () => 
  {
    joined.value = true;
  })
}

const sendMessage = () => 
{
  socket.emit('createMessage', {name: name.value, text: messageText.value}, () => 
  {
    messageText.value = '';
  })
}


const socket = io('http://localhost:3001');
</script>

<template>
  <div class="chat"> 
  <div v-if="!joined">
    <form @submit.prevent="join">
    <label> What's your name? </label>
    <input v-model="name" />
    <button type="submit"> Send </button>
    </form>
    
  </div>
  <div class="chat-container" v-else>
  <div v-for="message in messages">
  [{{message.name}}] : {{message.text}}
  </div>
  
  <div class="message-input">
   <form @submit.prevent="sendMessage">
    <label> Message: </label>
    <input v-model="messageText" @input="emitTyping" />
    <button type="submit"> Send </button>
    </form>

  </div>

  <div v-if="typingDisplay"> 
      {{typingDisplay}}
  </div> 
  </div>
  </div>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
