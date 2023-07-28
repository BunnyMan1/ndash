<template>
  <div class="media-list-display__root">
    <div class="media-list-display">
      <!-- PDFs Display -->
      <div
        :class="
          isSmallImage
            ? 'media-list-display__item_fit'
            : 'media-list-display__item'
        "
        v-for="(item, index) in documentItems"
        :key="index"
      >
        <a :href="item.url" target="_blank">
          <div class="media-list-display__item_top">
            <div>
              <i class="el-icon-paperclip"></i>
            </div>
          </div>
          <div class="media-list-display__item_bottom">
            <i
              class="el-icon-star-on media-list-display__item_primary"
              v-if="item.is_primary"
            ></i>
            <p class="media-list-display__item_label" v-if="item.display_label">
              {{ item.display_label }}
            </p>
            <p class="media-list-display__item_desc" v-if="item.description">
              {{ item.description }}
            </p>
            <p
              class="media-list-display__item_desc"
              v-if="item.uploaded_by_full_name"
            >
              {{ item.uploaded_by_full_name }}
            </p>
            <p class="media-list-display__item_desc" v-if="item.created_at">
              {{ outputDate(item.created_at) }}
            </p>
          </div>
        </a>
      </div>

      <!-- Non PDFs Display (Images) -->
      <div
        :class="
          isSmallImage
            ? 'media-list-display__item_fit'
            : 'media-list-display__item'
        "
        v-for="(item, index) in imageOrVideoItems"
        :key="'img' + index"
      >
        <div @click="openCarousel(index)">
          <div class="video-icon__main">
            <div class="media-list-display__item_top">
              <img
                v-if="item.is_image"
                :src="item.url"
                :id="isSmallImage ? 'attachment-image' : ''"
              />
              <video
                class="media-list-display__item"
                v-if="checkIfIsVideo(item.url)"
                :src="item.url"
              ></video>
            </div>
            <el-icon
              v-if="checkIfIsVideo(item.url)"
              class="el-icon-video-play video-icon"
            ></el-icon>
          </div>
          <div class="media-list-display__item_bottom">
            <i
              class="el-icon-star-on media-list-display__item_primary"
              v-if="item.is_primary"
            ></i>
            <p class="media-list-display__item_label" v-if="item.display_label">
              {{ item.display_label }}
            </p>
            <p class="media-list-display__item_desc" v-if="item.description">
              {{ item.description }}
            </p>
            <p
              class="media-list-display__item_desc"
              v-if="item.uploaded_by_full_name"
            >
              {{ item.uploaded_by_full_name }}
            </p>
            <p class="media-list-display__item_desc" v-if="item.created_at">
              {{ outputDate(item.created_at) }}
            </p>
          </div>
        </div>
      </div>
    </div>

    <el-dialog
      :visible.sync="dialogVisible"
      :close-on-click-modal="true"
      :close-on-press-escape="true"
    >
      <el-carousel
        height="70vh"
        :autoplay="false"
        trigger="click"
        :initial-index="startIndex"
        v-if="dialogVisible"
      >
        <!-- List of Carousel Items -->
        <el-carousel-item
          change
          v-for="(item, index) in imageOrVideoItems"
          :key="index"
        >
          <video
            v-if="checkIfIsVideo(item.url)"
            controls="true"
            controlsList="nodownload"
          >
            <source :src="item.url" type="video/mp4" />
          </video>
          <img v-else :src="item.url" />

          <!-- Open in new Tab Icon -->
          <div class="carousel-media-link">
            <a :href="item.url" target="_blank">
              <i class="el-input__icon el-icon-link"></i>
            </a>
          </div>
        </el-carousel-item>
      </el-carousel>
    </el-dialog>
  </div>
</template>

<script lang='ts'>
import Vue, { PropType } from "vue";
import { MediaLiteItem } from "~/store";

