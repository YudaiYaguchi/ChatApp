<script setup>
import { inject, ref } from "vue"
import { useRouter } from "vue-router"
import socketManager from '../socketManager.js'

// #region global state
const userName = inject("userName")
// #endregion

// #region local variable
const router = useRouter()
const socket = socketManager.getInstance()
// #endregion

// #region reactive variable
const inputUserName = ref("")
// #endregion

// #region browser event handler
// 入室メッセージをクライアントに送信する
const onEnter = () => {
  if(inputUserName.value.length === 0){
    alert('ユーザー名を入力してください');
  }else{
    userName.value = inputUserName.value;  // 全体で使用するnameに入力されたユーザー名を格納
    console.log(userName.value);
  // チャット画面へ遷移
    router.push({ name: "chat" })
     // 入室メッセージを送信
    socket.emit("enterEvent", { userName: userName.value })
  }
}
// #endregion
</script>

<template>
<v-card
      class="mx-auto pa-12 pb-8"
      elevation="8"
      max-width="448"
      rounded="lg"
      style = "border: 2px solid #FFAC5B; margin-top: 10%;"
    >
    <v-card-title class="text-h5 font-weight-medium font-weight-black">vue.js chat サンプル</v-card-title>
      <div class="text-subtitle-1 text-medium-emphasis">ユーザー名</div>

      <v-text-field
      v-model="inputUserName"
        density="compact"
        prepend-inner-icon="mdi-email-outline"
        variant="outlined"
      ></v-text-field>

      <v-btn
      @click="onEnter"
        class="mb-8"
        color="orange"
        size="large"
        variant="tonal"
        block
      >
        入室する
      </v-btn>
    </v-card>
</template>

<!-- <style scoped>
.user-name-text {
  width: 200px;
  border: 1px solid #888;
  margin-bottom: 16px;
}
</style> -->
