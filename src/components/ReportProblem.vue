<template>
  <q-dialog ref="dialog" @hide="onDialogHide">
    <q-card class="q-dialog-plugin" v-if="!submitted">
      <q-form
        name="report-problem"
        method="POST"
        data-netlify="true"
        data-netlify-honeypot="bot-field"
        enctype="multipart/form-data"
        @submit="reportProblem"
      >
        <div class="q-pa-md q-gutter-sm">
          <p class="text-h5">Uh oh, found a problem out there?</p>
          <p>We'll do our best to fix it.</p>

          <q-card-section>
            <input type="hidden" name="bot-field" />
            <q-input
              dense
              name="name"
              v-model="name"
              label="Your name"
              hint="This is optional."
            >
              <template v-slot:prepend>
                <q-icon name="person" />
              </template>
            </q-input>
            <q-input
              dense
              name="email"
              v-model="email"
              label="Your email"
              hint="This is optional."
              ><template v-slot:prepend>
                <q-icon name="mail" />
              </template>
            </q-input>
            <q-file
              dense
              name="picture"
              v-model="picture"
              accept=".jpg, image/*"
              capture="environment"
              :clearable="true"
              max-file-size="8388608"
              label="Add a picture"
              hint="This is optional."
            >
              <template v-slot:prepend>
                <q-icon name="add_a_photo" />
              </template>
            </q-file>
          </q-card-section>
          <q-input
            name="comments"
            outlined
            autogrow
            v-model="comments"
            placeholder="Tell us about your problem on the trails."
            :rules="[(val) => !!val || 'Please enter a report.']"
          ></q-input>
        </div>

        <q-card-actions align="right">
          <q-btn label="Cancel" @click="onCancelClick" />
          <q-btn
            color="primary"
            type="submit"
            :loading="submitting"
            label="Send Report"
          />
        </q-card-actions>
      </q-form>
    </q-card>
    <q-card v-if="submitted">
      <q-card-section>
        <h5 align="center">Thank you for your feedback!</h5>
        <p class="text-body1">
          We have received your report and will be investigating. If you
          provided contact information, we may be reaching out to you. Happy
          trails!
        </p>
      </q-card-section>
      <q-card-actions align="right">
        <q-btn color="primary" label="Close" @click="onOKClick" />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script>
import { ref } from "vue";

export default {
  data() {
    return {
      name: "",
      email: "",
      comments: "",
      picture: ref(null),
      submitted: false,
      submitting: false,
      sendError: null,
    };
  },
  props: {
    location: null,
  },
  emits: [
    // REQUIRED
    "ok",
    "hide",
  ],
  methods: {
    // following method is REQUIRED
    // (don't change its name --> "show")
    show() {
      this.$refs.dialog.show();
    },

    // following method is REQUIRED
    // (don't change its name --> "hide")
    hide() {
      this.$refs.dialog.hide();
    },

    onDialogHide() {
      // required to be emitted
      // when QDialog emits "hide" event
      this.$emit("hide");
    },

    async reportProblem() {
      let data = new FormData();
      data.append("name", this.name);
      data.append("email", this.email);
      data.append("comments", this.comments);
      data.append("picture", this.picture);

      if (this.location) {
        data.append(
          "location",
          `https://www.openstreetmap.org/?mlat=${this.location.lat}&mlon=${this.location.lng}#map=18/${this.location.lat}/${this.location.lng}&layers=Y`
        );
      }

      data.append("form-name", "report-problem");

      try {
        this.submitting = true;
        await fetch("/", {
          method: "POST",
          body: data,
        });
        this.submitting = false;
        this.submitted = true;
      } catch (e) {
        this.submitting = false;
        this.sendError = "Sorry, there was an error sending this in :(";
      }
    },

    onOKClick() {
      // on OK, it is REQUIRED to
      // emit "ok" event (with optional payload)
      // before hiding the QDialog
      this.$emit("ok");
      // or with payload: this.$emit('ok', { ... })

      // then hiding dialog
      this.hide();
    },

    onCancelClick() {
      // we just need to hide the dialog
      this.hide();
    },
  },
};
</script>
