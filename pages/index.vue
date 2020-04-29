<template>
  <div>
    <section class="hero is-light">
      <div class="hero-body has-text-centered">
        <img src="~/assets/logo.png" class="logo" />
        <div class="links">
          <a href="https://twitter.com/takanakahiko" target="_blank" class="button--green">作った人</a>
          <a
            href="https://github.com/takanakahiko/adboard-generator"
            target="_blank"
            class="button--grey"
          >GitHub</a>
        </div>
      </div>
    </section>
    <section class="section">
      <div class="container content is-centered has-text-centered">
        <p>背景の色を決めましょう</p>
        <client-only>
          <chrome-picker v-model="colors" disableAlpha class="max400"></chrome-picker>
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
        <div v-if="image1DataURL && image2DataURL" class="content">
          <p>こんな感じでどうでしょうか？</p>
          <div class="links">
            <button @click="download" class="button--green">いいよ（ダウンロード）</button>
            <a :href="dameLink" target="_blank" class="button--grey">だめだよ</a>
          </div>
        </div>
        <canvas width="2400" height="1600" class="canvas" ref="canvas"></canvas>
      </div>
    </section>
    <footer class="footer">
      <div class="content has-text-centered">
        <p>
          <strong>Adboard Generator</strong> by
          <a href="https://twitter.com/takanakahiko">takanakahiko</a>.
        </p>
        <p>© Copyright takanakahiko All rights reserved.</p>
      </div>
    </footer>
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
        hex: "#FF0000"
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
      ctx.fillRect(0, 0, ctx.canvas.width, ctx.canvas.height);
      if (this.image1DataURL && this.image2DataURL && this.colors) {
        const colCount = 6;
        const rowCount = 5;
        const cellWidth = ctx.canvas.width / colCount;
        const cellHeight = ctx.canvas.height / rowCount;

        const image1 = await this.getImage(this.image1DataURL);
        const image2 = await this.getImage(this.image2DataURL);
        const info1 = this.getImageMeta(image1, cellWidth, cellHeight, 10);
        const info2 = this.getImageMeta(image2, cellWidth, cellHeight, 10);

        ctx.fillStyle = this.colors.hex;
        for (let i = 0; i <= colCount; i++) {
          for (let j = 0; j <= rowCount; j++) {
            const x = cellWidth * i;
            const y = cellHeight * j;
            if ((i + j) % 2 == 0) {
              ctx.fillRect(x, y, cellWidth, cellHeight);
              const imageX = x + info1.ofsetX;
              const imageY = y + info1.ofsetY;
              const imageW = info1.scaledW;
              const imageH = info1.scaledH;
              ctx.drawImage(image1, imageX, imageY, imageW, imageH);
            } else {
              const imageX = x + info2.ofsetX;
              const imageY = y + info2.ofsetY;
              const imageW = info2.scaledW;
              const imageH = info2.scaledH;
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
    },
    getImageMeta(
      image: HTMLImageElement,
      cellWidth: number,
      cellHeight: number,
      padding: number
    ) {
      const maxW = cellWidth - padding * 2;
      const maxH = cellHeight - padding * 2;
      const scale = Math.min(maxW / image.width, maxH / image.height);
      const scaledW = image.width * scale;
      const scaledH = image.height * scale;
      return {
        ofsetX: (maxW - scaledW) / 2 + padding,
        ofsetY: (maxH - scaledH) / 2 + padding,
        scaledW,
        scaledH
      };
    },
    download() {
      const canvas = this.$refs.canvas as HTMLCanvasElement;
      const link = document.createElement("a");
      link.href = canvas.toDataURL("image/png");
      link.download = "adboard.png";
      link.click();
    }
  },

  computed: {
    dameLink() {
      const text =
        "@takanakahiko Adboard Generator がダメだったのでもっと頑張ってください。";
      return `https://twitter.com/intent/tweet?text=${encodeURIComponent(
        text
      )}`;
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

.button--green {
  display: inline-block;
  border-radius: 4px;
  border: 1px solid #3b8070;
  color: #3b8070;
  text-decoration: none;
  padding: 10px 30px;
}
.button--green:hover {
  color: #fff;
  background-color: #3b8070;
}
.button--grey {
  display: inline-block;
  border-radius: 4px;
  border: 1px solid #35495e;
  color: #35495e;
  text-decoration: none;
  padding: 10px 30px;
  margin-left: 15px;
}
.button--grey:hover {
  color: #fff;
  background-color: #35495e;
}
</style>