<template>
  <v-app>
    <v-card>
      <v-app-bar app v-if="showMenu()">
          
        <v-toolbar-title class="headline text-uppercase">
          <span>Tasking</span>
        </v-toolbar-title>
        <v-spacer></v-spacer>
        <v-switch v-model="$vuetify.theme.dark" inset
            label="Vuetify Theme Dark" persistent-hint></v-switch>
        <template v-slot:extension>
          <div class="d-flex">
            <span class="mr-3">
              <p class="font-weight-bold">Nombre del Proyecto</p>
            </span>
            <span class="mr-3">
              <v-avatar size="30">
                <img src="https://t4.ftcdn.net/jpg/01/16/06/45/240_F_116064582_KlXENacGmdt4xl8H6fQRYfSZLntLNKSX.jpg"
                  alt="John">
              </v-avatar>

            </span>
          </div>

        </template>
      </v-app-bar>

      <v-main>
        <router-view></router-view>
      </v-main>
    </v-card>
  </v-app>
</template>

<script>

export default {
  name: 'App',
  data: () => ({
    hiddenInRoutes: ['login']
  }),
  methods: {
    showMenu() {
      if (this.hiddenInRoutes.includes(this.$route.name)) return false
      return true
    }
  }
};
</script>
