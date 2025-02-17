<template>
  <div class='p-4 max-w-lg mx-auto'>
    <h1 class="text-4xl font-extrabold tracking-wide text-white-800 mb-10 w-96">Talk to a Celebrity Bot!</h1>
    <div class='flex justify-center gap-5'>
      <div
        v-for="artist in artists" 
        :key="artist.id"
        class='flex justify-center mb-4'
      >
        <img 
          :src="artist.img" 
          :alt="artist.name"
          class="w-20 h-20 rounded-full shadow-lg object-cover aspect-square cursor-pointer transition-all duration-200 ease-in-out" 
          :class="{
            'brightness-70 border-0 scale-90': artist.name !== selectedArtist.name,
            'border-4 border-gray-300 scale-105': artist.name === selectedArtist.name
          }"
          @click="{selectedArtist = artist; }"
        />
      </div>
    </div>
    <div class='w-96 h-96 overflow-y-auto border p-2 mb-2'>
      <div
        v-for="(msg, index) in messages.filter(m => m.sender !== 'system')"
        :key='index'
        :class="{'text-right': msg.sender === 'user', 'text-left': msg.sender === 'bot'}"
      >
        <p class='m-2 p-2 inline-block rounded' :class="{'bg-blue-200': msg.sender === 'user', 'bg-gray-200': msg.sender === 'bot'}">
          {{ msg.text }}
        </p>
      </div>
    </div>
    <input v-model='userInput' @keyup.enter='sendMessage' class='border p-2 w-full' placeholder='Type a message...' />
  </div>
</template>

<script>
import { ref } from 'vue';
import axios from 'axios';

export default {
  data() {
    const artists = [
      {
        name: 'Kanye West',
        img: 'https://www.usatoday.com/gcdn/presto/2019/03/07/USAT/71d24511-e504-40e1-95eb-180f883eeb81-LL_MW_KanyeWest_010319.JPG?width=300&height=450&fit=crop&format=pjpg&auto=webp',
        system_message: "You are  Kanye West talking to a fan. Act like Kanye West and be knowledgeable about your songs, exes, and your current wife, Bianca Censori.",
        bot_message: "'Sup, 'lil fella. My name's Kanye West! Talk to me.",
      },
      {
        name: 'Taylor Swift',
        img: 'data:image/jpeg;base64,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',
        system_message: "You are Taylor Swift talking to a fan. Act like Taylor Swift and be knowledgeable about your songs, exes, and your current boyfriend.",
        bot_message: "Hello! I'm Taylor Swift.",
      },
      {
        name: 'Lisa',
        img: 'data:image/jpeg;base64,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',
        system_message: "You are Lalisa Manoban talking to a fan. Act like Lisa and be knowledgeable about your songs and your career history. Only respond in Korean even if your fan talks in English. Follow the Korean with an English translation enclosed in parentheses.",
        bot_message: "안녕하세요! (Hello!)",
      },
    ];

    const selectedArtist = artists[0];

    return {
      apiKey: import.meta.env.VITE_API_KEY,
      artists,
      selectedArtist,
      messages: [
        {text: selectedArtist.system_message, sender: 'system'},
        {text: selectedArtist.bot_message, sender: 'bot'},
      ],
      userInput: '',
    };
  },

  watch: {
    selectedArtist() {
    this.messages =  [
      {text: this.selectedArtist.system_message, sender: 'system'},
      {text: this.selectedArtist.bot_message, sender: 'bot'},
    ];
      console.log(this.selectedArtist);
    },
  },

  methods: {
    async sendMessage() {
      if (!this.userInput.trim()) return;

      this.messages.push({ text: this.userInput, sender: 'user' });
      const userMessage = this.userInput;
      this.userInput = '';

      console.log('messages', this.messages);
      console.log('messages mapped', this.messages.map(m => ({ role: m.sender === 'user' ? 'user' : (m.sender === 'bot' ? 'assistant' : 'system'), content: m.text })));

      try {
        const response = await axios.post('https://api.openai.com/v1/chat/completions', {
          model: 'gpt-3.5-turbo',
          messages: this.messages.map(m => ({ role: m.sender === 'user' ? 'user' : (m.sender === 'bot' ? 'assistant' : 'system'), content: m.text }))
        }, {
          headers: { Authorization: `Bearer ${this.apiKey}`, 'Content-Type': 'application/json' }
        });

        this.messages.push({ text: response.data.choices[0].message.content, sender: 'bot' });
      } catch (error) {
        console.error('Error:', error);
      }
    }
  },

};
</script>

<style scoped>
.text-right {
  text-align: right;
}
.text-left {
  text-align: left;
}
.bg-blue-200 {
  background-color: #0d386c;
}
.bg-gray-200 {
  background-color: #383838;
}
</style>
