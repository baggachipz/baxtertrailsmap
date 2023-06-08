<template>
  <q-dialog ref="dialog" @hide="onDialogHide">
    <q-card class="q-dialog-plugin">
      <q-form
        name="report-problem"
        method="POST"
        data-netlify="true"
        netlify-honeypot="bot-field"
        data-netlify-recaptcha="true"
        @submit="reportProblem"
      >
        <div class="q-pa-md q-gutter-sm">
          <p class="text-h5">Uh oh, found a problem out there?</p>
          <p>We'll do our best to fix it.</p>

          <q-card-section>
            <input type="hidden" name="bot-field" />
            <q-input
              name="name"
              v-model="name"
              label="Your name"
              hint="This is optional."
            ></q-input>
            <q-input
              name="email"
              v-model="email"
              label="Your email"
              hint="This is optional."
            ></q-input>
          </q-card-section>
          <q-input
            name="comments"
            outlined
            autogrow
            v-model="comments"
            placeholder="Tell us about your problem on the trails."
          ></q-input>

          <div data-netlify-recaptcha="true"></div>
        </div>
        <q-card-actions align="right">
          <q-btn color="primary" type="submit" label="Send Report" />
          <q-btn color="primary" label="Cancel" @click="onCancelClick" />
        </q-card-actions>
      </q-form>
    </q-card>
  </q-dialog>
</template>

<script>
export default {
  data() {
    return {
      name: "",
      email: "",
      comments: "",
      submitted: false,
      sendError: null,
    };
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
      data.append("form-name", "report-problem");

      try {
        await fetch("/", {
          method: "POST",
          headers: { "Content-Type": "application/x-www-form-urlencoded" },
          body: data,
        });
        this.submitted = true;
        setTimeout(this.onOKClick, 5000);
      } catch (e) {
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