export default Vue.extend({
  data() {
    return {
      isDialog: true,
      dialogVisible: false,
      startIndex: 0,

      imageOrVideoExtensions: ["png", "jpeg", "jpg", "mp3", "mp4"],
      documentExtensions: [
        "pdf",
        "doc",
        "docx",
        "xls",
        "xlsx",
        "csv",
        "json",
        "svg",
      ],
    };
  },
  methods: {
    openCarousel(index: number) {
      this.startIndex = index;
      this.$nextTick(() => {
        this.dialogVisible = true;
      });
    },

    checkIfImageOrVideo(url: string): boolean {
      return this.imageOrVideoExtensions.some((x) => url?.includes("." + x));
    },

    checkIfDocument(url: string): boolean {
      return this.documentExtensions.some((x) => url?.includes("." + x));
    },

    checkIfIsVideo(url: string): boolean {
      return url.includes(".mp4") || url.includes(".mp3");
    },
  },

  computed: {
    itemsSorted(): MediaLiteItem[] {
      let itemsAgain = [...this.items];
      let sorted = itemsAgain.sort(
        (a, b) => a.display_order! - b.display_order!
      );
      return sorted;
    },

    imageOrVideoItems(): MediaLiteItem[] {
      return this.itemsSorted.filter((a) => this.checkIfImageOrVideo(a.url));
    },

    documentItems(): MediaLiteItem[] {
      return this.itemsSorted.filter((a) => this.checkIfDocument(a.url));
    },
  },

  props: {
    items: {
      type: Array as PropType<MediaLiteItem[]>,
      required: true,
    },

    isSmallImage: {
      type: Boolean,
      default: false,
    },
  },
});
</script>

<style lang="scss">
.el-carousel__item {
  display: flex;
  justify-content: center;

  * {
    height: 100%;
  }
}

.video-icon {
  position: absolute;
  left: 40%;
  top: 40%;
  font-size: 2rem;

  color: white;

  &__main {
    position: relative;
  }
}

.carousel-media-link {
  position: absolute;
  right: 2%;
  bottom: 0px;
  top: 0px;

  a {
    display: flex;
    width: 40px;
    height: 40px;
    align-self: center;
    justify-content: center;
    font-size: 1.2rem;
    text-decoration: none;
    border-radius: 50px;
    color: black;
    text-decoration: none;
    background-color: #dddddd;
    border: 1px solid #c0deed;
  }
}

.media-list-display__root {
  .el-dialog {
    width: auto;
    max-width: 80%;
    margin-top: 5vh !important;
  }
}

.media-list-display {
  display: flex;
  flex-wrap: wrap;
  margin-top: 16px;
  margin-bottom: 40px;
  align-items: flex-start;
  margin-bottom: 10px;
}

.media-list-display__item {
  display: flex;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12);
  flex-direction: column;
  align-items: stretch;
  cursor: pointer;
  min-width: 160px;
  max-width: 180px;

  &:not(:last-child) {
    margin-right: 32px;
  }
}

.media-list-display__item_fit {
  display: flex;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12);
  flex-direction: column;
  align-items: stretch;
  cursor: pointer;
  min-width: 120px;
  max-width: 140px;
  margin-top: 10px;
  &:not(:last-child) {
    margin-right: 30px;
  }
}

.media-list-display__item_top {
  line-height: 0;

  img {
    max-width: 100%;
    line-height: 0;
    max-height: 140px;
    height: 100%;
    object-fit: cover;
  }

  div {
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 20px;
    height: 140px;
    // background-color: green;
  }

  video,
  img,
  div {
    border-radius: 8px 8px 0 0;
  }
}

.media-list-display__item_bottom {
  // background-color: red;
  border-radius: 0 0 8px 8px;
  padding: 12px;
}

.media-list-display p:not(:last-child) {
  margin-bottom: initial;
}

// .media-list-display__item_primary {
//   color: red;
// }

.media-list-display__item_label {
  font-size: 15;
  font-weight: 500;
  color: darkslategray !important;
  word-break: break-all;
}

.media-list-display__item_desc {
  margin-top: 4px;
  color: gray !important;
  font-size: 12px !important;
}
</style>
