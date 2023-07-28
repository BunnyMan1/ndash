<template>
  <div class="media-list-insert">
    <div
      @click="addMedia"
      class="media-list-display__blank"
      v-if="data.length < limit"
    >
      <div class="media-list-display__item_top">
        <i class="el-icon-plus"></i>
      </div>
      <div class="media-list-display__item_bottom">
        <p class="media-list-display__item_label">
          Add {{ imageOnly ? "Image" : "Attachment" }}
        </p>
      </div>
    </div>

    <div v-if="data.length > 0" class="media-list-display__wrapper">
      <div
        :class="
          isSmallImage
            ? 'media-list-display__item_fit'
            : 'media-list-display__item'
        "
        v-for="(item, index) in data"
        :key="index"
        @click="addToEdit(item)"
        target="_blank"
      >
        <div class="media-list-display__item_top">
          <img
            v-if="item.is_image"
            :src="item.url"
            :id="isSmallImage ? 'attachment-image' : ''"
          />
          <div v-else>
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
        </div>
      </div>
    </div>
    <MediaInsertDialog
      :visible="dialogOpen"
      :editItem="editItem"
      @close="closeDialog"
      @submit="addFile"
      :imageOnly="imageOnly"
      :allowVideo="allowVideo"
      @uploadedFile="uploadedFile"
    />
  </div>
</template>

<script>
import MediaInsertDialog from "@/components/Media/MediaInsertDialog";

export default {
  components: {
    MediaInsertDialog,
  },
  data() {
    return {
      data: [],
      dialogOpen: false,
      editItem: null,
    };
  },

  methods: {
    submit() {
      return this.data;
    },

    addToEdit(data) {
      this.editItem = data;
      this.dialogOpen = true;
    },

    closeDialog() {
      this.editItem = null;
      this.dialogOpen = false;
    },

    addFile(item) {
      if (typeof item == "number") {
        let ind = this.data.findIndex((x) => x.id == item);
        if (ind != -1) this.data.splice(ind, 1);
      } else {
        if (!item || item.id === null) return;
        let ind = this.data.findIndex((x) => x.id == item.id);
        if (ind == -1) {
          console.log("Entered");
          this.$nextTick(() => this.data.push(item));
        } else this.data[ind] = item;
      }
      this.data.sort((a, b) => a.display_order - b.display_order);
      this.editItem = null;
      this.dialogOpen = false;
      this.$forceUpdate();
    },

    addMedia() {
      this.dialogOpen = true;
    },

    checkUpdate() {
      this.data = [];
      if (this.existing && this.existing.length > 0) {
        this.data = [...this.existing];
      }
    },

    uploadedFile(mediaId) {
      this.$emit("uploadedFile", mediaId);
    },
  },

  props: {
    limit: {
      type: Number,
      default: 16,
    },
    existing: {
      type: Array,
      default: null,
    },
    imageOnly: {
      type: Boolean,
      default: false,
    },
    allowVideo: {
      type: Boolean,
      default: false,
    },
    isSmallImage: {
      type: Boolean,
      default: false,
    },
  },
};
</script>

<style lang='scss'>
.media-list-insert {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-start;
  line-height: initial !important;
}

.media-list-display__wrapper {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-start;
}

.media-list-display__blank {
  display: flex;
  border-radius: 8px;
  flex-direction: column;
  align-items: stretch;
  cursor: pointer;
  min-width: 160px;
  max-width: 180px;
  // background-color: red;
  // border: 2px dashed green;
  margin-right: 32px;
  transition: 0.2s;
  font-weight: 500;

  // &:hover {
  //   // background-color: blue;
  // }

  .media-list-display__item_top {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 80px;

    i {
      font-size: 22px !important;
    }
  }

  .media-list-display__item_bottom {
    text-align: center;
    border-radius: 0 0 8px 8px;
    padding: 12px;
    background-color: transparent !important;
  }
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
    border-radius: 8px 8px 0 0;
    line-height: 0;
    max-height: 140px;
    height: 100%;
    object-fit: cover;
  }
  div {
    border-radius: 8px 8px 0 0;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 20px;
    height: 140px;
    // background-color: green;
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
  font-size: 15px;
  color: darkslategray !important;
}

#attachment-image {
  object-fit: cover;
  width: 140px;
  height: 120px;
}

.media-list-display__item_desc {
  margin-top: 4px;
  color: gray !important;
  font-size: 12px !important;
}
</style>
