<template>
  <div class="board-compose-container" @click="closeCompose">
    <div class="board-compose">
      <div
        class="board-compose-preview"
        :style="{
          backgroundColor: selectColor,
          backgroundImage: `url(${selectImg})`,
        }"
      >
       <span class="close material-icons-outlined close-compose" @click="close">close</span>
        <el-input
          placeholder="Add board title"
          v-model="board.title"
           @keyup.enter.native="createBoard"
        ></el-input>
        <el-button
          @click="createBoard"
          type="primary"
          :disabled="!board.title"
          class="compose-btn"
          >Create board</el-button
        >
        <el-button @click="renderColors" type="primary" class="compose-btn change-colors"
          >Change Colors</el-button
        >
      </div>
      <section class="backgrounds">
        <div class="board-compose-imgs">
          <span
            v-for="(img, idx) in imgs"
            :key="idx"
            :style="{ backgroundImage: `url(${img})` }"
            @click="setimg(img)"
          ></span>
        </div>
        <div class="board-compose-bgcs">
          <span
            v-for="(color, idx) in colors"
            :key="idx"
            :style="{ backgroundColor: color }"
            @click="setBcg(color)"
          ></span>
        </div>
      </section>
    </div>
  </div>
</template>

<script>
import { boardService } from "../services/board.service.js";
import { utilService } from "../services/util.service.js";
export default {
  data() {
    return {
      board: boardService.getEmptyBoard(),
      colors: [],
      imgs: [
        "https://images-na.ssl-images-amazon.com/images/S/sgp-catalog-images/region_US/u8lua-4AD76J88AJT-Full-Image_GalleryBackground-en-US-1585673473334._RI_.jpg",
        "https://wallpaperaccess.com/full/1146927.jpg",
        "https://wallpaperaccess.com/full/109672.jpg",
        "https://wallpaperaccess.com/full/1378654.jpg",
        "https://wallpaperaccess.com/full/2348657.jpg",
        "https://images.wunderstock.com/Bodies-of-Water-Near-Mountain-At-Starry-Night_u9zZpYBztI57_1600.jpeg",
        "https://images8.alphacoders.com/926/thumb-1920-926492.jpg",
        "https://i.pinimg.com/originals/45/b3/c8/45b3c8edad36060ad7908ae64b10b87d.jpg",
        "https://cdn.wallpapersafari.com/23/23/bq9Hw7.jpg",
      ],
      selectImg: "https://wallpaperaccess.com/full/109672.jpg",
      selectColor: "",
    };
  },
  created() {
    this.renderColors();
  },
  methods: {
    renderColors() {
      this.colors = [];
      for (var i = 0; i < 9; i++) {
        this.colors.push(utilService.getRandomColor());
      }
    },
    closeCompose(ev) {
      if (ev.target.className === "board-compose-container") {
        this.$emit("closeCompose");
      }
    },
    async createBoard() {
      this.board.style = {
        "background-color": this.selectColor,
        "background-image": `url(${this.selectImg})`,
      };
      this.board.createdAt = Date.now();
      try {
        const loggedUserId = this.$store.getters.loggedinUser._id;
        const miniLoggedInUser = await this.$store.dispatch({
          type: "getMiniUser",
          userId: loggedUserId,
        });
        this.board.createdBy = miniLoggedInUser;
        this.board.members = [miniLoggedInUser];
        this.$emit("addBoard", this.board);
      } catch (err) {
        console.log("failed to get miniLoggedInUser", err);
      }
    },
    close() {
      this.$emit("closeCompose");
    },
    setBcg(color) {
      this.selectImg = "";
      this.selectColor = color;
      this.board.style = { "background-color": color };
    },
    setimg(img) {
      this.selectColor = "";
      this.selectImg = img;
      this.board.style = { "background-image:": `url(${img})` };
    },
  },
  destroyed() {
    this.colors = [];
  },
};
</script>