<template>
    <v-dialog v-model="dialog" persistent max-width="400px">
        <v-card>
          <v-card-title>
            <span class="headline">{{ message }}</span>
          </v-card-title>
          <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn variant="tonal"
                    color="gray darken-1" class="dialog-Button" v-on:keydown.enter="ok">OK</v-btn>
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
        message: "",
      }
    },
    methods: {
      open(message) {
        this.dialog = true;
        this.message = message;

        return new Promise((resolve, reject) => {
            this.resolve = resolve;
            this.reject = reject;
        });
      },
      ok() {
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