<template>
  <!--<div v-if="user">-->
  <div v-if="user" class = "main">
    <div id="browse-div" class="browse flex-center" ref="playground">

       <!--//==================================================-->
          <div v-if="drageVals.showUser && nextUser"  :user='nextUser' class="img-frame next-user"  >
                <div class="img-container">
                  <img v-if="nextUser.photos" 
                        :src="nextUser.photos && nextUser.photos[0]">
                </div>
                <div class="user-details">
                      <h4 class = "photo-txt">{{ nextUser.name }}, {{ nextUser.age }}</h4>
                    <div class="description" v-show="expand">
                      <h4>{{ nextUser.name }}, 1{{ newDate - nextUser.birth }}nextUser</h4>
                      <p> {{nextUser.description}}
                        <div class="expand">
                          <p @click="expand = !expand">
                            <md-icon>keyboard_arrow_down</md-icon>
                          </p>
                        </div>
                    </div>
                </div>
        </div>
       
       <!--//==================================================-->
       <transition name="fade">
        <div v-if="drageVals.showUser "  :user='currUser' class="img-frame curr-user" 
              
              @mousemove="touchMove"  @touchmove="touchMove" 
              @mousedown="dragModeTrue" @mouseup="dragModeFalse" 
               @touchstart ="dragModeTrue" @touchend ="dragModeFalse" >
                <div class="img-container">
                  <!--<img v-if="users[userIdx].photos" :src="users[userIdx].photos && users[userIdx].photos[userIdx]">-->
                  <img  class="presented-photo" :src="currUser.photos && currUser.photos[0]">
                </div>
                <div class="user-details">
                      <h4 class = "photo-txt">{{ currUser.name }}, {{ currUser.age }}</h4>
                    <div class="description" v-show="expand">
                      <h4>{{ currUser.name }}, {{ newDate - currUser.birth }}</h4>
                      <!--<p> Lorem ipsum dolor sit amet, consectetur adipisicing elit. Optio itaque ea, nostrum odio. Dolores, sed accusantium quasi non, voluptas eius illo quas, saepe voluptate pariatur in deleniti minus sint. Excepturi. </p>-->
                      <p> {{currUser.description}}
                        <div class="expand">
                          <p @click="expand = !expand">
                            <md-icon>keyboard_arrow_down</md-icon>
                          </p>
                        </div>
                    </div>
                </div>
        </div>
      </transition>
       <!--//==================================================-->

    </div>
  
    <!--<section class="actions" v-if="!newMatch">-->
    <section class="actions">
      <a href="#" @click.prevent="userDislike">
        <md-icon class="material-icons md-size-2x dislike">highlight_off</md-icon>
      </a>
      <a @click.prevent="userLike">
        <md-icon class="material-icons md-size-2x like heart">favorite</md-icon>
      </a>
    </section>
    
    <transition name="fade">
          <div v-if="newMatch" transition="fade"  class="match-popup">
            <h1>Congratulations! </h1>
            <h1> You have a NEW MATCH! </h1>
            <img class="popup-image" :src="this.newMatch.photos[0]"></img>
            <h2>You and {{this.newMatch.name}} like each other</h2>
            <div class="popup-buttons">
              <el-button class="button" @click="closePopup">CLOSE</el-button>
              <el-button class="button" @click="viewMatches">View Matches</el-button>
            </div>
          </div>
    </transition>


  </div>
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
        
        totalXDist : 0,
        parentEl:null,
        frameEl :null,
        parentWidth:0,
        diff:50,
        // xLeft:0,
        // oldX:0,
        // xDiff:0,
        // xRight:0,
        // xMid:0,
        xPercent:0,
        // xDistFromInitial:0,
        // xRightParent:0,
        // distToLeft: 0,
        // distToRight: 0,
        showUser : true,
        isDraged:false,
        opacity:1,
        rotate:0
      }
    }
  },
  mounted() {

  },
  created() {
    this.$store.dispatch({ type: GET_BROWSED });
    console.log('browse: created - after GET_BROWSED');
    this.$router.push('Browse');
    this.users = this.$store.getters.fetchUsersBrowsed;
    this.pushUsers();
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
    pushUsers(){
        console.log('pushUsers.users',this.users )
      this.userIdx  = (this.userIdx === this.users.length - 1) ? 0 : this.userIdx + 1;
      this.currUser = (this.nextUser)? this.nextUser : this.users[0] ;
      this.nextUser = this.users[this.userIdx];
    },
    dragModeTrue(e){
        console.log('dragModeTrue.drageVals.dragStatus1', this.drageVals.dragStatus )
       e.preventDefault();
        // this.calculateCardPos(e);
            if (this.drageVals.dragStatus ==='init'){     //only on first round  
                console.log('dragModeTrue.1.5.e.path[2].offsetTop',e.path[2].offsetTop )
                this.drageVals.startX = e.path[2].offsetLeft;
                // this.startY = e.path[2].offsetTop;
            } 
                if(e.clientX){                //big screen
                    this.initialMouseX = e.clientX ;
                    this.initialMouseY = e.clientY ;
                }else{                        //mobile
                    this.initialMouseX = e.changedTouches[0].clientX;
                    this.initialMouseY = e.changedTouches[0].clientY;
            }
            // this.drageVals.isDraged = true;frameEl
            this.drageVals.frameEl =  document.getElementById("browse-div");
            this.drageVals.parentEl =  document.getElementById("browse-div");
            this.drageVals.parentWidth =  this.drageVals.parentEl.getBoundingClientRect().width;

            this.drageVals.dragStatus ='clikced';
          console.log('dragModeTrue.drageVals.dragStatus2', this.drageVals.dragStatus )
  },
    dragModeFalse(e){
        console.log('dragModeFalse.e.path[2].offsetLeft.left',  e.path[2].style.top )
            this.isDraged = false;
            this.showUser = false;
            if(this.drageVals.dragStatus ='clicked'){
              // e.path[2].classList.add("fly-out")
                this.slideHome(e);

            }

            this.drageVals.dragStatus ='unclicked'
        console.log('dragModeFalse.end' )
    },
    //===================================
    slideHome(e){
          console.log( '------------goHome.216' );
            e.path[2].classList.add("slide-home")
            e.path[2].style.left =this.drageVals.startX +'px';
            e.path[2].style.opacity  =1;
            e.path[2].style.transform  =`rotate(0deg)`;
            this.drageVals.dragStatus ='unclicked';
          setTimeout(function() {
            e.path[2].classList.remove("slide-home")
          }, 300);
            

    },
    //===================================
    goHome(e){
          console.log( '------------goHome.216' );
            e.path[2].classList.remove("fly-out")
            e.path[2].style.left =this.drageVals.startX +'px';
            // e.path[2].style.top =this.drageVals.initialTop +'px';
            e.path[2].style.opacity  =1;
            e.path[2].style.transform  =`rotate(0deg)`;
            this.drageVals.dragStatus ='unclicked'
    },
    //===================================
    flyOut(e,direction){
          // console.log( '------------flyOut.');
           var str =this.drageVals.parentWidth + 400 +'px'
            if  (direction ==='left'){
              str = '-400px';
            }
          // console.log( '------------flyOut.str', str);
          var ev = e
          var that = this;
          e.path[2].classList.add("fly-out")
          e.path[2].style.left =str;
          setTimeout(function() {
                  console.log('flyOut.setTimeout' );
                     ev.path[2].classList.remove("fly-out")
                    that.goHome(e);
                    that.pushUsers();

          }, 400);


    },
    //===================================
    touchMove(e){
          // console.log('touchMove.drageVals.dragStatus',this.drageVals.dragStatus );
          if (this.drageVals.dragStatus ==='clikced'){
              // this.calculateCardPos(e);
              this.followMouse(e);
          }

          // console.log('touchMove.e.target', e.target.x , e.target.clientWidth );
          // console.log(this.users[0] );
    },
    //===================================
    followMouse(e){
          // console.log('followMouse.e' , e);
        var vals = this.drageVals;
        var el = e.target;
      // vals.status = 'followMouse';
            if(e.clientX){
                    var dx = e.clientX - this.initialMouseX;
            }else{
                    var dx = e.changedTouches[0].clientX - this.initialMouseX;
                    // var dy = e.changedTouches[0].clientY - this.initialMouseY;
                    // e.path[2].style.top = this.startY + dy + 'px';
            } 
            e.path[2].style.left = this.drageVals.startX + dx + 'px';

          // vals.xPercent = (vals.xLeft - vals.initialLeft)/ (vals.initialLeft+  vals.diff)
          vals.xPercent = (dx)/(this.initialMouseX+  vals.diff);


          //  vals.opacity = 1-(Math.abs(vals.xPercent)/2)
          vals.rotate =70 * vals.xPercent
          e.path[2].style.transform  =`rotate(${  vals.rotate}deg)`;
          // e.path[2].style.opacity  =vals.opacity;
                
          if(Math.abs(vals.xPercent) >0.3){
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
    userDislike(e) {
      if(this.drageVals.dragStatus != 'clikced') return;
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
      
      if(this.drageVals.dragStatus != 'clikced') return;
      console.log('Browse: before LIKE! id:', this.userIdx, this.users.length)
      const msg = { id1: this.$store.state.user.currUser.id, id2: this.user.id, bul: true }
      this.$store.dispatch({ type: LIKE, data: msg })
      console.log('Browse:  LIKE! id:', this.userIdx, this.users.length)
      var el = document.getElementsByClassName("fly-out")[0];
      console.log('Browse:  LIKE.el:', el)
      this.drageVals.dragStatus = 'afterLikeEvent';
  },
    viewMatches() {
      console.log('Browse: clicked on "VEIW MATCHES"')
      this.newMatchFlag = false;
      this.moveToMatches();
    },
    closePopup() {
      console.log('Browse: clicked on "CLOSE POPUP"')
      this.newMatchFlag = false;
    }

  }
}
      // console.log('Browse: before LIKE! id:', this.userIdx, this.users.length)
      // const msg = { id1: this.$store.state.user.currUser.id, id2: this.user.id, bul: true }
      // this.drageVals.isDraged =false;
      // this.drageVals.showUser =false;
      // var that = this;
      // // e.path[2].classList.add("fly-out")
      // e.path[2].style.left ='-210px';

      
      // setTimeout(function() {
      //       that.drageVals.showUser =true;
      //       that.userIdx  = (that.userIdx === that.users.length - 1) ? 0 : that.userIdx + 1;
      //       that.nextUser = (that.nextUser === that.users.length - 1) ? 0 : that.userIdx + 1;
      //       // e.path[2].classList.remove("fly-out")
      // },1);
  

</script>

<style scoped lang="scss">
// .presented-photo{
//   min-height:100%;
//   width: auto;
// }

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
    height: calc(100vh - 150px);
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
    overflow: hidden;
    text-overflow: clip ellipsis;
    z-index: 5;
    background-color: gray;
  }

