<template>
  <div>
      <div class="elevation-10 pa-10">
        <h1 class="text-center">LOGIN</h1>

        <v-text-field label="Email"></v-text-field>

        <v-text-field label="Password" type="password" ></v-text-field>
        <p align="right" class="mb-0 pb-4 subtext">Forgot Password? <a>Get Help</a></p>
        <v-btn width="100%" :tile="tile" class="mb-4 mt-2" color="primary">LOGIN</v-btn>
        <v-divider class="my-5"></v-divider>
        <v-row class="mx-0">
            <v-col class="py-0 px-0 mb-2" cols="12">
                <p class="my-0 subtext text-center">Or use an alternative provider:</p>
            </v-col>
            <v-col cols="6" class="px-0 pr-2">
                <v-btn width="100%" :tile="tile" color="secondary"><v-icon left>mdi-google</v-icon>Google</v-btn>
            </v-col>
            <v-col cols="6" class="px-0 pl-2">
                <v-btn width="100%" :tile="tile" color="secondary"><v-icon left>mdi-facebook</v-icon>Facebook</v-btn>
            </v-col>
        </v-row>
      </div>
      <p class="mt-10 subtext text-center">Don't have an account already? <a href="/register">Sign Up</a></p>
  </div>
</template>

<script>

export default {
    props: {
        tile: {
            type: Boolean,
            default: false
        }
    },
    data() {
        return {

        }
    }
}


</script>

<style scoped>
.subtext {
    font-size: 14px;
    color: #555555;
}
</style>