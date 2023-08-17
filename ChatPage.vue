<template>
  <q-page padding>
    <div class="main">
      <div class="chatLog">
        <div v-for="(item, index) in reversedLog" :key="index">
          <q-chat-message
            :bg-color="item.role === 'user' ? 'grey-3' : 'primary'"
            :text-color="item.role === 'user' ? 'black' : 'white'"
            :avatar="
              item.role === 'user'
                ? 'src/assets/customerIcon.png'
                : 'src/assets/pLogo.png'
            "
            :name="item.role === 'user' ? 'You' : 'Parknshop Bot'"
            :text="[item.content]"
            :sent="item.role === 'user'"
          />
          <div v-if="isLoading && index === 0 && item.role === 'user'">
            <q-spinner-dots color="primary" size="5vh" class="spinner-dot" />
          </div>
        </div>
      </div>
      <div class="form">
        <div class="input-container">
          <q-input
            v-model="input"
            label="Ask something.."
            outlined
            style="width: 80%"
            :disable="isLoading || !isBotMessageVisible"
            @keyup.enter="submitForm"
          ></q-input>
          <div class="button-container">
            <q-btn
              v-if="!isLoading && isBotMessageVisible"
              icon="send"
              color="primary"
              @click="generateResponse"
            ></q-btn>
            <q-btn
              v-else-if="isLoading && !isBotMessageVisible"
              icon="stop_circle"
              color="red"
              @click="stopLoading"
            ></q-btn>
          </div>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";
import { Configuration, OpenAIApi } from "openai";

export default defineComponent({
  name: "ChatPage",
  data() {
    return {
      OPENAI_API_KEY: "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",
      input: "",
      response: "",
      log: [],
      messages: [],
      isLoading: false,
      isBotMessageVisible: true,
    };
  },
  computed: {
    reversedLog() {
      return this.log.slice().reverse();
    },
  },
  methods: {
    async completionCall(input) {
      this.messages.push({ role: "user", content: input });
      const configuration = new Configuration({
        apiKey: process.env.OPENAI_API_KEY || this.OPENAI_API_KEY,
      });
      const openai = new OpenAIApi(configuration);

      this.isLoading = true;
      this.isBotMessageVisible = false;

      try {
        const completion = await openai.createChatCompletion({
          model: "gpt-3.5-turbo-16k",
          messages: this.messages,
        });

        // Check if loading was stopped while the API call was in progress
        if (!this.isLoading) {
          return;
        }

        this.response = completion.data.choices[0].message.content;
        console.log(completion.data.choices[0].message.content);
      } catch (error) {
        // Handle error if necessary
        console.error(error);
      } finally {
        this.isLoading = false;
        this.isBotMessageVisible = true;
      }
    },
    async generateResponse() {
      const userMessage = { role: "user", content: this.input };
      this.log.push(userMessage);
      await this.completionCall(this.input).then(() => {
        const botMessage = { role: "bot", content: this.response };
        this.log.push(botMessage);
        this.input = "";
        this.response = "";
      });
    },
    submitForm() {
      if (this.input.trim() !== "") {
        this.isLoading = true;
        this.isBotMessageVisible = false;
        this.generateResponse();
        this.input = "";
      }
    },
    stopLoading() {
      this.isLoading = false;
      this.isBotMessageVisible = true;
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

.spinner-dot {
  margin-left: 4px;
}
</style>
