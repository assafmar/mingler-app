<template>
  <transition name="fade">
  <!--<div v-if="user">-->
  <div v-if="user" class = "main">
    <div id="browse-div" class="browse flex-center" ref="playground">
<!--v-if="drageVals.showUser && nextUser"-->
<!--{{nextUser}}-->
       <!--//==================================================-->
          <div  class="img-frame next-user"  >
                <div class="img-container">
                  <img v-if="nextUser && nextUser.photos[0]" 
                        :src="nextUser.photos && nextUser.photos[0]">
                </div>
                <div class="user-details" >
                      <h4 class = "photo-txt">{{ nextUser.name }}, {{ newDate - nextUser.birth }}</h4>
                </div>
                <!--<div class="description" v-show="expand">
                      <h4>{{ nextUser.name }}, 1{{ newDate - nextUser.birth }}nextUser</h4>
                      <p> {{nextUser.description}}
                        <div class="expand">
                          <p @click="expand = !expand">
                          </p>
                        </div>
                </div>-->

        </div>
       
       <!--//===========================================:user='currUser'=======-->
       <transition name="fade">
        <div id="curr-user-frame" v-if="drageVals.showUser "   class="img-frame curr-user" >
              
               
                <div class="img-container"
              @mousemove="touchMove"  @touchmove="touchMove" 
              @mousedown="dragModeTrue" @mouseup="dragModeFalse" 
               @touchstart ="dragModeTrue" @touchend ="dragModeFalse" 
                
                
                >
                  <!--<transition name="fade">-->
                  <div v-if="drageVals.msg" id="v-like-tag" :class="vClass()">{{drageVals.msg}} </div>
                    <!--</transition>-->
                          <img v-if="currUser.photos"  :src="currUser.photos && currUser.photos[0]">
                    <!--<p @click="expand = !expand">
                              <!--<md-icon>keyboard_arrow_down</md-icon>-->
                    </p>-->
                </div>
                
                <div class="user-details" @click="expand = !expand">
                <!--<div class="user-details" @click.stop="showUserDetails">-->
                      <h4 class = "photo-txt">{{ currUser.name }}, {{ newDate - currUser.birth }}</h4>
                  <!--<div class="expand" @click.stop="expand = !expand">-->
                </div>
                  <div class="expand" >
                    <transition name="slide-fade">
                          <div class="description" v-show="expand" @click="expand = !expand">
                                dhsbt<br>rtvhtrvsyr<br>yceywecy<br>we4tr3tq<br>34tq4t
                                <h4>{{ user.name }}, {{ newDate - user.birth }}</h4>
                                <p> {{user.description}}</p>                          
                          </div>
                    </transition>
                
                </div>
        </div>
      </transition>
       <!--//==================================================-->

    </div>
  
    <!--<section class="actions" v-if="!newMatch">-->
    <section class="actions">
      <a href="#" @click.prevent="launchLikeAction($event, 'left')">
        <md-icon class="material-icons md-size-2x dislike">highlight_off</md-icon>
      </a>
      <a @click.prevent="launchLikeAction($event, 'right')">
        <md-icon class="material-icons md-size-2x like heart">favorite</md-icon>
      </a>
    </section>
    
    <transition name="fade">
          <div v-if="newMatch" transition="fade"  class="match-popup">
            <h1>Congratulations! </h1>
            <h1> You have a NEW MATCH! </h1>
            <img v-if="newMatch.photos" class="popup-image" :src="this.newMatch.photos[0]"></img>
            <span>You and {{this.newMatch.name}} like each other</span>
            <div class="popup-buttons">
              <el-button class="button" @click="closePopup">CLOSE</el-button>
              <el-button class="button" @click="viewMatches">View Matches</el-button>
            </div>
          </div>
    </transition>


  </div>
  </transition>

</template>

<script>
import { LOG_IN } from '../store/store'
import { LOG_OUT } from '../store/store'
import { SND_MSG } from '../store/store'
import { GET_BROWSED } from '../store/store'
import { ADD_USER } from '../store/store'
import { LIKE } from '../store/store'
import { RESTART_USERS } from '../store/store'
import { GET_USER } from '../store/store'
import Vue from 'vue'
import Sortable from 'vue-sortable'

