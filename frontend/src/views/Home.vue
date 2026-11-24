<script setup>
import { computed } from "vue";
import authServices from "../services/authServices.js";
import Utils from "../config/utils.js";

const firstName = computed(() => Utils.getStore("user")?.fName || "");

const signOut = () => authServices.logoutUser();
</script>

<template>
  <v-container class="fill-height">
    <v-row align="center" justify="center">
      <v-col cols="12" sm="8" md="6">
        <v-card elevation="2">
          <v-card-title class="text-h5">Welcome, {{ firstName }}</v-card-title>
          <v-card-text>Your account is signed in.</v-card-text>
          <v-card-actions>
            <v-btn color="primary" variant="elevated" class="oc-cta" @click="signOut">
              Sign out
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>
