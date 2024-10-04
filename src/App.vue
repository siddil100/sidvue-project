<template>
  <div class="call-toolbar-container">
    <div class="call-toolbar">
      <button @click="receiveCall">Receive Call</button>
      <button @click="dialCall">Dial Call</button>
      <button @click="toggleMute">{{ isMuted ? "Unmute" : "Mute" }}</button>
      <button @click="togglePause">{{ isPaused ? "Continue" : "Pause" }}</button>
    </div>

    <div v-if="incomingCall" class="incoming-call">
      <p>Incoming call from: {{ incomingCall.from }}</p>
      <div class="incoming-call-actions">
        <button class="accept-btn" @click="acceptCall">Accept</button>
        <button class="reject-btn" @click="rejectCall">Reject</button>
      </div>
    </div>

    <!-- Toast notification -->
    <div v-if="toastMessage" class="toast">
      <p>{{ toastMessage }}</p>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import * as signalR from '@microsoft/signalr';

export default {
  data() {
    return {
      isMuted: false,
      isPaused: false,
      callSid: null,
      incomingCall: null,
      connection: null,
      toastMessage: null,
    };
  },
  created() {
    this.setupSignalRConnection();
  },
  methods: {
    async setupSignalRConnection() {
      this.connection = new signalR.HubConnectionBuilder()
        .withUrl('https://sidvoice20241004183650.azurewebsites.net/callHub') // URL  SignalR hub
        .build();

      this.connection.on('ReceiveCall', (callSid, from) => {
        this.incomingCall = { callSid, from };
        console.log(`Incoming call from ${from}, callSid: ${callSid}`);
      });

      try {
        await this.connection.start();
        console.log('SignalR connection established successfully');
      } catch (error) {
        console.error('Error establishing SignalR connection:', error);
      }
    },

    async dialCall() {
      const phoneNumber = prompt('Enter the phone number to call:');
      if (!phoneNumber) return;
      try {
        const response = await axios.post(
          'https://sidvoice20241004183650.azurewebsites.net/api/call/makeCall', 
          phoneNumber, 
          { headers: { 'Content-Type': 'application/json' } }
        );
        this.callSid = response.data;
        alert('Calling...');
      } catch (error) {
        console.error('Error making call:', error);
      }
    },

    receiveCall() {
      alert('Ready to receive calls.');
    },

    async toggleMute() {
      if (!this.callSid) return;
      try {
        await axios.post('http://localhost:5232/api/call/mute', { callSid: this.callSid });
        this.isMuted = !this.isMuted;
      } catch (error) {
        console.error('Error toggling mute:', error);
      }
    },

    togglePause() {
      this.isPaused = !this.isPaused;
    },

    acceptCall() {
      this.showToast('Call accepted');
      this.incomingCall = null; // Hide the incoming call UI
    },

    rejectCall() {
      this.showToast('Call rejected');
      this.incomingCall = null; // Hide the incoming call UI
    },

    showToast(message) {
      this.toastMessage = message;
      setTimeout(() => {
        this.toastMessage = null; // Hide the toast after 3 seconds
      }, 3000);
    }
  }
};
</script>

<style scoped>
.call-toolbar-container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  height: 100vh;
}

.call-toolbar {
  display: flex;
  gap: 10px;
  justify-content: center;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  border: none;
  border-radius: 5px;
  transition: background-color 0.3s;
}

/* Styling for Accept and Reject buttons */
.accept-btn {
  background-color: #28a745; /* Green */
  color: white;
}

.accept-btn:hover {
  background-color: #218838;
}

.reject-btn {
  background-color: #dc3545; /* Red */
  color: white;
}

.reject-btn:hover {
  background-color: #c82333;
}

.incoming-call {
  margin-top: 20px;
  background-color: #f0f0f0;
  padding: 10px;
  border-radius: 5px;
  text-align: center;
}

.incoming-call-actions {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-top: 10px;
}

.toast {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  background-color: #333;
  color: white;
  padding: 10px 20px;
  border-radius: 5px;
  opacity: 0.9;
  text-align: center;
}
</style>
