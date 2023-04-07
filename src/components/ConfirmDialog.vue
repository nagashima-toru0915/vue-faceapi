<template>
    <v-dialog v-model="dialog" persistent max-width="400px">
        <v-card>
        <v-card-title>
            <span class="headline">{{ title }}</span>
        </v-card-title>
        <v-card-text>
            {{ message }}
        </v-card-text>
        <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn variant="tonal"
                   color="gray darken-1" class="dialog-Button" v-on:keydown.enter="cancel">キャンセル</v-btn>
            <v-btn variant="tonal"
                   color="gray darken-1" class="dialog-Button" v-on:keydown.enter="confirm">{{buttonMessage}}</v-btn>
        </v-card-actions>
        </v-card>
    </v-dialog>
</template>
  
<script>
  export default {
    data() {
      return {
        dialog: false,
        resolve: null,
        reject: null,
        title: "",
        message: "",
        buttonMessage: "",
      }
    },
    methods: {
      open(title, message, buttonMessage) {
        this.dialog = true;
        this.title = title;
        this.message = message;
        this.buttonMessage = buttonMessage;
        return new Promise((resolve, reject) => {
            this.resolve = resolve;
            this.reject = reject;
        });
      },
      cancel() {
        this.dialog = false;
        this.resolve(false);
      },
      confirm() {
        this.dialog = false;
        this.resolve(true);
      }
    },
  }
</script>

<style scoped>
  .dialog-Button {
    font-weight: bold;
  }
</style>