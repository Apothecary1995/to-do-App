<template>
  <div id="main-container">
    <div id="chat" class="right-panel">
      <h2>Chat</h2>
      <div class="chat-box">
        <div
          class="chat-message"
          v-for="msg in messages"
          :key="msg.messageId"
          :class="{ 'from-me': msg.sender === username, 'from-other': msg.sender !== username }"
        >
          <div class="bubble">
            <strong>{{ msg.sender }}:</strong>
            <p>{{ msg.message }}</p>
            <small>{{ formatTimestamp(msg.timestamp) }}</small>
          </div>
        </div>
      </div>
      <input
        type="text"
        v-model="chatInput"
        @keydown.enter="sendMessage"
        placeholder="Bura mesaj Y DUR GARİ..."
        class="chat-input"
      />
    </div>

    <div class="image-wrapper">
      <img alt="logo" src="./assets/logo3.jpg" class="header-image" />
    </div>

    <div id="APP">
      <h1>TO-DO APP</h1>
      <div class="add task">
        <input type="text" v-model="newTaskInput" @keydown.enter="addTask" />
        <button @click="addTask">Add task</button>
      </div>
      <div class="task" v-for="task in tasks" :key="task.id">
        <span>{{ task.name }}</span>
        <span class="Delete">
          <button @click="removeTask(task.id)">X</button>
        </span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      newTaskInput: "",
      tasks: [],
      chatInput: "",
      messages: [],
      username: "User" + Math.floor(Math.random() * 1000), // Random username
      pollInterval: null
    }
  },
  async created() {
  
    const taskResponse = await fetch("https://g2vulkypqk.execute-api.eu-north-1.amazonaws.com/todos");
    this.tasks = await taskResponse.json();
    
    
    await this.loadMessages();
    
   
    this.pollInterval = setInterval(this.loadMessages, 2000);
  },
  beforeUnmount() {
    // Clear the polling interval when component is destroyed
    clearInterval(this.pollInterval);
  },
  methods: {
    formatTimestamp(timestamp) {
      if (!timestamp) return '';
      const date = new Date(timestamp);
      return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    },
    async loadMessages() {
      try {
        const response = await fetch("https://z2hyq4vwn2.execute-api.eu-north-1.amazonaws.com/prod/messages");
        const data = await response.json();
        this.messages = data.sort((a, b) => new Date(a.timestamp) - new Date(b.timestamp));
      } catch (error) {
        console.error("Error loading messages:", error);
      }
    },
    async sendMessage() {
      if (!this.chatInput.trim()) return;
      
      const newMessage = {
        messageId: Date.now().toString(),
        sender: this.username,
        message: this.chatInput,
        timestamp: new Date().toISOString()
      };
      
      try {
        await fetch("https://z2hyq4vwn2.execute-api.eu-north-1.amazonaws.com/prod/messages", {
          method: "POST",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify(newMessage)
        });
        
        this.chatInput = "";
        await this.loadMessages(); 
      } catch (error) {
        console.error("Error sending message:", error);
      }
    },
    async addTask() {
      let taskId = Math.floor(Math.random() * (999999999 - 100000000) + 100000000);
      let newTask = {
        id: taskId,
        name: this.newTaskInput
      };
      
      await fetch("https://g2vulkypqk.execute-api.eu-north-1.amazonaws.com/todos", {
        method: "post",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify(newTask)
      });
      
      this.tasks.push(newTask);
      this.newTaskInput = "";
    },
    async removeTask(taskId) {
      await fetch("https://g2vulkypqk.execute-api.eu-north-1.amazonaws.com/todos", {
        method: "delete",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify({ id: taskId })
      });
      
      this.tasks = this.tasks.filter(t => t.id !== taskId);
    }
  }
}
</script>

<style scoped>
#main-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  min-height: 100vh;
  padding: 20px;
}

.image-wrapper {
  display: flex;
  justify-content: center;
  margin-top: 20px;
  margin-bottom: 20px;
}

.header-image {
  width: 400px;
  height: auto;
  border-radius: 16px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

#chat.right-panel {
  position: fixed;
  right: 20px;
  top: 20px;
  width: 280px;
  height: calc(100vh - 40px);
  background: #fefefe;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
  font-family: 'Poppins', sans-serif;
  display: flex;
  flex-direction: column;
  z-index: 100;
}

#chat h2 {
  margin-bottom: 12px;
  font-size: 18px;
  font-weight: 600;
  color: #6c5ce7;
  text-align: center;
}

.chat-box {
  flex-grow: 1;
  overflow-y: auto;
  margin-bottom: 15px;
  padding-right: 8px;
}

.chat-message {
  margin-bottom: 10px;
  display: flex;
  flex-direction: column;
}

.chat-message .bubble {
  padding: 10px 14px;
  border-radius: 14px;
  max-width: 90%;
  word-wrap: break-word;
  font-size: 14px;
  line-height: 1.4;
  position: relative;
}

.from-me .bubble {
  background-color: #dfe6fd;
  align-self: flex-end;
  text-align: right;
  border-bottom-right-radius: 0;
}

.from-other .bubble {
  background-color: #f1f1f1;
  align-self: flex-start;
  text-align: left;
  border-bottom-left-radius: 0;
}

.bubble small {
  display: block;
  font-size: 0.7em;
  opacity: 0.7;
  margin-top: 4px;
}

.chat-input {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
  background-color: #f8f8f8;
  box-sizing: border-box;
}

.chat-input:focus {
  outline: none;
  border-color: #6c5ce7;
  box-shadow: 0 0 0 2px rgba(108, 92, 231, 0.2);
}

#APP {
  font-family: 'Poppins', sans-serif;
  max-width: 420px;
  padding: 30px 20px;
  background: linear-gradient(145deg, #ffffff, #f0f0f0);
  border-radius: 20px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
  margin-top: 20px;
}


h1 {
  font-size: 28px;
  font-weight: 700;
  color: #333;
  margin-bottom: 30px;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.add.task {
  display: flex;
  gap: 10px;
  margin-bottom: 25px;
}

.add.task input {
  flex: 1;
  padding: 12px 16px;
  font-size: 15px;
  border: none;
  border-radius: 12px;
  background-color: #f8f8f8;
  box-shadow: inset 2px 2px 6px #e0e0e0, inset -2px -2px 6px #ffffff;
  transition: all 0.2s ease-in-out;
}

.add.task input:focus {
  outline: none;
  box-shadow: 0 0 0 2px #6c5ce7;
}

.add.task button {
  padding: 12px 18px;
  background-color: #6c5ce7;
  color: white;
  font-weight: 600;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.add.task button:hover {
  background-color: #5a4ec9;
}

.task {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: #ffffff;
  padding: 14px 16px;
  border-radius: 12px;
  box-shadow: 0 4px 8px rgba(108, 92, 231, 0.1);
  margin-bottom: 12px;
  transition: transform 0.2s ease-in-out;
}

.task:hover {
  transform: scale(1.02);
}

.task span:first-child {
  font-size: 16px;
  color: #333;
  word-break: break-word;
  flex: 1;
}

.Delete button {
  background-color: #ff6b81;
  color: white;
  border: none;
  padding: 8px 12px;
  font-weight: bold;
  border-radius: 10px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.Delete button:hover {
  background-color: #e55060;
}
</style>
