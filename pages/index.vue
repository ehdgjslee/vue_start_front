<template>
  <v-form>
    <v-container>
      <v-row>
        <v-col cols="12" sm="4">
          <v-text-field v-model="nickname" label="ID" required></v-text-field>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="4">
          <v-textarea label="내용 입력" v-model="content" @keyup.enter="send"></v-textarea>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field v-for="(item, idx) in recvList" :key="idx">{{ item }}</v-text-field>
        </v-col>
      </v-row>
    </v-container>
    <v-btn @click="connect">connect</v-btn>
  </v-form>
</template>

<script>
import SockJS from 'sockjs-client';
import Stomp from 'webstomp-client';

export default {
  name: "IndexPage",
  data() {
    return {
      nickname: '닉네임',
      content: '',
      recvList: [],
    };
  },
  methods: {
    addRecvData(res) {
      const recv = res.body;
      this.recvList.push(recv);
    },
    connect() {
      const serverURL = "http://172.20.10.6:8080";
      let socket = new SockJS(serverURL); //소켓을위한 서버주소 셋팅
      this.stompClient = Stomp.over(socket); //Stomp에 소켓 등록
      this.stompClient.connect(
        //등록된 소켓으로 실제로 연결
        {},
        () => {
          this.stompClient.subscribe("/send", res => this.addRecvData(res));
        },
        e => {
          console.log(JSON.stringify(e, null ,2));
        }
      );
    },
    send() {
      if (!this.content) {
        alert("내용 입력 후 전송해주세요.");
        return;
      }
      if (!this.stompClient || !this.stompClient.connected) {
        alert("소켓 연결 후 전송해주세요.");
        return;
      }
      const msg = {
        nickname: this.nickname,
        content: this.content,
      };
      this.stompClient.send("/receive", JSON.stringify(msg), {});
      this.content = '';
    },
  },
};
</script>
