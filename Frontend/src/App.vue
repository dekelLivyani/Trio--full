<template>
  <div
    class="app" :class="isHomePageClass"
    :style="{
      backgroundColor: this.backgroundColor,
      backgroundImage: this.backgroundImg,
    }"
  >
    <app-header
      v-if="!isHome"
      @addBoard="addBoard"
      @setBackground="setBackground"
    />
    <router-view
     class="main"
      @setBackground="setBackground"
      :loggedinUser="loggedinUser"
      @setToPreviewEdit="setDarkWindow"
      @setDarkWindow="setDarkWindow"
      :darkWindow="darkWindow"
    />
    <user-msg />
    <div
      class="darkWindow"
      v-if="isDarkWindow"
      @click="closeDarkWindow"
    ></div>
  </div>
</template>


<script>
import appHeader from "@/cmps/app-header";
import userMsg from "./cmps/user-msg";
import { socketService } from "@/services/socket.service";

export default {
  components: {
    appHeader,
    userMsg,
  },
  data() {
    return {
      isHome: false,
      isHomePage:false,
      backgroundColor: "",
      backgroundImg:
        "https://images-na.ssl-images-amazon.com/images/S/sgp-catalog-images/region_US/u8lua-4AD76J88AJT-Full-Image_GalleryBackground-en-US-1585673473334._RI_.jpg",
      darkWindow: {
         editCard:false,
         deleteBoard:false,
      }
    };
  },
  async created() {
    try {
      socketService.setup();
      await this.$store.dispatch({ type: "loadUsers" });
      await this.$store.dispatch({ type: "loadBoards" });
    } catch (err) {
      console.log("ERROR cannot load users or boards");
    }
  },
  watch: {
    "$route.path": {
      immediate: true,
      handler() {
        const routes = ["/", "/login", "/signup"];
        if (routes.includes(this.$route.path)) {
           this.isHome = true;
           if (this.$route.path.includes("/") && !this.$route.path.includes("/login") &&
           !this.$route.path.includes("/signup")) this.isHomePage = true;
           else this.isHomePage = false
        }
        else this.isHome = false;
      },
    },
    watchedUser: {
      // immediate: true,
      deep: true,
      handler() {
         if(this.loggedinUser && this.watchedUser){
             if (this.loggedinUser._id === this.watchedUser._id) this.updateUserMentions(); 
         } 
      },
    },
  },
  computed: {
    loggedinUser() {
      return this.$store.getters.loggedinUser;
    },
    watchedUser() {
      return this.$store.getters.watchedUser;
    },
    isDarkWindow(){
      return (this.darkWindow.editCard || this.darkWindow.deleteBoard)
    },
    isHomePageClass(){
       return { 'is-home-page' : this.isHomePage }
    },
  },
  methods: {
    updateUserMentions() {
      this.loggedinUser.mentions = this.watchedUser.mentions;
    },
    async addBoard(board) {
      try {
        const newBoard = await this.$store.dispatch({
          type: "saveBoard",
          board,
        });
        this.$store.commit({ type: "setCurrBoard", board: newBoard });
        this.backgroundColor = board.style["background-color"];
        this.backgroundImg = board.style["background-image"];
        const activity = { txt: "created this board" };
        await this.$store.dispatch({ type: "addActivity", activity });
        this.$router.push(`/b/${newBoard._id}`);
      } catch (err) {
        console.log("ERROR cannot add board");
      }
    },
    setBackground(style) {
      this.backgroundColor = style["background-color"];
      this.backgroundImg = style["background-image"];
    },
    setDarkWindow(type,deff) {
      this.darkWindow[type] = deff;
    },
    closeDarkWindow(){
       for (const type in this.darkWindow) {
          this.darkWindow[type] = false;
       }
    }
  },
};
</script>