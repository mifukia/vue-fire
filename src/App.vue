<template>
  <div id="app">
    <main class="main">
      <div class="container">
        <button type="button" class="btn btn-default" @click="login">
          匿名ユーザーでログイン
        </button>
        <label for="nameInput">名前</label>
        <input type="text" id="nameInput" v-model="name">
        <label for="messageInput">メッセージ</label>
        <input type="text" id="messageInput" v-model="message">
        <button type="button" class="btn btn-default" @click="sendMessage">
          送信
        </button>
        <ul class="list-group">
          <li v-for="item in list" class="list-group-item">
            {{item.name}} / {{item.message}}
          </li>
        </ul>
      </div>
    </main>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      list: [],
      name:'',
      message:''
    }
  },
  methods:{
    login(){
      firebase.auth().signInAnonymously().then(e => {
        //ログイン成功
        console.log(e)
        this.listen();
      }).catch((error) => {
        //エラーメッセージ
        const errorCode = error.code;
        const errorMessage = error.message;
        console.log('エラーメッセージ',errorCode,errorMessage)
      })
    },
    listen(){
      firebase.database().ref('myBoad/').on('value',snapshot => {
        if(snapshot){
          const rootList = snapshot.val();
          let list = [];
          console.log(Object.keys(rootList))
          //データリストを配列に変換する
          Object.keys(rootList).forEach((val,key) => {
            rootList[val].id = val;
            list.push(rootList[val])
          })
          this.list = list;
        }
      })
    },
    pushData(){
      firebase.database().ref('myBoad/').push({
        name:'test',
        message:'foo'
      })
    },
    sendMessage(){
      // 空欄の場合は送信しない
      if(!this.name || !this.message) return;
      //送信
      firebase.database().ref('myBoad/').push({
        name:this.name,
        message:this.message
      })
      //送信後にinputを空にする
      this.message = "";
    }
  },
  created(){
    firebase.auth().onAuthStateChanged(user=>{
      if(user){
        console.log('is login.')
        this.listen();
      }else{
        console.log('No user is signed in.')
      }
    })
  }
}
</script>

<style lang="scss">
  .main{
    margin-top: 70px;
  }
</style>
