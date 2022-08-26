<template>

<div style="display: flex; flex-direction: column; max-width: 600px; margin:auto">
  <h2>Chattrum</h2>
  <button 
  style="padding:5px; margin: 5px" 
  v-for="room in roomS" :key="room.room"
  @click="connect(room.room)">
    {{room.room}}
  </button>
</div>

<form @submit.prevent style="max-width: 600px; margin: 30px auto">
<div style="display:flex; flex-direction:column">
<input type="text" v-model="roomValue" placeholder="room">
<button @click="createRoom">Create room</button> 
<div class="error" v-if="errorInputCreate">{{errorInputCreate}}</div>
</div>

<div style="margin: 20px">
<button @click="leaveRoom(activeRoom)">Leave Chat</button> 
</div>

<div>
<h2>{{activeRoom}}</h2>
<input type="text" v-model="messages" placeholder="message">
<div class="error" v-if="errorInputMessage">{{errorInputMessage}}</div>
<button @click="handleSubmit">Skicka</button> 
</div>

<div class="messageBox" style="display:flex; flex-direction: column" v-if="messageArray" v-for="items in messageArray" :key="items">
<div class="headerMessage" style="display:flex; justify-content: space-between; color:white; padding: 20px 30px"><p class="messageName">{{items.name}}</p><p class="timeAndDate">{{items.timestamp}}</p></div>
<div style="padding: 20px; "><p class="messageText">{{items.message}}</p></div>
</div>
</form>

</template>

<script>

import {io} from 'socket.io-client'
import {ref} from 'vue'


export default {
setup(){
let userNamene = prompt("Skriv in ditt användarnamn")
const socket = io("https://desolate-everglades-50260.herokuapp.com/")
const messages = ref("")
const errorInputCreate = ref("")
const errorInputMessage = ref("")
const roomS = ref()
const roomValue =ref("")
const messageArray = ref()
const joinedUsers = ref()
const activeRoom = ref("Skapa eller joina ett rum")
/* const userName = ref() */

socket.on('connect', (data) => {
while (userNamene === null) {
userNamene = prompt("Du måste skriva in ett användarnamn")
} 
socket.emit('new user', (userNamene))
      socket.on("rooooms", (data) => {
        roomS.value = data
      })
      socket.on("activeRoom", (data) => {
        activeRoom.value = data
      })
      socket.on("error", (data) => {
        if (data.location === "create") {
          errorInputMessage.value = null 
           errorInputCreate.value = data.error
        }
        else if (data.location === "message") {
          errorInputCreate.value = null
          errorInputMessage.value = data.error
        } else {
          errorInputCreate.value = null
          errorInputMessage.value = null 
        }
      })

      socket.on('joined_room', (data) =>{
        console.log("Har joinat",data)
        joinedUsers.value = data
        messageArray.value = data
      })  
      
 
      socket.on('message', (data) => {
        messageArray.value = data
        console.log(messageArray.value)
      })

     socket.on('leave_room', (data) => {
        console.log("datalämna", data)
        messageArray.value = false
        roomS.value = data

      })
    })

function connect (room) {
console.log(room)
socket.emit('join_room', room)
}

function createRoom() {
 errorInputCreate.value = null
errorInputMessage.value = null 
socket.emit('create_room', roomValue.value)
}


function leaveRoom(data) {
   errorInputCreate.value = null
          errorInputMessage.value = null 
  console.log(data)
socket.emit('leave_room', data)
activeRoom.value ="Skapa eller joina ett rum"
}

function handleSubmit() {
socket.emit('message', messages.value)

}




return {errorInputMessage,errorInputCreate,activeRoom,createRoom,roomS,roomValue,connect, messageArray, messages,handleSubmit, leaveRoom}
}}

</script>

<style>
*{
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}


.error{
  font-size: 15px;
  color: rgb(161, 11, 11);
  margin: 5px;
} 
.messageName{
font-size: 16px;
font-weight: 800;
color: white
}
.timeAndDate{
font-size: 11px;

}
.messageText{
font-size: 16px;
text-align: left;
}
.messageBox{
  overflow: hidden;
  border-radius: 5px;
  margin: 30px;
  border: solid 1px rgba(0, 0, 0, 0.339)
}
.headerMessage{
background-color: rgb(10, 9, 9);

}
</style>