// Vue.use(Sortable)

export default {
  name: 'browse',
  data() {
    return {
      msg: 'Browse screen',
      expand: false,
      newMatchFlag: false,
      newDate: 2017,
      currentId: 'TBD - need to grab ID from click',
      userIdx:0,
      currUser: '',
      nextUser: '',
      
      drageVals:{
        dragStatus:'init',
        initialLeft:null,
        initialTop:null,
        initialMouseX: null,
        startX:null,
        startY:null,
        msg:'',
        totalXDist : 0,
        parentEl:null,
        frameEl :null,
        parentWidth:null,
        diff:50,
        xPercent:0,
        showUser : true,
        isDraged:false,
        opacity:1,
        rotate:0
      }
    }
  },
  mounted() {
    // this.users = this.$store.getters.fetchUsersBrowsed;
    // this.drageVals.frameEl =  document.getElementById("curr-user-frame");
    // this.drageVals.parentEl =  document.getElementById("browse-div");

    // this.pushUsers();
    var that = this;
      setTimeout(function() {
        that.drageVals.frameEl =  document.getElementById("curr-user-frame");
        that.drageVals.parentEl =  document.getElementById("browse-div");
        that.initEl()
        that.pushUsers();
      }, 50);
  },
  created() {
    console.log(this.nextUser, 'sdfsdfsdfsddsfsf')
    this.$store.dispatch({ type: GET_BROWSED })
    .then(x => console.log(x));
    console.log('browse: created - after GET_BROWSED');
    this.$router.push('Browse');
        this.pushUsers();
    this.users = this.$store.getters.fetchUsersBrowsed;
    
  },
  computed: {
    users() {
      var users11 = this.$store.getters.fetchUsersBrowsed;
      console.log('browse: computed - users:', users11);

      return users11;
    },
    user() {
      return this.currUser;
    },

    newMatch() {
      return this.newMatchFlag && this.$store.getters.fetchLastMatch;
    }
  },

  methods: {
    showUserDetails(e){
       e.preventDefault();
        // this.expand = !this.expand;
        console.log('rrrrrrrrrrrrrrrrrrrrrrrr')
    },
    vClass() { 
      var txt = (Math.abs(this.drageVals.xPercent ==0 ))? '': 'v-like-action'
      var str =  (this.drageVals.xPercent >0) ?  'v-like-action v-like' :  'v-like-action v-dislike';
        console.log('vClass',str )
      return str
      // return (this.drageVals.xPercent >0) ?  'v-like-action v-like' : 'v-like-action v-dislike';
    },

    pushUsers(){
        console.log('pushUsers.users',this.users )
      this.userIdx  = (this.userIdx === this.users.length - 1) ? 0 : this.userIdx + 1;
      this.currUser = (this.nextUser)? this.nextUser : this.users[0] ;
      this.nextUser = this.users[this.userIdx];
    },
    dragModeTrue(e){
        console.log('dragModeTrue.drageVals.dragStatus1', this.drageVals.dragStatus )
       e.preventDefault();
            this.initEl()
            
            if (this.drageVals.dragStatus ==='init'){     //only on first round  
                console.log('dragModeTrue.1.5.e.path[2].offsetTop',e.path[2].offsetTop )
                this.drageVals.startX = this.drageVals.frameEl.offsetLeft;
            } 
                if(e.clientX){                //big screen
                    this.initialMouseX = e.clientX ;
                    this.initialMouseY = e.clientY ;
                }else{                        //mobile
                    this.initialMouseX = e.changedTouches[0].clientX;
                    this.initialMouseY = e.changedTouches[0].clientY;
            }
            this.drageVals.parentWidth =  this.drageVals.parentEl.getBoundingClientRect().width;
            this.drageVals.dragStatus ='clicked';
          console.log('dragModeTrue.drageVals.dragStatus2', this.drageVals.dragStatus )
  },
    dragModeFalse(e){
        console.log('dragModeFalse.e.path[2].offsetLeft.left',  this.drageVals.frameEl.style.top )
            this.isDraged = false;
            this.showUser = false;
            if(this.drageVals.dragStatus ==='clicked'){
                this.slideHome(e);

            }

            this.drageVals.dragStatus ='unclicked'
        console.log('dragModeFalse.end' )
    },
    //===================================
    slideHome(e){
          console.log( '------------slideHome.211' );
            var t= 9;
            var vals = this.drageVals;
            // if(this.drageVals.dragStatus=== 'clicked') return;
            
            vals.frameEl.classList.add("slide-home")
            vals.frameEl.style.left =this.drageVals.startX +'px';
            vals.frameEl.style.opacity  =1;
            vals.frameEl.style.transform  =`rotate(0deg)`;
            this.drageVals.dragStatus ='unclicked';
            this.drageVals.msg ='';
            this.drageVals.xPercent =0;
            var that = this;
          setTimeout(function() {
            that.drageVals.frameEl.classList.remove("slide-home")
          }, 300);
            

    },
    initEl(){
          this.drageVals.frameEl =  document.getElementById("curr-user-frame");
          console.log('initEl',document.getElementById("curr-user-frame"));
          this.drageVals.parentEl =  document.getElementById("browse-div");
          if( this.drageVals.frameEl && !this.drageVals.startX){
              this.drageVals.startX = this.drageVals.frameEl.offsetLeft;
              this.drageVals.parentWidth = this.drageVals.parentEl.getBoundingClientRect().width;
        }
              console.log('88888888888888888888888888888888');
    },
    //===================================
    goHome(e){
          console.log( '------------goHome.224', this.drageVals.frameEl );
            if (!this.drageVals.frameEl)  this.initEl()

            this.drageVals.frameEl.classList.remove("fly-out")
            // e.path[2].classList.remove("slide-home")
            this.drageVals.frameEl.style.left =this.drageVals.startX +'px';
            // e.path[2].style.top =this.drageVals.initialTop +'px';
            this.drageVals.frameEl.style.opacity  =1;
            this.drageVals.frameEl.style.transform  =`rotate(0deg)`;
            var el = document.getElementById("v-like-tag");//v-like
            if(el){
                // el.classList.remove("v-like");
                // el.classList.remove("v-dislike");
            }
            this.drageVals.dragStatus ='unclicked';
            this.drageVals.msg ='';
    },
    //===================================
    flyOut(e,direction){
          // console.log( '------------flyOut.',e.path[2],this.drageVals.frameEl);
           
           
           if(!this.drageVals.parentWidth) this.initEl()
           this.drageVals.msg = '';
           var leftStr =this.drageVals.parentWidth +100 +'px'
           var  rotateStr= 30; 
            if  (direction ==='left'){
              leftStr = -this.drageVals.parentWidth+'px';
              rotateStr = -30;
            }
          var el = document.getElementById("curr-user-frame");
          el.classList.add("fly-out")
          var str = el.offsetLeft ;
          el.style.left =str + 'px';

          el.style.left =leftStr;
          el.style.transform  =`rotate(${rotateStr}deg)`;
          var ev = e
          var that = this;
          setTimeout(function() {
                  console.log('flyOut.setTimeout' );
                     el.classList.remove("fly-out")
                    that.goHome(e);
                    that.pushUsers();
          }, 500);

    },
    //===================================
    touchMove(e){
          if (this.drageVals.dragStatus ==='clicked'){
              console.log('touchMove.e' , e);
              this.followMouse(e);
          }
    },
    //===================================
    followMouse(e){
          console.log('followMouse.e' , e);
        
        var vals = this.drageVals;
        var el = e.target;
        vals.status = 'followMouse';
            if(e.clientX){
                    var dx = (e.clientX - this.initialMouseX);
            }else{
                    // var dx = (e.changedTouches[0].clientX - this.initialMouseX);
                    var dx = (e.changedTouches[0].clientX - this.initialMouseX);
            } 
            vals.frameEl.style.left = this.drageVals.startX + dx + 'px';

          vals.xPercent = (dx)/(this.initialMouseX);
          vals.opacity = (4* Math.abs(vals.xPercent))
          var el = document.getElementsByClassName("v-like-action")[0];
          if(el) el.style.opacity  =vals.opacity;
          console.log('vals.e.changedTouches[0].clientX :',
          vals.xPercent,
          e.changedTouches[0].clientX ,
          e.changedTouches[0].clientX - this.initialMouseX,
          this.initialMouseX,
          dx

          
          )
            
            
            
            vals.msg = (vals.xPercent >0)? 'Like':'Dislike';
          vals.rotate =70 * vals.xPercent
          vals.frameEl.style.transform  =`rotate(${  vals.rotate}deg)`;
          // e.path[2].style.opacity  =vals.opacity;
        
        console.log('followMouse.msg', vals.msg, vals.xPercent );
          if(Math.abs(vals.xPercent) >111){
              if(vals.xPercent >0){
                  this.flyOut(e,'right');
                console.log('======like=====');
                  this.userLike(e);
              }else{
                  this.flyOut(e,'left');
                  this.userDislike(e);
              }
              var that = this;
          }
    },
    //===================================

    onSwipe(sw) {
      console.log(sw);
    },
    moveToMatches() {
      console.log('Browse: move to MATCHES ')
      this.$router.push('Matches')
    },
    moveToEdit() {
      console.log('Browse: MOVE to edit')
      this.$router.push('Edit')
    },
    launchLikeAction(e,direction){
      this.initEl()
  // dragModeTrue(e)
      // this.dragModeTrue(e)
      this.flyOut(e,direction);
      this.drageVals.dragStatus = 'clicked';
      (direction === 'left')? this.userDislike(e):this.userLike(e);
    },
    userDislike(e) {
      if(this.drageVals.dragStatus != 'clicked') return;
      console.log('Browse: before DISLIKE! id:', this.userIdx, this.users.length)
      const msg = { id1: this.$store.state.user.currUser.id, id2: this.user.id, bul: false }
      this.$store.dispatch({ type: LIKE, data: msg })
      console.log('Browse:  DISLIKE! id:', this.userIdx, this.users.length)
      var el = document.getElementsByClassName("fly-out")[0];
      console.log('Browse:  DISLIKE.el:', el)
      this.drageVals.dragStatus = 'afterLikeEvent';
},
    userLike(e) {
      this.newMatchFlag = true;
      
      if(this.drageVals.dragStatus != 'clicked') return;
      console.log('Browse: before LIKE! id:', this.userIdx, this.users.length)
      const msg = { id1: this.$store.state.user.currUser.id, id2: this.user.id, bul: true }
      this.$store.dispatch({ type: LIKE, data: msg })
      console.log('Browse:  LIKE! id:', this.userIdx, this.users.length)
      
      var el = document.getElementsByClassName("fly-out")[0];
      console.log('Browse:  LIKE.el:', el)
      this.drageVals.dragStatus = 'afterLikeEvent';
  },
    viewMatches() {
      console.log('Browse: clicked on "VIEW MATCHES"')
      this.newMatchFlag = false;
      // var that  = this;
          this.moveToMatches();
    },
    closePopup() {
      console.log('Browse: clicked on "CLOSE POPUP"')
      this.newMatchFlag = false;
    }

  }
}
  

