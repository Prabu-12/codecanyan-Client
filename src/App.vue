<template>
  <fullscreen ref="fullscreen" @change="fullscreenChange">
    <v-app v-if="isLanding==false">
      <Navbar v-if="isVisible==true"/>
      <v-main :class="getBgColor">
        <v-container fluid>
          <router-view></router-view>        
        </v-container>
      </v-main>      
      <Loading/>
      <CheckPassword/> 
      <Footer v-if="isVisible==true"/>            
    </v-app>  
    <Landing v-else/> 
  </fullscreen>
</template>

<script>
import Navbar from './components/common/Navbar'
import Footer from './components/common/Footer'
import router from './router/index'
import Loading from './components/common/Loading'
import CheckPassword from '../src/views/user/checkPassword'
import Landing from '../src/views/signin/Landing.vue'

export default {
  name:'App',
  data(){
    return{
      fullscreen: false,
      opacity:0.9
    } 
  },
  components:{
    Navbar,
    Footer,
    Loading,
    CheckPassword,
    Landing
  },
  methods:{
    fullscreenChange (fullscreen) {
      this.fullscreen = fullscreen
    }
  },
  computed:{
    isLanding:function(){
      return this.$store.getters['dashboard/isLanding']
    },
    isVisible:function(){
      return this.$store.getters['dashboard/visible']
    },
    getBgColor:function(){
      return this.$store.getters['settings/bgColor']
    }
  },
  created(){
    this.$http.interceptors.response.use(undefined, function (err) {
      //console.log(err.response)
      if(err.response.status===401){
        router.push({name:'Unauthorized'})
      }else if(err.response.status===403){
        router.push({name:'Forbidden'})
      }else if(err.response.status===404){
        router.push({name:'NotFound'})
      }else{
        console.log(err)
      }
    })

    if(this.$route.path=='/'){
      this.$store.dispatch('dashboard/signOut')
    }
  }
}
</script>

<style>
 body{
    position: relative;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    height: 100%;
    width: 100%;
    overflow: auto;
  }
  .btnExcel {
    border: 1px solid black;
    background-color: #EEEEEE;
    color: black;
    padding: 3.2px 7.5px;
    font-size: 13px;
    border-radius: 5px;
    cursor: pointer;
  }
</style>