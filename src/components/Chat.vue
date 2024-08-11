<script setup>
import { inject, ref, reactive, onMounted, provide } from "vue";
import socketManager from "../socketManager.js";
import ImageSubmission from "./ImageSubmission.vue";
import DisplayTyping from "./DisplayTyping.vue"

// #region global state
const userName = inject("userName");
// #endregion

// #region local variable
const socket = socketManager.getInstance();
provide("socket", socket);  // Provide socket to child components
// #endregion

// #region reactive variable
const chatContent = ref("");
const chatList = reactive([]);
const imageFiles = ref([]); // ImageSubmission.vueと共有するための変数
provide("chatList", chatList); // chatListを子コンポーネントに提供する
provide("imageFiles", imageFiles); // imageFilesを子コンポーネントに提供する
// #endregion

// 入力欄がフォーカスされているか（追記）
const isFocused = ref(false)
// 入力欄に表示するテキスト（追記）
const defaultText = ref("投稿文を入力してください")

// #region lifecycle
onMounted(() => {
  registerSocketEvent();
});
// #endregion

// #region browser event handler
// 入室メッセージをサーバに送信する
const onEnter = () => {
  socket.emit("enterEvent", { userName: userName.value });
};

// 投稿メッセージをサーバに送信する
const onPublish = () => {
  const message = chatContent.value.trim();
  const image = imageFiles.value.length > 0 ? imageFiles.value[0] : null;

  if (message || image) {
    socket.emit("publishEvent", {
      userName: userName.value,
      message: message,
      image: image,
    });
    chatContent.value = "";
    imageFiles.value = []; // 送信後、画像リストをクリア
  }
};

// 退室メッセージをサーバに送信する
const onExit = () => {
  socket.emit("exitEvent", { userName: userName.value });
};

// メモを画面上に表示する
const onMemo = () => {
  chatList.unshift({
    message: userName.value + "さんのメモ：" + chatContent.value,
    image: null,
  });
  chatContent.value = "";
};
// #endregion

// #region socket event handler
// サーバから受信した入室メッセージ画面上に表示する
const onReceiveEnter = (data) => {
  chatList.unshift({
    message: `${data.userName}さんが入室しました。`,
    image: null,
  });
};

// サーバから受信した退室メッセージを受け取り画面上に表示する
const onReceiveExit = (data) => {
  chatList.unshift({
    message: `${data.userName}さんが退室しました。`,
    image: null,
  });
};

// サーバから受信した投稿メッセージを画面上に表示する
const onReceivePublish = (data) => {
  chatList.unshift({
    message: `${data.userName}さん: ${data.message}`,
    image: data.image,
  });
};
// #endregion

// #region local methods
// イベント登録をまとめる
const registerSocketEvent = () => {
  socket.on("enterEvent", (data) => {
    onReceiveEnter(data);
  });

  socket.on("exitEvent", (data) => {
    onReceiveExit(data);
  });

  socket.on("publishEvent", (data) => {
    onReceivePublish(data);
  });
};
// #endregion
</script>

<template>
    <v-layout>
      <v-app-bar
        color="#FFAC5B"
        density="compact"
      >

        <v-app-bar-title class = "text-white font-weight-black text-h4">Vue.jsチャットルーム</v-app-bar-title>
        <template v-slot:append>
          <p style = "margin-right: 20px;">ログインユーザ：{{ userName }}さん</p>
          
          <router-link to="/" class="link">
            <v-btn round color="orange" class = "bg-white font-weight-bold" dark @click="onExit">退室する</v-btn>
          </router-link>
        </template>
      </v-app-bar>

      <v-main>
        <v-card class="mx-auto mt-10" max-width="1000" border flat>
          <v-container align="center" justify="center">
            <v-row 
            >
              <v-col cols="12" class = "mb-2">
                <DisplayTyping :Focused="isFocused" :Myname="userName" @sendText="defaultText = $event" />
                <textarea v-model="chatContent" variant="outlined" v-bind:placeholder="defaultText" rows="4" class="area ma-0"
                @focus="isFocused = true" @blur="isFocused = false"></textarea>
              </v-col>
            </v-row>
            
            <v-row align="center" justify="center">
              <v-col cols="3" class = "d-flex align-center justify-center">
                <v-btn density="default" @click="onPublish" color="#FFAC5B"
                  size="large"
                  block class = "text-white text-h6 font-weight-bold">投稿</v-btn>
              </v-col>
              <v-col cols="3" class = "d-flex align-center justify-center">
                <v-btn  @click="onMemo" 
                density="default"
                size="large"
                
                block class = "text-orange text-h6 font-weight-bold">メモ</v-btn>
              </v-col>
              <v-col cols="2">
                  <ImageSubmission /><!-- ImageSubmission.vueの呼び出し -->
              </v-col>
            </v-row>
          </v-container>
        </v-card>

        <!-- ここまで完成 -->

        <div class="mx-auto my-5 px-4">
          <div class="mt-10">
            <div class="mt-5" v-if="chatList.length !== 0">
              <ul>
                <!-- v-forでメッセージと画像をループ表示 -->
                <li class="item mt-4" v-for="(chat, i) in chatList" :key="i">
                  <p>{{ chat.message }}</p>
                  <!-- imageプロパティが存在するかをチェックして表示 -->
                  <img v-if="chat.image" :src="chat.image" class="ImageDisplay" />
                </li>
              </ul>
            </div>
          </div>
        </div>
      </v-main>
    </v-layout>
</template>

<style scoped>
.link {
  text-decoration: none;
}

.area {
  width: 80%;
  border: 1px solid #000;
  margin-top: 8px;
}

.item {
  display: block;
}

.util-ml-8px {
  margin-left: 8px;
}

.button-exit {
  color: #000;
  margin-top: 8px;
}

.ImageDisplay {
  width: 200px;
  height: 200px;
  object-fit: cover;
  border: 1px solid #ccc;
  margin: 5px;
}
</style>
