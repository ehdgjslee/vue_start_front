<template>
  <v-form>
    <v-container>
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field v-model="user.mbId" label="ID" required></v-text-field>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" sm="4">
          <v-text-field
            v-model="user.mbPw"
            :append-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'"
            :rules="[rules.required, rules.min]"
            :type="show1 ? 'text' : 'password'"
            name="input-10-1"
            label="PASSWORD"
            hint="At least 11 characters"
            counter
            @click:append="show1 = !show1"
          ></v-text-field>
        </v-col>
      </v-row>
      <v-row v-for="users in users" v-bind:key="users.seq"
        >{{ users.mbId }}{{ users.mbPw }}</v-row
      >
    </v-container>
    <v-btn @click="connect">connect</v-btn>
    <v-btn @click="send">get</v-btn>
    <v-btn @click="login22">post</v-btn>
    <input type="file" id="inputImage" @change="handleFileChange" multiple />
    <img id="imgSrc" />
  </v-form>
</template>

<script>
import axios from "axios";
export default {
  name: "IndexPage",
  data() {
    return {
      id: "",
      show1: false,
      realId: "leedong",
      realPw: "12345678910",
      password: "",
      users: [],
      rules: {
        required: (value) => !!value || "Required.",
        min: (v) => v.length >= 11 || "Min 11 characters",
      },
      user: {
        mbId: "",
        mbPw: "",
        image: "",
      },
      imgSrc: "",
    };
  },
  methods: {
    async login() {
      try {
        let response = await axios.post("/api/member/find", this.user);
        console.log(response);
        const user = response.data;
        if (user.length <= 0) {
          alert("로그인 실패");
          return false;
        }
        alert("로그인 성공");
      } catch (error) {
        console.error(error);
      }
    },
    imgDelete() {
      console.log("ddddddd");
    },
    handleFileChange(e) {
      const ctx = this;
      const file = e.target.files[0];
      this.imgSrc = URL.createObjectURL(file);
      document.getElementById("imgSrc").src = this.imgSrc;
      const FR = new FileReader();
      FR.readAsDataURL(file);
      FR.onload = () => {
        ctx.user.image = FR.result;
      };
    },
    readImage(e) {
      let users = this.user;
      console.log(e.target.result);
      let FR = new FileReader();
      FR.readAsDataURL(e.target.files);
      FR.onload = function (e) {
        users.image = e.target.result;
        let response = axios.post("/api/member/resize", users);
        console.log(response);
      };
    },
    login22: function () {
      if (this.password.length <= 8) {
        alert("short");
        return false;
      }
      if (this.id === 0 || this.realId !== this.id) {
        alert("id not corredt");
        return false;
      }
      if (this.realPw !== this.password) {
        alert("pw not corredt");
        return false;
      }
      var vm = this.users;
      axios
        .get("https://jsonplaceholder.typicode.com/users/")
        .then(function (response) {
          console.log(response.data);
          vm.users = response.data;
        })
        .catch(function (error) {
          console.log(error);
        });
      // this.$router.push({ path: "/inspire" });
    },
    async asyncDate() {
      console.log("dkdkdkdkdkdkdk");
      try {
        let { data } = await axios.get("/api/member/find");
        this.users = data;
        console.log(data);
      } catch (error) {
        console.error(error);
      }
    },
    connect() {
      const serverURL = "http://172.20.10.6:8080";
      let socket = new SockJS(serverURL); //소켓을위한 서버주소 셋팅
      this.stompClient = Stomp.over(socket); //Stomp에 소켓 등록

      this.stompClient.connect(
        //등록된 소켓으로 실제로 연결
        {},
        (frame) => {
          this.stompClient.subscribe("/send", (res) => {
            console.log("구독으로 받은 메시지", res.body);
          });
        }
      );
    },
    send() {
      if (this.stompClient && this.stompClient.connected) {
        const msg = {
          userName: this.userName,
          content: this.message,
        };
        this.stompClient.send("/receive", JSON.stringify(msg), {});
      }
    },
  },
};
</script>
