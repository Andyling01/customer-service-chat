  <template>
    <div class="chat-window">
      <div class="chat-header">客服對話</div>
      <div class="chat-messages" ref="chatMessages">
        <div v-for="(message, index) in messages" :key="index" 
             :class="['message', message.isUser ? 'user-message' : 'bot-message']">
          {{ message.text }}
        </div>
      </div>
      <div class="chat-input">
        <input type="text" v-model="newMessage" @keyup.enter="sendMessage" placeholder="請輸入您的問題...">
        <button @click="sendMessage" :disabled="isLoading">發送</button>
      </div>
    </div>
  </template>

  <script>
  import axios from 'axios';

  export default {
    name: 'App',
    data() {
      return {
        messages: [
          { text: "您好！歡迎使用我們的客服系統。請問有什麼可以幫助您的嗎？", isUser: false }
        ],
        newMessage: '',
        isLoading: false
      }
    },
    methods: {
      async sendMessage() {
        if (this.newMessage.trim() && !this.isLoading) {
          // 添加用戶消息
          this.messages.push({ text: this.newMessage, isUser: true });
          const userMessage = this.newMessage;
          this.newMessage = '';
          this.isLoading = true;

          console.log('Sending message:', userMessage);

          try {
            // 發送請求到後端
            const response = await axios.post('http://localhost:5000/api/chat', {
              message: userMessage
            }, {
              headers: {
                'Content-Type': 'application/json'
              }
            });

            console.log('Received response:', response);

            // 添加後端回覆
            if (response.data && response.data.reply) {
              this.messages.push({ text: response.data.reply, isUser: false });
            } else {
              console.error('Invalid response structure:', response.data);
              throw new Error('Invalid response from server');
            }
          } catch (error) {
            console.error('Error sending message:', error);
            if (error.response) {
              console.error('Error response:', error.response.data);
              console.error('Error status:', error.response.status);
              console.error('Error headers:', error.response.headers);
            } else if (error.request) {
              console.error('Error request:', error.request);
            } else {
              console.error('Error message:', error.message);
            }
            this.messages.push({ text: "抱歉，發生了錯誤。請稍後再試。", isUser: false });
          } finally {
            this.isLoading = false;
            this.$nextTick(() => {
              this.scrollToBottom();
            });
          }
        }
      },
      scrollToBottom() { 
        const chatMessages = this.$refs.chatMessages;
        chatMessages.scrollTop = chatMessages.scrollHeight;
      }
    }
  }
  </script>



  <style>
  body {
    font-family: Arial, sans-serif;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    margin: 0;
    background-color: #f0f0f0;
  }
  .chat-window {
    width: 300px;
    height: 400px;
    border: 1px solid #ccc;
    border-radius: 10px;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    background-color: white;
  }
  .chat-header {
    background-color: #4CAF50;
    color: white;
    padding: 10px;
    text-align: center;
    font-weight: bold;
  }
  .chat-messages {
    flex-grow: 1;
    overflow-y: auto;
    padding: 10px;
  }
  .message {
    margin-bottom: 10px;
    padding: 5px 10px;
    border-radius: 10px;
    max-width: 70%;
  }
  .user-message {
    background-color: #e1ffc7;
    align-self: flex-end;
    margin-left: auto;
  }
  .bot-message {
    background-color: #f0f0f0;
    align-self: flex-start;
  }
  .chat-input {
    display: flex;
    padding: 10px;
    border-top: 1px solid #ccc;
  }
  .chat-input input {
    flex-grow: 1;
    padding: 5px;
    border: 1px solid #ccc;
    border-radius: 3px;
  }
  .chat-input button {
    padding: 5px 10px;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 3px;
    margin-left: 5px;
    cursor: pointer;
  }
  .chat-input button:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
  }
  </style>