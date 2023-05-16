<template>
  <o-page-container-forms>
    <v-form @submit.prevent="onSubmit">
      <v-col class="create-ticket text-left">
        <div class="text-h6 mb-2">Contact: {{ userSettingsStore?.userSettings?.name }}</div>
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
        <v-btn class="mt-9" color="primary" size="large" type="submit">Submit</v-btn>
      </v-col>
    </v-form>
  </o-page-container-forms>
</template>

<script setup>
import { OFileUpload, OPageContainerForms } from '@onify/helix-components';
import { useUserSettingsStore, useOnifyApi, useAlerts } from '@onify/helix-core';
import { reactive } from 'vue';

const userSettingsStore = useUserSettingsStore();
const onifyHttpRequest = useOnifyApi();
const alerts = useAlerts();

const CREATE_TICKET_WORKFLOW_URL = 'my/workflows/run/create-ticket?timeout=60';

const form = reactive({
  title: '',
  subject: '',
  attachments: [],
  contact: '',
});

async function onSubmit() {
  try {
    const fileUploadResults = await onifyHttpRequest.upload('/files/public/', form.attachments);
    form.attachments = fileUploadResults;
    const response = await onifyHttpRequest.update(CREATE_TICKET_WORKFLOW_URL, { json: form });

    if (response.status === 200) {
      // result.json() returns a promise, so we run an `await` to get the response body
      const createTicketResult = await response.json();

      alerts.addAlert({
        type: 'success',
        title: 'Ticket Submission Sucessful',
        body: `Ticket number: ${createTicketResult.output.ticketNumber}`,
      });
    }
  } catch (err) {
    alerts.addAlert({
      type: 'error',
      title: 'Ticket Submission Failed',
      body: 'Please try again',
      details: {
        title: err.name,
        description: err.stack,
        isCode: true,
      },
      actions: [
        {
          text: 'Show Details',
          willShowDetailsOnClick: true,
        },
      ],
    });
  }
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
