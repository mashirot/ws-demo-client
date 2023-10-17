<template>
  <h3>UserId: {{ userId }}</h3>
  <div class="operation">
    <div>
      <button type="button" @click="getUserId">获取Id</button>
    </div>
    <div>
      <button type="button" @click="connect2Server">连接ws</button>
      <button type="button" @click="disconnect2Server">断开ws</button>
      <span>状态: {{ connectStatus }}</span>
    </div>
    <div>
      <button type="button" @click="broadcast">发送广播</button>
      <span>Content: </span><input type="text" v-model="broadcastContent">
    </div>
    <div>
      <button type="button" @click="privateChat">指定UserId发送</button>
      <span>Receiver's userId: </span><input type="text" v-model="privateChatUserId">
      <span>Content: </span><input type="text" v-model="privateChatContent">
    </div>
  </div>
  <hr/>
  <div>
    <div v-for="(msg, idx) in msgQueue" :key="idx">
      {{ msg }}
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import {reactive, ref} from "vue";

const uri = "127.0.0.1:8080"
const httpUri = `http://${uri}`
const wsEndpoint = `ws://${uri}/api/ws/`

const userId = ref(0);
const connectStatus = ref("断开")
const broadcastContent = ref("")
const privateChatUserId = ref("")
const privateChatContent = ref("")
const msgQueue = reactive([])
let webSocket;

const getUserId = () => axios.get(`${httpUri}/api/rest/ws/userId`)
    .then(resp => {
      userId.value = resp.data
    })
    .catch(e => console.log(e));

function connect2Server() {
  webSocket = new WebSocket(wsEndpoint + userId.value);
  webSocket.onopen = onOpen
  webSocket.onclose = onClose
  webSocket.onerror = onErr
  webSocket.onmessage = onMessage
}

function disconnect2Server() {
  webSocket.close()
}

const send2Server = (msg) => {
  webSocket.send(msg);
}

const onOpen = () => {
  sysBroadcast(`【广播】userId: ${userId.value} 连接到Server`)
  connectStatus.value = "连接"
}

const onClose = (e) => {
  sysBroadcast(`【广播】userId: ${userId.value} 断开连接, ${e.code}`)
  connectStatus.value = "断开"
}

const onMessage = (e) => {
  const data = eval("(" + e.data + ")")
  msgQueue.push(data.msg)
}

const onErr = () => console.log("WebSocket连接发生错误")

const broadcast = () => {
  axios.post(`${httpUri}/api/rest/ws/broadcast`, {
    msg: "【广播】" + userId.value + ": " + broadcastContent.value
  })
  broadcastContent.value = ""
}

const sysBroadcast = (msg) => axios.post(`${httpUri}/api/rest/ws/broadcast`, {
  msg: msg
})

const privateChat = () => {
  axios.post(`${httpUri}/api/rest/ws/sendMsg/user/${privateChatUserId.value}`, {
    msg: "【私聊】" + userId.value + ": " + privateChatContent.value
  })
  msgQueue.push(`【私聊】To ${privateChatUserId.value}: ${privateChatContent.value}`)
  privateChatUserId.value = ""
  privateChatContent.value = ""
}
</script>

<style scoped>
</style>
