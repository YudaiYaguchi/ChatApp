<script setup>
import { watch, ref, reactive, onMounted} from 'vue';
import socketManager from '../socketManager.js'

// 入力欄に表示するテキスト
const defaultText = ref("")

// 親からもらう値
const props = defineProps({
Focused: {
    type: Boolean,
    required: true
},
Myname: {
    type: String,
    required: true
}
});

// 親に送る値
const emit = defineEmits(['sendText']);

const socket = socketManager.getInstance()

onMounted(() => {
    registerSocketEvent()
})
// 入力中表示をサーバーに送信する，flagが変化するたびに実行
const onTyping = (flag) => {
    changeText(flag)
    socket.emit('typingEvent',{ FormText: defaultText.value });
}
// サーバーから受信したdataをdefaultTextに格納
const onReceiveTyping = (data) => {
    // 受信した内容に書き換える
    defaultText.value = data.FormText
    // Chat.vueに送る
    emit('sendText', defaultText.value)
    //console.log("サーバーから受信したデータ",data.FormText)
}
const registerSocketEvent = () => {
    // 入力中イベントを受け取ったら実行
    socket.on("typingEvent", (data) => {
        onReceiveTyping(data)
    })
}

// 入力欄に表示されるテキストを変更する
const changeText = (FormIsActive) =>{
    if(FormIsActive){
        defaultText.value = props.Myname+"さんが入力中"
    }
    else{
        defaultText.value = "投稿文を入力してください"
    }
}

// 入力欄のTFの変化を監視する
watch(() => props.Focused,
(newVal) => {
    onTyping(newVal)
});
</script>

<template>
</template>