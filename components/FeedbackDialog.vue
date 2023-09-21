<template>
  <div>
    <!-- Feedback Types -->
    <el-dialog
      :visible.sync="dialogVisible"
      width="40%"
      style="z-index: 9999"
      class="feedback-dialog"
    >
      <template v-slot:title>
        <div class="dialog-title">
          <span
            ><b
              >Hi there!<br />
              We can’t wait to get your thoughts on our application!</b
            ></span
          >
          <br />
          <p class="sub-heading">What would you like to do?</p>
        </div>
      </template>
      <div>
        <div
          @click="selectFeedbackType(item)"
          v-for="item in feedbackTypes"
          :key="item.id"
          class="option-list"
          :class="{ 'border-active': feedbackType === item.name }"
        >
          <el-radio
            class="feedback-option"
            v-model="feedbackType"
            :label="item.name"
            ref="feedback"
          ></el-radio>
          <br />
          <span class="sub-title" style="margin-left: 24px">{{
            item.description
          }}</span>
        </div>
        <br />
        <el-button
          :disabled="feedbackType == null"
          @click="openForm()"
          type="primary"
          style="width: 100%"
          >Continue</el-button
        >
      </div>
    </el-dialog>

    <!-- Feedback Form -->
    <el-dialog
      :visible.sync="formVisible"
      width="40%"
      style="z-index: 9999"
      class="feedback-dialog"
      :show-close="false"
    >
      <template v-slot:title>
        <div class="dialog-title">
          <span class="title-container">
            <el-button
              @click="goBack"
              icon="el-icon-arrow-left"
              class="dialog-back-button"
            ></el-button>
            <span class="selected-option dialog-title">{{ feedbackType }}</span>
          </span>

          <div class="form-divider"></div>
          <p v-if="shouldShowAttachments" class="sub-heading">
            We are listening! Please provide as much information as possible so
            that we can help you.
          </p>
          <p v-if="!shouldShowAttachments" class="sub-heading">
            Shout-out your favorite feature! We'd love to hear what you like!
          </p>
        </div>
      </template>
      <template>
        <el-form>
          <el-form-item label="Feedback" required :error="messageError">
            <el-input
              v-model="feedbackItem.message"
              type="textarea"
              placeholder="Enter your feedback."
              :autosize="{ minRows: 4, maxRows: 4 }"
              class="feedback-textarea"
            ></el-input>
          </el-form-item>
          <el-form-item label="E-mail ID" required :error="emailError">
            <el-input
              disabled
              v-model="feedbackItem.email"
              placeholder="Enter your E-mail ID."
              style="width: 100%; font-family: Roboto"
            ></el-input>
          </el-form-item>

          <el-form-item v-if="shouldShowAttachments" label="Attachments"
            ><br />
            <el-upload
              ref="uploadRef"
              class="upload-demo"
              :action="feedbackItem.media_url"
              :on-remove="handleRemove"
              :on-success="onUploadSuccess"
              list-type="picture"
              :headers="{
                Authorization: 'Bearer ' + feedbackItem.auth_token,
              }"
            >
              <el-button size="small" type="primary">Click to upload</el-button>
            </el-upload>
          </el-form-item>
          <span style="display: flex">
            <el-button @click="goBack()">Cancel</el-button>
            <el-button @click="submit" type="primary" style="width: 100%"
              >Submit</el-button
            >
          </span>
        </el-form>
      </template>
    </el-dialog>
  </div>
</template>

<script lang="ts">
import { ElUploadInternalFileDetail } from "element-ui/types/upload";
import Vue from "vue";
import axios from "axios";

