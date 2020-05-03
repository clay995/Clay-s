<template>
  <div id="app">
    <choose-user v-if="$root.me==null" :userlist='userlist'></choose-user>
    <userlist :islogin='islogin' :users='users' :chooseUser='chooseUser' :unreadlist='unreadlist'></userlist> 
    <chat-user v-if="ischat" :touser='touser' :closeChat='closeChat' :newmsg='newmsg'></chat-user>
  </div>
</template>

<script>
import chooseUser from './components/chooseUser'
import axios from 'axios'
import userlist from './components/userlist'
import socket from './socket'
import chatUser from './components/chatUser';

export default {
  name: 'App',
  components: {
    chooseUser,
    userlist,
    chatUser,
  },
  data() {
    return {
      userlist:[],
      islogin:false,
      users:[],
      ischat:false,
      touser:null,
      unreadlist:[],
      newmsg:null
    }
  },
  async  beforeMount() {
    let res = await  axios.get('/api/userlist')
          this.userlist=res.data
          
  },
  mounted() {
    //监听登陆事件，登陆成功islogin=true 
        socket.on('login',(data)=>{
              if(data.state=='ok'){
                this.islogin=true;
                socket.emit('users')
              }
        })  
    //监听登出事件，islogin=false  
        socket.on('logout',(data)=>{
          console.log(data)
          this.islogin=false;
          //断开连接
          socket.disconnect()
        })  
        
        //监听断开连接事件
        socket.on('disconnect',(data)=>{
          console.log('断开连接')
        })

        socket.on('users',(data)=>{
            this.users=data
        })

        socket.on('unreadmsg',(data)=>{
          data.forEach((item,index)=>{
            //设置未读的红点
            //将聊天的内容分别添加到本地的存储
            //将from/to改为有头像的
            item.from=this.usersObj[item.from];
            item.to=this.usersObj[item.to];
          this.unreadlist.push(item.from);
          let strKey = 'chat-user-'+this.$root.me.username+'-'+item.from.username;
          //console.log(strKey);
          //先解析本地存储的数据，再添加
          localStorage[strKey]=localStorage[strKey]?localStorage[strKey]:'[]';
          let newArr=JSON.parse(localStorage[strKey]);
          newArr.push(item);
          localStorage[strKey]=JSON.stringify(newArr);
          })
         
        })
        socket.on('accept',(msg)=>{
          
            if((this.ischat==true&&msg.from.username==this.touser.username)||(msg.to.username==this.touser.username&&msg.to.isgroup=='true')){
                   this.newmsg=msg
            }
            else{
              let strKey = 'chat-user-'+this.$root.me.username+'-'+msg.from.username;
          //console.log(strKey);
          //先解析本地存储的数据，再添加
          localStorage[strKey]=localStorage[strKey]?localStorage[strKey]:'[]';
          let newArr=JSON.parse(localStorage[strKey]);
          newArr.push(msg);
          localStorage[strKey]=JSON.stringify(newArr);
          this.unreadlist.push(msg.from.username)
            }
        })
   
  },
  methods: {
    chooseUser:function(user){
      this.touser=user;
      this.ischat=true;
      //已读，消除红点
      let index= this.unreadlist.indexOf(user.username);
      this.unreadlist.splice(index,1);
    },
    closeChat:function(){
        this.ischat=false
    }
  },
  computed: {
    usersObj:function(){
      let obj = {};
      this.users.forEach((item,index)=>{
              obj[item.username]=item;
      })
      return obj;
    }
  },
}
</script>

<style >
*{
  margin: 0;
  padding: 0;
}
</style>