</script>

<style scoped lang="scss">

.v-like-action{
      position: absolute;
      top:20px;
      z-index: 5;
      font-size:3em;
      padding:0.3em;
      border-radius: 7px;
}
.v-like{
  color:green;
  border:2px solid green;
  left:20px;
}
.v-dislike{
  color:red;
  border:2px solid red;
  right:20px;

}

.fly-out{
  transition: .5s all;
}
.slide-home{
  transition: .3s all;
}
.main{
    overflow: hidden;
  }
  .browse{
    position: relative;
    overflow: hidden;
    justify-content: center;
    align-items: center;
    height: 75vh;
}
.flex-center{
    display: flex;
    justify-content: center;
    align-items: center;

}
.card {
  background-color: rgba(250, 230, 230, 0.9);
  overflow: hidden;
}
.user-details {
    position: relative;
    white-space: nowrap;
    width:100%;
    font-size:1.5em;
    // overflow: hidden;
    text-overflow: clip ellipsis;
    z-index: 6;
    background-color: #c4a6a6;
    height:3em;
    cursor: pointer;
    box-shadow:0 0 15px gray;
    padding-top:1em;
    // outline: 1px solid red;
    h4{
      margin-top:0;
    }
  }

.actions {
  top:100px;
  padding: {
    top: 0.5em;
    right: 2em;
    bottom: 1em;
    left:2em;
  }
  display: flex;
  justify-content: space-between;
  flex-wrap:wrap;
  background: lightgrey;
  .like {
        left: 2em;

    color: red;
    opacity: 0.9;
    cursor: pointer;
    &:hover {
      opacity: 1;
    }
  }
  .dislike {
    right: 2em;
    color: rgba(124, 1, 87, 1);
    opacity: 0.8;
    cursor: pointer;
    &:hover {
      opacity: 1;
    }
  }
}
.img-frame{
    // transition: all 0.2s;
    // background: lightgrey;
    position: absolute;
    top:10%;
    width: 85%;
    height: 70%;
    // margin:auto;
}
 .next-user{
  box-shadow:0 0 15px gray;
  z-index: 0;
  // background-color: green;
}
.img-container:active {
      cursor: -webkit-grabbing; 
      // cursor:-moz-grabbing;
  }
