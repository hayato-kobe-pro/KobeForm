<template>
  <v-row justify="center">
    <v-dialog v-model="dialogDetail" persistent max-width="600">
      <v-card>
        <v-card-title>
          <span class="headline">User Profile</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col md="4">
                <h3>{{ dialogInfo.arg.name }}</h3>
              </v-col>
              <v-col md="4">
                <h3>{{ dialogInfo.arg.age }}歳</h3>
              </v-col>
              <v-col md="4">
                <h3>{{ dialogInfo.arg.experience }}</h3>
              </v-col>
              <v-col cols="12">
                <p>＜志望動機＞</p>
                <v-text>{{ dialogInfo.arg.passion }}</v-text>
              </v-col>
              <v-container />
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn color="blue darken-1" text @click.stop="closeProfile()">
            Close
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>
<script>
export default {
  props: {
    dialogDetail: { type: Boolean, default: false },
    dialogInfo: {
      type: Object,
      default () {
        return {}
      }
    }
  },
  data () {
    return {

    }
  },

  methods: {
    closeProfile () {
      this.$emit('closeProfile')
    }
  }
}
</script>
<style>
h3 {
  border-bottom: thin solid rgba(255,255,255,0.5);
}
p {
  color: azure;
}

</style>
