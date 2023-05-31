<template>
  <o-page-container type="form">
    <v-text-field
      v-model="user.name"
      class="mb-3"
      label="Contact"
      hint="Lorem ipsum dolor sit amet, consectetur adipiscing elit"
      prepend-icon="mdi:mdi-account"
      persistent-hint
      disabled
    />
    <v-form ref="vFormRef" validate-on="input" @submit.prevent="onSubmit">
      <v-text-field
        v-model="formData.title"
        class="mb-3"
        label="Title"
        prepend-icon="mdi:mdi-form-textbox"
        hint="Lorem ipsum dolor sit amet, consectetur adipiscing elit"
        :rules="[validate.required, validate.minLength(5)]"
        persistent-hint
      />
      <v-textarea
        v-model="formData.subject"
        class="mb-3"
        label="Subject"
        prepend-icon="mdi:mdi-text"
        hint="Lorem ipsum dolor sit amet, consectetur adipiscing elit"
        :rules="[validate.required, validate.minLength(20)]"
        persistent-hint
      />
      <v-file-input
        v-model="files"
        label="Attachments"
        class="mb-3"
        hint="Lorem ipsum dolor sit amet, consectetur adipiscing elit"
        persistent-hint
        multiple
        accept="image/png, image/jpeg, .png, .jpg, .jpeg .doc, .docx, .xml,application/msword, application/vnd.openxmlformats-officedocument.wordprocessingml.document"
        show-size
        :rules="[validate.maxLength(3), validate.maxFileSize(30_000, 10_000)]"
      />
      <v-btn class="mt-9" color="primary" type="submit" :loading="isSubmitting">Submit</v-btn>
    </v-form>
  </o-page-container>
</template>

<script setup>
const { alerts, user, onifyApiRequest } = usePageCommon();

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
      const fileUploadResults = await onifyApiRequest.uploadFiles({
        type: 'public',
        files: files.value,
        ttl: 3000,
        role: ['admin', 'manager'],
        refresh: false,
        tag: ['image', 'jpg'],
      });
      // Assign uploaded files to `formData.attachments`
      formData.attachments = fileUploadResults;

      // Submit the form. Sends a `POST` request to the API with the form data as payload
      const response = await onifyApiRequest('my/workflows/run/create-ticket?timeout=60', { method: 'post', json: formData }).response();

      // Submission attempt completed. Removes the loading effect on the button
      isSubmitting.value = false;

      // Resets the form
      vFormRef.value.reset();

      // If `response.status` is 200(OK) submission was successful and we can present the ticket number to the user
      if (response.status === HTTPStatusCode.OK) {
        // result.json() returns a promise, so we run an `await` to get the response body
        const createTicketResult = await response.json();

        // Show "Submission Successful" alert
        alerts.addAlert({
          type: 'success',
          title: 'Ticket Submission Successful',
          body: `Ticket number: ${createTicketResult.output.ticketNumber}`,
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
