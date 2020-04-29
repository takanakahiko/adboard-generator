<template>
  <div>
    <section class="hero is-light">
      <div class="hero-body has-text-centered">
        <img src="~/assets/logo.png" class="logo" />
      </div>
    </section>
    <section class="section">
      <div class="container content is-centered has-text-centered">
        <p>背景の色を決めましょう</p>
        <client-only>
          <chrome-picker v-model="colors" disableAlpha disableFields class="max400"></chrome-picker>
        </client-only>
      </div>
      <div class="container content is-centered has-text-centered">
        <ImageSelector label="色背景におく画像を選んでください" v-model="image1DataURL" class="max400" />
      </div>
      <div class="container content is-centered has-text-centered">
        <ImageSelector label="白背景におく画像を選んでください" v-model="image2DataURL" class="max400" />
      </div>
    </section>
    <section class="section">
      <div class="container content is-centered has-text-centered">
        <p>こんな感じでどうでしょうか？</p>
        <canvas width="1200" height="800" class="canvas" ref="canvas"></canvas>
      </div>
    </section>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import ImageSelector from "~/components/ImageSelector.vue";
import { Chrome } from "vue-color";

export default Vue.extend({
  name: "HomePage",

  components: {
    ImageSelector,
    "chrome-picker": Chrome
  },

  data() {
    return {
      image1DataURL: null,
      image2DataURL: null,
      colors: {
        rgba: { r: 255, g: 0, b: 0, a: 1 }
      }
    } as {
      image1DataURL: string | null;
      image2DataURL: string | null;
      colors: any | null;
    };
  },

  watch: {
    async colors() {
      await this.draw();
    },
    async image1DataURL() {
      await this.draw();
    },
    async image2DataURL() {
      await this.draw();
    }
  },

  methods: {
    async draw() {
      const ctx = (this.$refs.canvas as HTMLCanvasElement).getContext("2d");
      if (!ctx) return;
      ctx.fillStyle = `rgb(255,255,255)`;
      ctx.fillRect(0, 0, 1200, 800);
      if (this.image1DataURL && this.image2DataURL && this.colors) {
        const image1 = await this.getImage(this.image1DataURL);
        const image2 = await this.getImage(this.image2DataURL);
        const colCount = 5;
        const rowCount = 6;
        const cellWidth = 1200 / colCount;
        const cellHeight = 800 / rowCount;
        ctx.fillStyle = this.colors.hex;
        for (let i = 0; i <= colCount; i++) {
          for (let j = 0; j <= rowCount; j++) {
            const x = cellWidth * i;
            const y = cellHeight * j;
            const imageX = x + 10;
            const imageY = y + 10;
            const imageW = cellWidth - 20;
            const imageH = cellHeight - 20;
            if ((i + j) % 2 == 0) {
              ctx.fillRect(x, y, cellWidth, cellHeight);
              ctx.drawImage(image1, imageX, imageY, imageW, imageH);
            } else {
              ctx.drawImage(image2, imageX, imageY, imageW, imageH);
            }
          }
        }
        console.log("書いた");
      }
    },
    async getImage(value: string): Promise<HTMLImageElement> {
      return new Promise((resolve, reject) => {
        const image = new Image();
        image.onload = () => resolve(image);
        image.src = value;
      });
    }
  }
});
</script>

<style scoped>
.logo {
  width: 80%;
  max-width: 400px;
}

.max400 {
  width: 80%;
  max-width: 400px;
  margin: 0 auto;
}

.canvas {
  width: 80%;
}
</style>