.actions {
  top:100px;
  padding: {
    top: 0.5em;
    left: 2em;
    right: 2em;
    bottom: 1em;
  }
  display: flex;
  justify-content: space-between;
  flex-wrap:wrap;
  background: lightgrey;
  .like {
    color: red;
    opacity: 0.9;
    cursor: pointer;
    &:hover {
      opacity: 1;
    }
  }
  .dislike {
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
    background: lightgrey;
    position: absolute;
    // margin:auto;
}
 .next-user{
  // opacity:0.8;
  // z-index: -1;
  // background-color: green;
}
.img-container {
  margin:auto;
  width: 18em;
  height: 20em;
  overflow: hidden;
  margin-top: 0.5em;
  margin-bottom: 0.5em;
  position: relative;
  display:flex;
  flex-direction: column;
  // justify-content: center; //
  // align-items: center;
.photo-txt{
  font-size: 1em;
  color:red;
}
  img {
    position: absolute;
   margin: auto;
    min-height: 100%;
  //  min-width: 100%;
  width:auto;
    left: -100%;
    right: -100%;
    top: -100%;
    bottom: -100%;
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
  .button {
    font-family: 'Kurale', Helvetica, Arial, sans-serif;
    text-transform: uppercase;
    margin-bottom: 1.5em;
  }
}

.description {
  background: rgba(252, 217, 217, 1);
  position: absolute;
  display: block;
  width: 100%;
  bottom: 0;
}

.expand {
  cursor: pointer;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity .01s
}
.fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ {
  opacity: 0
}
.el-hide{
  visibility:hidden;
}
</style>
 