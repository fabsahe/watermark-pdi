<template lang="pug">
  v-app
    v-toolbar(
      color="grey darken-4"
      dark
      flat
      extension-height="40"
    )
      v-app-bar-nav-icon

      v-toolbar-title WATERMARKING

      v-spacer

      v-tooltip(bottom)
        template(v-slot:activator="{ on, attrs }")
          v-btn(
            icon
            dark
            v-bind="attrs"
            v-on="on"
          )
            v-icon mdi-restart
        span Reset

      v-menu(bottom left :offset-y="true")
        template(v-slot:activator="{ on, attrs }")

          v-btn(
            dark
            icon
            v-bind="attrs"
            v-on="on"
          )
            v-icon mdi-dots-vertical



        v-list
          v-list-item(
            v-for="(item, i) in items"
            :key="i"
          )
            v-list-item-title {{ item.title }}


      template(v-slot:extension)
        v-tabs(
          v-model="model"
          centered
          slider-color="yellow"
        )
          v-tab(:href="`#insert`") Insertar
          v-tab(:href="`#extract`") Extraer

    v-main

      v-tabs-items(v-model="model")
        v-tab-item(:value="`insert`")
          Insert
               
        v-tab-item(:value="`extract`")
          Extract

       
    //- footer 
    v-footer(
        dark
        padless
      )
        v-card(
          flat
          tile
          width="100%"
          class="indigo lighten-1 white--text text-center"
        )
          v-card-text
            v-btn(
              v-for="icon in icons"
              :key="icon"
              class="mx-4 white--text"
              icon
            )
              v-icon(size="24px") {{ icon }}
              

          v-card-text(class="white--text pt-0")
            | Integrantes:

          v-divider

          v-card-text(class="white--text") {{ new Date().getFullYear() }} — 
            strong Equipo 3

      


</template>

<script>
import Insert from './components/Insert';
import Extract from './components/Extract';

export default {
  name: 'App',

  components: {
    Insert,
    Extract,
  },

  data: () => ({
    model: 'tab-2',
    icons: [
        'mdi-heart'
      ],
    items: [
        { title: 'Ayuda' },
      ],
  }),
};
</script>

<style scoped>
.left-marg {
  margin-left: 0.6rem;
}

</style>