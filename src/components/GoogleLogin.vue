<template>
  <div>
    <div id="google-signin"></div>
    <button id="google-signout" v-on:click="signOut">Sign out</button>
  </div>
</template>

<script>
import { parseToken } from "../utils/jwt";

export default {
  name: "login",
  props: {
    onLogin: { type: Function, required: true },
    nonce: { type: String },
    forceSignin: { type: Boolean },
  },
  data() {
    return {
      error: null,
      tokenClient: null, // Token client for OAuth
    };
  },
  mounted() {
    this.loadGoogleIdentityServices();
  },
  computed: {
    computedNonce() {
      return this.nonce || this.generateNonce();
    }
  },
  methods: {
    // Generate a random nonce for the OAuth2 token request
    generateNonce() {
      const array = new Uint8Array(16);
      window.crypto.getRandomValues(array);
      return Array.from(array, byte => byte.toString(16).padStart(2, '0')).join('');
    },
    // Dynamically load GIS script
    loadGoogleIdentityServices() {
      const script = document.createElement("script");
      script.src = "https://accounts.google.com/gsi/client";
      script.async = true;
      script.defer = true;
      script.onload = this.initializeGoogleServices;
      document.head.appendChild(script);
    },
    // Initialize GIS library
    initializeGoogleServices() {
      if (!window.google) {
        this.error = "Google Identity Services not loaded.";
        return;
      }
      console.log("nonce initialize", this.computedNonce);
      // Initialize Token Client for OAuth2
      this.tokenClient = google.accounts.oauth2.initTokenClient({
        client_id: process.env.VUE_APP_GOOGLE_CLIENT_ID,
        nonce: this.computedNonce ? Buffer.from(this.computedNonce.slice(2), "hex").toString("base64").replace(/\+/g, "-").replace(/\//g, "_").replace(/=/g, "") : null,
        scope: "openid email",
        callback: this.handleLogin, 
      });

      // Render Google Sign-In button
      google.accounts.id.initialize({
        client_id: process.env.VUE_APP_GOOGLE_CLIENT_ID,
        nonce: this.computedNonce ? Buffer.from(this.computedNonce.slice(2), "hex").toString("base64").replace(/\+/g, "-").replace(/\//g, "_").replace(/=/g, "") : null,
        callback: this.handleLoginWithIdToken, // Handle ID token from sign-in
      });
      google.accounts.id.renderButton(
        document.getElementById("google-signin"),
        {
          theme: "outline",
          size: "large",
          text: "continue_with",
        }
      );

      // Optionally prompt the user if forceSignin is enabled
      if (this.forceSignin) {
        google.accounts.id.prompt();
      }
    },
    handleLoginWithIdToken(response) {
      // Handle ID token from Google Sign-In
      const idToken = response.credential;
      console.log("Signed in with ID token:", idToken, parseToken(idToken));
      console.log("nonce", this.computedNonce);
      this.onLogin(idToken);
      this.$notify({
        group: 'auth',
        type: 'success',
        title: 'Login Successful',
        text: 'You have successfully logged in.'
      });
    },
    handleLogin(tokenResponse) {
      // Handle OAuth2 access token response
      console.log("Access token response:", tokenResponse);
    },
    signOut() {
      // Google Identity Services does not provide a direct sign-out method
      google.accounts.id.disableAutoSelect();
      console.log("Signed out");
      this.$notify({
        group: 'auth',
        type: 'success',
        title: 'Sign Out Successful',
        text: 'You have successfully signed out.'
      });
    },
  },
};
</script>

<style>
#google-signin {
  width: 180px;
  margin: auto;
  margin-bottom: 20px;
}
</style>