.img-container {
  cursor: -webkit-grab;
  margin:auto;
  // width: 20em;
  height: 100%;
  width:100%;
  overflow: hidden;
  margin-top: 0.5em;
  margin-bottom: 0.5em;
  position: relative;
  display:flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin:auto;
  z-index: 1;
  box-shadow:0 0 15px gray;
.photo-txt{
  font-size: 1em;
  color:red;
}
  img {
    position: absolute;
    margin: auto;
    min-height: 100%;
    // min-width: 100%;
    width:100%;
    overflow-y: hidden;
  //  left: -100%;
 //   right: -100%;
 //   top: -100%;
 //   bottom: -100%;
  }
}

.match-popup {
  position: fixed;
  top: 0;
  left: 0;
  background-color: gray;
  z-index: 1000;
  width: 100%;
  height: 100%;
  line-height: 3em;
  color: white;
  text-shadow: -1px -1px 2px rgba(89, 89, 89, 0.66); // background-color: lightseagreen;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  align-items: center;
}

.popup-image {
  max-width: 15em;
  max-height: 15em;
  ;
}

.popup-buttons {
  width: 100%;
  // position: absolute;
  // bottom:10vh;
  .button {
    font-family: 'Kurale', Helvetica, Arial, sans-serif;
    text-transform: uppercase;
    margin-bottom: 1.5em;
  }
}


.expand {
  cursor: pointer;
}

.fade-enter-active{
// .fade-enter-active{
  transition: opacity .5s
}
.fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ {
  opacity: 0
}
.el-hide{
  visibility:hidden;
}
.description {
  background: rgba(252, 217, 217, 1);
  position: absolute;
  display: block;
  width: 100%;
  bottom: 0;
  height:100%;
  z-index: 5;
  font-size:1.5em;
  box-shadow:0 0 15px gray;
  opacity:0.8;
  // outline: 1px solid red;
    overflow:hidden;

}

.slide-fade-enter-active {
  transition: all .5s ease;
    // height:100%;
// opacity:0;
}
.slide-fade-leave-active {
  transition: all .8s ;
}
.slide-fade-enter, .slide-fade-leave-to{
  // transform: translateY(-7em);  
  height:0;
  overflow:hidden;
}

.slide-fade-down-enter-active {
  transition: all .5s ease;
    height:0;
}
.slide-fade-down-leave-active {
    height:0;
  transition: all .8s ;
}


* {
  // outline: 1px solid red;
}
</style>
 