<template>
  <o-page-container-forms>
    <v-form @submit.prevent="onSubmit">
      <v-col class="create-ticket text-left">
        <div class="text-h6 mb-2">
          Contact: {{ userSettingsStore?.userSettings?.name }}
        </div>
        <v-text-field
          v-model="form.title"
          label="Title"
          hint="Lorem ipsum dolor sit amet, consectetur adipiscing elit"
          persistent-hint
        ></v-text-field>
        <v-textarea
          v-model="form.subject"
          label="Subject"
          hint="Lorem ipsum dolor sit amet, consectetur adipiscing elit"
          persistent-hint
        ></v-textarea>
        <o-file-upload
          v-model="form.attachments"
          label="Attachments"
          hint="Lorem ipsum dolor sit amet, consectetur adipiscing elit"
          persistent-hint
        ></o-file-upload>
        <v-btn class="mt-9" color="primary" size="large" type="submit"
          >Submit</v-btn
        >
      </v-col>
    </v-form>
  </o-page-container-forms>
</template>

<script setup>
import { OFileUpload, OPageContainerForms } from "@onify/helix-components";
import { useUserSettingsStore, useOnifyApi } from "@onify/helix-core";
import { reactive } from "vue";

const userSettingsStore = useUserSettingsStore();

const onifyHttpRequest = useOnifyApi();

const form = reactive({
  title: "",
  subject: "",
  attachments: [],
  contact: "",
});

async function onSubmit() {
  const fileUploadResults = await onifyHttpRequest.upload(
    "/files/public/",
    form.attachments
  );
}
</script>

<style>
.create-ticket {
  max-width: 587px;
}

.create-ticket > * {
  margin-bottom: 12px;
}
</style>
