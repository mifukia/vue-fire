<template>
  <div id="app">
    <main class="main">
      <div class="container">
        <ul class="list-group displayArea">
          <li v-for="item in list" class="list-group-item">
            {{item.name}} / {{item.message}}
          </li>
        </ul>
        <div class="login">
          <button type="button" class="btn btn-default" @click="login" v-show="!isLogin">
            匿名ユーザーでログイン
          </button>
        </div>
        <label for="nameInput">名前</label>
        <input type="text" id="nameInput" v-model="name">
        <label for="messageInput">メッセージ</label>
        <input type="text" id="messageInput" v-model="message">
        <button type="button" class="btn btn-default" @click="sendMessage">
          送信
        </button>
      </div>
    </main>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      isLogin:false,
      list: [],
      name:'',
      message:''
    }
  },
  methods:{
    autoScroll(){
      $('.displayArea').animate({scrollTop: $('.displayArea')[0].scrollHeight})
    },
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
      this.autoScroll();
    }
  },
  created(){
    firebase.auth().onAuthStateChanged(user=>{
      if(user){
        console.log('is login.')
        this.isLogin = true;
        this.listen();
      }else{
        console.log('No user is signed in.')
      }
    })
  },
  mounted(){
    this.autoScroll();
  }
}
</script>

<style lang="scss">
  .main{
    margin-top: 50px;
  }
  .displayArea{
    overflow-y: auto;
    height: 450px;
  }
</style>
