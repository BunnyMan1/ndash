<template>
  <el-dialog
    title="Add Attachment"
    :visible="visible"
    @close="$emit('close')"
    append-to-body
  >
    <el-form :model="form">
      <el-form-item>
        <el-upload
          :auto-upload="false"
          :limit="1"
          :multiple="false"
          :on-remove="handleRemove"
          :on-change="handleChange"
          :accept="actualAccepted.join(', ')"
          ref="upload"
          action=""
          ><el-button slot="trigger" size="small" type="primary"
            >Select File</el-button
          >
          <el-button
            style="margin-left: 10px"
            size="small"
            type="success"
            :disabled="!file || uploadedMedia.id > 0"
            @click="submitUpload"
            :loading="uploadLoading"
            >Upload to server</el-button
          >
        </el-upload>
        <p
          v-if="uploadedMedia.id == 0"
          style="line-height: 20px; margin-top: 4px"
        >
          Upload Max Size: 25MB.<br />Allowed Extensions:
          {{
            imageOnly
              ? allowVideo
                ? "mp4"
                : "jpg, jpeg, png"
              : "pdf, docx, svg, doc, xlsx, xls, csv, json, jpg, jpeg, png, mp4"
          }}.
        </p>
        <a v-if="uploadedMedia.id > 0" :href="uploadedMedia.url" target="_blank"
          >Preview Media</a
        >
      </el-form-item>

      <div v-if="uploadedMedia && uploadedMedia.id > 0">
        <el-form-item label="Attachment ID">
          <el-input type="text" v-model="uploadedMedia.id" disabled></el-input>
        </el-form-item>

        <el-form-item label="Attachment URL">
          <el-input type="text" v-model="uploadedMedia.url" disabled></el-input>
        </el-form-item>

        <el-form-item label="Attachment Is Image">
          <el-switch v-model="uploadedMedia.is_image" disabled></el-switch>
        </el-form-item>

        <el-form-item label="Is Primary">
          <el-switch v-model="uploadedMedia.is_primary"></el-switch>
        </el-form-item>

        <el-form-item label="Display Label">
          <el-input
            type="text"
            v-model="uploadedMedia.display_label"
          ></el-input>
        </el-form-item>

        <el-form-item label="Description">
          <el-input type="text" v-model="uploadedMedia.description"></el-input>
        </el-form-item>

        <el-form-item label="Display Order">
          <el-input-number
            type="text"
            v-model="uploadedMedia.display_order"
            :min="1"
            :max="99"
          ></el-input-number>
        </el-form-item>
      </div>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button v-if="editItem" @click="$emit('submit', editItem.id)"
        >Delete</el-button
      >
      <el-button @click="$emit('close')">Cancel</el-button>
      <el-button
        type="primary"
        @click="addBack"
        :disabled="!uploadedMedia || uploadedMedia.id == 0"
        >Add</el-button
      >
    </span>
  </el-dialog>
</template>

<script lang='ts'>
import { ElUploadInternalFileDetail } from "element-ui/types/upload";
import Vue from "vue";

export interface MediaLiteItem {
  /** Format: int64 */
  id: number;
  url: string;
  thumbnail_url?: string | null;
  large_url?: string | null;
  display_label?: string | null;
  /** Format: int32 */
  display_order?: number;
  is_image: boolean;
  description?: string | null;
  is_primary: boolean;
}

export default Vue.extend({
  data() {
    return {
      file: null as ElUploadInternalFileDetail | null,
      form: {},
      uploadLoading: false,
      uploadedMedia: {
        id: 0,
        url: "",
        display_label: "",
        display_order: 0,
        is_image: false,
        description: "",
        is_primary: false,
      },
      accepted: [
        "image/png",
        "image/jpeg",
        "application/pdf",
        "text/csv",
        "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
        "application/json",
        "application/msword",
        "image/svg+xml",
        "application/vnd.ms-excel",
        "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
        "video/mp4",
      ],
    };
  },

  methods: {
    resetItem() {
      return {
        id: 0,
        url: "",
        display_label: "",
        display_order: 0,
        is_image: false,
        description: "",
        is_primary: false,
      };
    },

    addBack(): void {
      this.$emit("submit", this.uploadedMedia);
      this.uploadedMedia = this.resetItem();
      this.file = null;
      (this.$refs.upload as any).clearFiles();
    },

    handleRemove(): void {
      this.file = null;
      this.uploadedMedia = this.resetItem();
    },

    handleChange(file: ElUploadInternalFileDetail | null): void {
      this.file = file;
    },

    async submitUpload(): Promise<void> {
      if (this.file === null) return;
      this.uploadLoading = true;
      const fileData = new FormData();
      //   console.log(this.file)
      fileData.append("file", this.file.raw);
      try {
        const response = await this.$axios.post<MediaLiteItem[]>(
          this.mediaUrl,
          fileData,
          {
            headers: {
              Authorization: "Bearer " + this.token,
            },
          }
        );

        this.$emit("uploadedFile", response.data[0]);

        this.$message({
          duration: 4000,
          message: `File uploaded. Complete the form and submit.`,
          type: "success",
        });
      } catch (err: any) {
        this.$message({
          duration: 4000,
          message: err.message ?? `File Upload failed. Try again or try later.`,
          type: "error",
        });
      } finally {
        this.uploadLoading = false;
      }
    },
  },

  computed: {
    actualAccepted(): string[] {
      if (this.imageOnly) {
        if (this.allowVideo) return ["video/mp4"];
        else return ["image/png", "image/jpeg"];
      } else return this.accepted;
    },
  },

  props: {
    visible: {
      type: Boolean,
      required: true,
    },

    mediaUrl: {
      type: String,
      required: true,
    },

    token: {
      type: String,
      required: true,
    },

    imageOnly: {
      type: Boolean,
      default: false,
    },
    allowVideo: {
      type: Boolean,
      default: false,
    },
  },
});
</script>

<style lang='scss'>
</style>
