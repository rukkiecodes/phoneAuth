<template>
  <v-container>
    <v-card width="400" class="mx-auto mt-16">
      <v-card-text>
        <v-text-field v-if="!isTOPReady" v-model="phone" variant="underlined" label="phone"></v-text-field>
        <v-text-field v-if="isTOPReady" v-model="otp" variant="underlined" label="OTP"></v-text-field>
      </v-card-text>
      <v-card-actions>
        <v-btn v-if="!isTOPReady" block class="bg-indigo" @click="getOTP">Get OTP</v-btn>
        <v-btn v-if="isTOPReady" block class="bg-indigo" @click="signIn">Sign in</v-btn>
      </v-card-actions>
    </v-card>

    <div v-if="!isTOPReady" id="recaptcha-container"></div>
  </v-container>
</template>

<script>
import { auth } from "@/plugins/firebase";
import { PhoneAuthProvider, RecaptchaVerifier, signInWithCredential, signInWithPhoneNumber } from "firebase/auth";

export default {
  data: () => ({
    phone: '+2348071657443',
    otp: '',
    isTOPReady: false
  }),

  mounted() {
    window.recaptchaVerifier = new RecaptchaVerifier('recaptcha-container', {}, auth);
  },

  methods: {
    getOTP() {
      const appVerifier = window.recaptchaVerifier
      signInWithPhoneNumber(auth, this.phone, appVerifier)
        .then((confirmationResult) => {
          // SMS sent. Prompt user to type the code from the message, then sign the
          // user in with confirmationResult.confirm(code).
          window.confirmationResult = confirmationResult;
          console.log('confirmationResult: ', confirmationResult)
          this.isTOPReady = true
          // ...
        }).catch((error) => {
          console.log('error: ', error)
          this.isTOPReady = false
          // Error; SMS not sent
          // ...
        });
    },

    signIn() {
      let credential = PhoneAuthProvider.credential(window.confirmationResult.verificationId, this.otp)

      signInWithCredential(auth, credential)
    }
  }
}
</script>
