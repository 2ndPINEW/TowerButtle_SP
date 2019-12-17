<template>
  <v-app>
    <v-app-bar app>
      <v-toolbar-title class="headline text-uppercase">
        <span></span>
        <span class="font-weight-light">電子部品タワーバトル3D</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <!--<v-btn
          text
          href="https://twitter.com/2ndpinew"
          target="_blank"
        ><span class="mr-2">Contact</span>
      </v-btn>-->
    </v-app-bar>

    <v-content>
      <router-view/>
    </v-content>
  </v-app>
</template>

<script>
import HelloWorld from './components/HelloWorld'

export default {
  name: 'App',
  components: {
    HelloWorld
  },
  data: () => ({
    //
  })
}
</script>
