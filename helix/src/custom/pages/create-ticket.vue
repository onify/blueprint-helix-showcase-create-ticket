<template>
  <h-form-container>
    <h-page-head title="Create Ticket"></h-page-head>
    <h-page-title>Create Ticket</h-page-title>
    <h-page-description>
      Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
    </h-page-description>
    <div style="width: 600px">
      <v-text-field
        v-model="onifyUser.name"
        class="mb-3"
        label="Contact"
        hint="Who is the contact person for this ticket?"
        append-icon="mdi:mdi-account"
        persistent-hint
        disabled
      />
      <v-form ref="vFormRef" validate-on="input" @submit.prevent="onSubmit">
        <v-text-field
          v-model="formData.title"
          class="mb-3"
          label="Title"
          append-icon="mdi:mdi-form-textbox"
          hint="Brief description of the incident"
          :rules="[helixValidate.required, helixValidate.min([5])]"
          persistent-hint
        />
        <v-textarea
          v-model="formData.description"
          class="mb-3"
          label="Description"
          append-icon="mdi:mdi-text"
          hint="Detailed explanation on the incident"
          :rules="[helixValidate.required, helixValidate.min([20])]"
          persistent-hint
        />
        <v-file-input
          v-model="files"
          label="Attachments"
          class="mb-3"
          prepend-icon=""
          append-icon="mdi:mdi-paperclip"
          hint="Upload attachments (max 3 files, max 10mb/file, max 30mb in total)"
          persistent-hint
          multiple
          accept="image/png, image/jpeg, .png, .jpg, .jpeg .doc, .docx, .xml,application/msword, application/vnd.openxmlformats-officedocument.wordprocessingml.document"
          show-size
          :rules="[helixValidate.maxFileSize(30_000, 10_000)]"
        />
        <v-btn class="mt-9" color="primary" type="submit" prepend-icon="mdi:mdi-plus" :loading="isSubmitting">Create ticket</v-btn>
      </v-form>
    </div>
  </h-form-container>
</template>

<script setup>
const { helixAlerts, helixHttpRequest } = useHelixCommon();
const { onifyUser } = useOnifyCommon();

/*
  Vue Template refs: https://vuejs.org/guide/essentials/template-refs.html
  We do this to access vForm's `validate()` function
*/
const vFormRef = ref();

// Form data to submit to the api
const formData = reactive({});

// Ref variable is used for the loading effect for the submit button
const isSubmitting = ref(false);

// Files array
const files = ref([]);

async function onSubmit() {
  // Make sure to validate the before prior to submission
  const { valid } = await vFormRef.value.validate();

  if (valid) {
    try {
      isSubmitting.value = true;

      // Upload files to Onify API
      const fileUploadResults = await helixHttpRequest.uploadFiles({
        type: 'public',
        files: files.value,
        //ttl: 3000, # Time to live before deleted
        tag: ['attachment', 'ticket'],
      });
      // Assign uploaded files to `formData.attachments`
      formData.attachments = fileUploadResults;

      // Submit the form. Sends a `POST` request to the API with the form data as payload
      const response = await helixHttpRequest({ url: 'my/workflows/run/create-ticket', method: 'post', payload: formData }).response();

      // Submission attempt completed. Removes the loading effect on the button
      isSubmitting.value = false;

      // Resets the form
      vFormRef.value.reset();

      // If `response.status` is 200(OK) submission was successful and we can present the ticket number to the user
      if (response.status === HTTPStatusCode.OK) {
        // result.json() returns a promise, so we run an `await` to get the response body
        const createTicketResult = await response.json();

        // Show "Submission Successful" alert
        helixAlerts.addAlert({
          type: 'success',
          title: 'TICKET CREATED',
          body: `Your ticket number is ${createTicketResult.output.ticketNumber}`,
        });
      }
    } catch (err) {
      /*  An error occurred somewhere during the form submission attempt.
          Remove the loading effect on the submit button to allow user to attempt to submit again */
      isSubmitting.value = false;
    }
  }
}
</script>
