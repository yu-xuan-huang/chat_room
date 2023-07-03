<template>
  <div id="app">
    <Modal
      :mask-closable="false"
      v-model="modal"
      title="請輸入暱稱"
      @on-ok="login"
    >
      <div ref="loginDialog" title="請輸入暱稱">
        <input
          class="nickname"
          v-model="nickname"
          type="text"
          placeholder="請輸入暱稱"
        />
      </div>
    </Modal>
    <div class="web-im">
      <div class="header">聊天室</div>
      <div class="content-chat">
        <div
          class="li"
          :class="{ user: item.uid == uid }"
          v-for="(item, index) in messageList"
          :key="index"
        >
          <template v-if="item.type === 1">
            <p class="join-tips">{{ item.msg }}</p>
          </template>
          <template v-else>
            <div
              v-if="item.nickname === nickname"
              class="message-list-bymyself"
            >
              <div class="message-box">{{ item.msg }}</div>
              <div class="circle">
                <div class="img">{{ item.nickname }}</div>
              </div>
            </div>
            <div v-else class="message-list-by-others">
              <div class="circle">
                <div class="img">{{ item.nickname }}</div>
              </div>
              <div class="message-box-left">{{ item.msg }}</div>
            </div>
          </template>
        </div>
      </div>
      <div class="footer-chat">
        <Input
          @on-enter="send"
          type="text"
          v-model="msg"
          placeholder="請輸入內容"
        />
        <Button type="success" @click="send">發送訊息</Button>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: "ChatSample",
  data() {
    return {
      uid: "",
      nickname: "",
      socket: "",
      msg: "",
      messageList: [],
      modal: false,
    };
  },
  mounted() {
    let vm = this;
    let user = localStorage.getItem("WEB_IM_USER");
    user = (user && JSON.parse(user)) || {};
    vm.uid = user.uid;
    vm.nickname = user.nickname;
    if (vm.uid) {
      vm.conWebSocket();
    } else {
      vm.modal = true;
    }
  },
  methods: {
    send() {
      if (!this.msg) {
        return;
      }
      this.sendMessage(2, this.msg);
    },
    sendMessage(type, msg) {
      this.socket.send(
        JSON.stringify({
          uid: this.uid,
          type: type,
          nickname: this.nickname,
          msg: msg,
        })
      );
      this.msg = "";
    },
    conWebSocket() {
      let vm = this;
      if (window.WebSocket) {
        vm.socket = new WebSocket("wss://chat-sample-knyh.onrender.com");
        let socket = vm.socket;
        socket.onopen = function () {
          console.log("連接伺服器成功");
          if (!vm.uid) {
            vm.uid = "web_im_";
            localStorage.setItem(
              "WEB_IM_USER",
              JSON.stringify({
                uid: vm.uid,
                nickname: vm.nickname,
              })
            );
            vm.sendMessage(1);
          }
        };
        socket.onclose = function () {
          console.log("伺服器關閉");
        };
        socket.onerror = function () {
          console.log("連接錯誤");
        };
        // 接收伺服器的消息
        socket.onmessage = function (e) {
          let message = JSON.parse(e.data);
          vm.messageList.push(message);
        };
      }
    },
    login() {
      this.conWebSocket();
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped type='scss'>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.header {
  padding: 10px;
  border-radius: 5px;
  border: solid #888888;
  text-align: left;
}
.content-chat {
  width: 100%;
  height: 500px;
  background: #f2f1f1;
  overflow-y: scroll;
}
.footer-chat {
  display: flex;
  justify-content: center;
  align-items: center;
}
.message-list-bymyself {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  margin: 1%;
}
.message-list-by-others {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  margin: 1%;
}
.message-box {
  padding: 5px;
  background: deepskyblue;
  border-radius: 8px;
  margin-right: 5px;
}
.message-box-left {
  padding: 5px;
  background: #42b983;
  border-radius: 8px;
  margin-left: 5px;
}
.circle {
  display: block;
  float: left;
  position: relative;
  width: 50px;
  height: 50px;
  background-color: grey;
  display: block;
  overflow: hidden;
  border-radius: 50%;
}
.circle > div {
  color: black;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  white-space: nowrap;
}
</style>