export interface FeedbackItem {
  email?: string | null;
  message?: string;
  type: number;
  sdk_version: string;
  app_version: string;
  auth_token: string;
  user_agent?: string | null;
  device_id?: string | null;
  attachments: MediaLiteItem[];
  feedback_url: string;
  media_url: string;
}

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
  name: "FeedbackDialog",
  data() {
    return {
      file: null as ElUploadInternalFileDetail[] | null,
      feedbackItem: {
        email: null as string | null,
        message: null as string | null,
        auth_token: null as string | null,
        sdk_version: null as string | null,
        app_version: null as string | null,
        user_agent: null as string | null,
        device_id: null as string | null,
        feedback_url: null as string | null,
        media_url: null as string | null,
        attachments: [] as MediaLiteItem[],
        type: null as number | null,
      } as FeedbackItem,
      emailError: null as string | null,
      messageError: null as string | null,
      loading: false,
      dialogVisible: false,
      formVisible: false,
      feedbackType: null,
      canShowAttachment: false,
      feedbackTypes: [
        {
          id: 1,
          name: "Report a Bug",
          description: "Let us know so we can forward this to our bug control.",
        },
        {
          id: 2,
          name: "Request a Feature",
          description:
            "Do you have an idea that could make our app better? We would love to know!",
        },
        {
          id: 3,
          name: "Send Applause",
          description: "Let us know what you really like about our app!",
        },
      ],
    };
  },

  computed: {
    shouldShowAttachments(): boolean {
      return this.feedbackType != "Send Applause";
    },
  },

  methods: {
    selectFeedbackType(feedbackType: any) {
      this.feedbackType = feedbackType.name;
    },

    handleRemove(file: any) {
      let attachment = file.response[0];

      let index = this.feedbackItem.attachments.findIndex(
        (x) => (x.id = attachment.id)
      );

      if (index >= 0) this.feedbackItem.attachments.splice(index, 1);
    },

    onUploadSuccess(file: any) {
      let attachment = file[0];

      this.feedbackItem.attachments.push({
        id: attachment.id,
        display_label: attachment.display_label,
        description: attachment.description,
        display_order: attachment.display_order,
        is_primary: attachment.is_primary,
      } as MediaLiteItem);
    },

    openDialog(feedback: FeedbackItem) {
      this.dialogVisible = true;
      this.feedbackType = null;

      this.feedbackItem = {
        type: 0,
        message: "",
        email: feedback.email,
        sdk_version: feedback.sdk_version,
        auth_token: feedback.auth_token,
        feedback_url: feedback.feedback_url,
        app_version: feedback.app_version,
        user_agent: feedback.user_agent,
        device_id: feedback.device_id,
        media_url: feedback.media_url,
        attachments: [] as MediaLiteItem[],
      } as FeedbackItem;
    },

    uploadedFile(media: any) {
      console.log(media);
      this.feedbackItem.attachments.push({ id: media.id } as MediaLiteItem);
    },

    resetForm() {
      this.emailError = null;
      this.messageError = null;
      this.feedbackItem.message = "";
      try {
        (this.$refs.uploadRef as any).clearFiles();
      } catch (e) {}
    },

    openForm() {
      this.resetForm();
      this.dialogVisible = false;
      this.formVisible = true;
    },

    goBack() {
      this.$confirm(
        "This will clear all the filled data. Continue?",
        "Warning",
        {
          confirmButtonText: "OK",
          cancelButtonText: "Cancel",
          type: "warning",
        }
      )
        .then(() => {
          this.dialogVisible = true;
          this.formVisible = false;
          this.feedbackItem.message = "";
        })
        .catch(() => {
          return;
        });
    },

    async submit(): Promise<void> {
      this.emailError = "";
      this.messageError = "";
      if (
        !(this.feedbackItem?.message!.length > 0) &&
        !(this.feedbackItem?.email!.length > 0)
      ) {
        this.messageError = "Please enter your feedback.";
        this.emailError = "Please enter your email.";
        return;
      }

      if (!(this.feedbackItem?.message!.length > 0)) {
        this.messageError = "Please enter your feedback.";
        return;
      }

      if (this.feedbackItem?.email == null) {
        this.emailError = "Please enter your email.";
        return;
      }

      try {
        this.feedbackItem.type = this.feedbackTypes.find(
          (x) => x.name == this.feedbackType
        )!.id;
        this.loading = true;
        let authToken: string = this.feedbackItem.auth_token;

        const response = await axios.post(
          this.feedbackItem.feedback_url,
          this.feedbackItem,
          {
            headers: { Authorization: "Bearer " + authToken },
          }
        );

        this.$message({
          message: "Feedback Submitted.",
          type: "success",
        });

        this.dialogVisible = false;
        this.formVisible = false;
        this.feedbackItem = {} as FeedbackItem;
        this.resetForm();
      } catch (error: any) {
        this.$message({
          message: "Error in creating feedback.",
          type: "error",
        });
      } finally {
        this.loading = false;
      }
    },
  },
});
</script>

<style lang="scss">
.dialog-title {
  font-weight: 700;
  font-size: 16px;
  color: #171724;
  line-height: 22px;
  letter-spacing: -0.01px;
}

.sub-heading {
  color: #9d9da6;
  line-height: 24px;
  font-weight: 400;
  font-size: 14px;
  letter-spacing: -0.02px;
  margin-bottom: 10px;
  margin-top: 6px;
  .feedback-option {
    color: #171724;
    font-weight: 500;
  }
}

.sub-title {
  color: #9d9da6;
  line-height: 22px;
  font-weight: 500;
  font-size: 12px;
  letter-spacing: -0.02px;
  margin-bottom: 10px;
  margin-top: 6px;
  display: flex;
  flex-wrap: wrap;
  word-break: break-word;
}

.option-list {
  padding-left: 16px;
  padding-right: 16px;
  padding-top: 9px;
  border-radius: 4px;
  letter-spacing: -0.01px;
  cursor: pointer;
  border: 1px solid var(--border-color, #dcdfe6);
  background: var(--fill-color-blank, #fff);
  &:not(:first-child) {
    margin-top: 16px;
  }
}

.border-active {
  border: 1px solid #409eff;
  color: #409eff;
}

.feedback-dialog {
  min-width: 550px;
  .el-dialog__body {
    border-top: 1px solid #dcdfe6;
    padding: 20px 20px;
  }
  .el-radio__label {
    color: #171724;
  }
}

.form-divider {
  border-top: 1px solid #dcdfe6;
  margin-bottom: 12px;
  margin-top: 12px;
}

.title-container {
  display: flex;
  align-items: center;
}

.title-container .selected-option {
  margin-left: 160px;
  flex: 1;
}

.feedback-textarea {
  width: 100%;
  textarea {
    font-family: "Roboto", sans-serif;
  }
}

.el-form-item__label {
  color: #171724;
}

.icon {
  align-items: center;
}

.dialog-back-button {
  display: flex;
  width: 32px;
  height: 32px;
  justify-content: center;
  align-items: center;
  gap: 4px;
  border-radius: 4px;
  border: 1px solid rgb(152, 152, 152);
}

.el-button--primary.is-disabled {
  background-color: #d0d0d0;
  border-color: #d0d0d0;
}
</style>
