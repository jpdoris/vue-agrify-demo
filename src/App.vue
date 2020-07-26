<template>
  <v-app>
    <v-app-bar app color="#6b745e" dark min-height="80" class="py-2 navbar">
      <div class="d-flex align-center">
        <v-img
          alt="Agrify Logo"
          class="shrink mr-2"
          contain
          src="@/assets/agrify-logo-horizontal-white-july.webp"
          transition="scale-transition"
          width="250"
        />
      </div>

      <v-spacer></v-spacer>

      <v-btn text>
        <span class="mr-2">Code Challenge</span>
      </v-btn>
    </v-app-bar>

    <v-main>
      <InteractiveForm />
    </v-main>
  </v-app>
</template>

<script>
import InteractiveForm from '@/components/InteractiveForm';

export default {
  name: "App",
  components: {
    InteractiveForm
  }
};
</script>
