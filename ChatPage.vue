<template>
  <q-page padding>
    <div class="main">
      <div class="chatLog">
        <div v-for="(item, index) in reversedLog" :key="index">
          <q-chat-message 
            :bg-color ="item.role === 'user' ? 'grey' : 'primary'"
            :text-color ="item.role === 'user' ? 'black' : 'white'"
            :avatar="'../assets/pLogo.png'"
            :name="item.role === 'user' ? 'You' : 'Parknshop Bot'"
            :text="[item.content]"
            :sent="item.role === 'user'"
            
          />
          <q-spinner-dots size="2rem" />
        </div>
      </div>
      <div class="form">
        <div class="input-container">
          <q-input
            v-model="input"
            label="Type something.."
            outlined
            style="width: 80%;"
            @keyup.enter="submitForm"
          ></q-input>
          <div class="button-container">
            <q-btn
              v-if="!isLoading"
              label="Submit"
              type="submit"
              color="primary"
              @click="generateResponse"
            ></q-btn>
            <q-btn
              v-else
              label="Loading..."
              type="submit"
              color="primary"
              disable
            ></q-btn>
          </div>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from 'vue';
import { Configuration, OpenAIApi } from 'openai';

export default defineComponent({
  name: 'ChatPage',
  data() {
    return {
      OPENAI_API_KEY: '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',
      input: '',
      response: '',
      log: [],
      messages: [],
      isLoading: false,
    };
  },
  computed: {
    reversedLog() {
      return this.log.slice().reverse();
    },
  },
  methods: {
    async completionCall(input) {
      this.messages.push({ role: 'user', content: input });

      const configuration = new Configuration({
        apiKey: process.env.OPENAI_API_KEY || this.OPENAI_API_KEY, // Use environment variable or secret management solution
      });

      const openai = new OpenAIApi(configuration);

      this.isLoading = true;

      const completion = await openai.createChatCompletion({
        model: 'gpt-3.5-turbo',
        messages: this.messages,
      });

      this.isLoading = false;

      this.response = completion.data.choices[0].message.content;
      console.log(completion.data.choices[0].message.content);
    },
    async generateResponse() {
      const userMessage = {
        role: 'user',
        content: this.input,
      };

      this.log.push(userMessage);

      await this.completionCall(this.input).then(() => {
        const botMessage = {
          role: 'bot',
          content: this.response,
        };

        this.log.push(botMessage);

        this.input = '';
        this.response = '';
      });
    },
    submitForm() {
      if (this.input.trim() !== '') {
        this.isLoading = true;
        this.generateResponse();
        this.input = '';
      }
    },
  },
});
</script>

<style>
.main {
  max-width: 1200px;
  margin: 0 auto;
}

.form {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  margin-top: 16px;
}

.input-container {
  display: flex;
  align-items: center;
  width: 95%;
  position: absolute;
  bottom: 1%;
}

.button-container {
  margin-left: 10px;
}

.chatLog {
  display: flex;
  flex-direction: column-reverse;
  max-height: 75vh; /* Adjust the maximum height as needed */
  overflow: auto;
}
</style>