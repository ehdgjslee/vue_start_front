<template>
  <v-form>
    <v-btn @click="connect">connect</v-btn>
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
      <input type="file" id="inputImage" @change="handleFileChange" multiple />
      <img :src="sendImgSrc" />
      <v-row>
        <v-col v-if="recvList" cols="12" md="4">
          <v-card width="400" v-for="(item, idx) in recvList" :key="idx">
            <v-card-item :name="idx">
              <v-card-title>{{ item.nickname }}</v-card-title>
              <v-card-subtitle>This is a subtitle</v-card-subtitle>
            </v-card-item>
            <v-card-text>
              {{ item.content }}
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-form>
</template>

<script>
import SockJS from 'sockjs-client';
import Stomp from 'webstomp-client';

export default {
  name: "socketMessenger",
  data() {
    return {
      nickname: '닉네임',
      content: '',
      sendImgSrc: '',

      recvList: [],
    };
  },
  mounted() {
    this.connect();
  },
  methods: {
    addRecvData(res) {
      const recv = JSON.parse(res.body);
      this.recvList.push(recv);
    },
    connect() {
      const serverURL = "http://172.20.10.7:8080";
      let socket = new SockJS(serverURL); //소켓을위한 서버주소 셋팅
      this.stompClient = Stomp.over(socket); //Stomp에 소켓 등록
      this.stompClient.connect(
        //등록된 소켓으로 실제로 연결
        {},
        () => {
          this.stompClient.subscribe("/send", res => this.addRecvData(res), e => {const errorJson = JSON.stringify(e, null, 2); console.log(errorJson); alert(errorJson)});
        },
        e => {
          console.log(JSON.stringify(e, null ,2));
        }
      );
    },
    send() {
      if (!this.content && !this.sendImgSrc) {
        alert("내용 입력 또는 사진 업로드 후 전송해주세요.");
        return;
      }
      if (!this.stompClient || !this.stompClient.connected) {
        alert("소켓 연결 후 전송해주세요.");
        return;
      }
      const msg = {
        nickname: this.nickname,
        content: this.content,
        sendImgSrc: this.sendImgSrc,
      };
      this.stompClient.send("/receive", JSON.stringify(msg), {});
      this.content = '';
    },
    handleFileChange(e) {
      const ctx = this;
      const file = e.target.files[0];
      const FR = new FileReader();
      FR.readAsDataURL(file);
      FR.onload = () => {
        ctx.sendImgSrc = FR.result;
      };
    },
  },
};
</script>
