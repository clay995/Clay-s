<template>
    <div class="userlist" v-if="$root.me!=null">
         <div class="nav">
             <div class="headerimg" :class="{online:islogin}">
                 <img  :src="$root.me.headerimg" alt="">
             </div>
             <div class="title">消 息</div>
             <div class="headerimg"></div>
         </div>
         <div class="users">
             <div @click="chooseUser(item)" class="useritem" v-for="(item,index) in friends" :key="index">
                 <div class="left" :class="{online:item.isonline=='true',unread:unreadlist.indexOf(item.username)!=-1}">
                   <img :src="item.headerimg" alt="">
                </div>
                <div class="right">
                    <span class="username"> {{item.username}}</span>
                       <span class="msg">

                       </span>
                    
                </div>
             </div>
         </div>
    </div>
</template>
<script>
export default {
    props:['islogin','users','chooseUser','unreadlist'],
    computed: {
        friends:function(){
            if(this.$root.me==null){
                return [];
            }
            else{
            let username=this.$root.me.username;
             return   this.users.filter((item,index)=>{
                 return item.username!=username;
             
            })
            }
        }
    },
}
</script>
<style scoped>
.unread{
    position: relative;
}
.unread::before{
     position: absolute;
     content: "";
     display: block;
     width:10px;
     height: 10px;
     border-radius: 5px;
     background: red;
     bottom: 5px;
     right: 5px;
}
.useritem .left{
    filter: grayscale(1);
}
.headerimg{
    height: 50px;
    width: 50px;
    filter: grayscale(1);
    margin: 0 10px;
}
.online{
    filter: grayscale(0) !important;
}
.nav{
    height: 80px;
    width: 100vw;
    background: skyblue;
    display: flex;
    align-items: center;
    justify-content: space-between;
}
.nav .title{
    font-size: 18px;
    font-weight: 900;
}
.headerimg img{
    height: 50px;
    width: 50px;
    border-radius: 50%;
}
.useritem{
    display: flex;
    height: 80;
    background: #eee;
    border-bottom: 1px solid #ccc;
    align-items: center;
    padding: 0 10px;
}
.useritem .left img{
    width: 60px;
    height: 60px;
    border-radius: 30px;
}
.useritem .right{
    padding: 0 10px;
}
</style>