<template lang="pug">
  v-app
    v-toolbar(
      color="grey darken-4"
      dark
      flat
      extension-height="40"
    )
      v-btn(icon dark)
        v-icon mdi-image-plus

      v-toolbar-title WATERMARKING

      v-spacer

      v-tooltip(bottom)
        template(v-slot:activator="{ on, attrs }")
          v-btn(
            icon
            dark
            v-bind="attrs"
            v-on="on"
            @click="forceRerender()"
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
          v-tab(:href="`#home`") Inicio
          v-tab(:href="`#insert`") Inserción
          v-tab(:href="`#extract`") Extracción

    v-main

      v-tabs-items(v-model="model")
        v-tab-item(:value="`home`")
          v-container
            v-row
              v-col(cols="12" md="12")
                h2 Sistema de marcas de agua en imágenes para ocultamiento de reportes
                br
            v-row
              v-col(cols="12" md="12" align="center")
                v-card(class="mx-auto" max-width="500")

                  v-img(
                      alt="Insert"
                      class="img-right"
                      src="./assets/insert-file.png"
                      transition="scale-transition"
                      width="50")

                  v-list-item(three-line)
                    v-list-item-content
                      div(class="overline mb-4")
                        span 
                      v-list-item-title(class="headline mb-1")
                        span MÓDULO DE INSERCIÓN
                      v-list-item-subtitle(class="marb") Esta función le permitirá obtener una imagen cifrada con marca de agua a partir de una imagen médica y de un reporte  

                  v-card-actions
                    v-btn(
                      color="light-green darken-3"
                      rounded
                      text
                      @click="changeTab('insert')"
                    ) IR A INSERCIÓN
                br

            v-row
              v-col(cols="12" md="12" align="center")
                v-card(class="mx-auto" max-width="500")

                  v-img(
                      alt="Insert"
                      class="img-right"
                      src="./assets/extract-file.png"
                      transition="scale-transition"
                      width="50")

                  v-list-item(three-line)
                    v-list-item-content
                      div(class="overline mb-4")
                        span 
                      v-list-item-title(class="headline mb-1")
                        span MÓDULO DE EXTRACCIÓN

                      v-list-item-subtitle(class="marb") Esta función le permitirá obtener el reporte médico de una imagen cifrada con marca de agua 

                  v-card-actions
                    v-btn(
                      color="light-green darken-3"
                      rounded
                      text
                      @click="changeTab('extract')"
                    ) IR A EXTRACCIÓN
                br
            v-row
              br
              div(class="bottom-space")      

        v-tab-item(:value="`insert`")
          Insert(:key="componentKey")
               
        v-tab-item(:value="`extract`")
          Extract(:key="componentKey")

       
    //- footer 
    v-footer(
        dark
        padless
      )
        v-card(
          flat
          tile
          width="100%"
          class="blue-grey darken-4 white--text text-center"
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
            p Integrantes:
            div(class="inteli")
              p Aguilar Reyes Jose Luis
              p García Mercado Andres
              p Guzmán Torres Jesica Janet
              p Herrera Reyes David
              p Salinas Hernández Fabián


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
    model: 'home',
    componentKey: 0,
    icons: [
        'mdi-lightbulb'
      ],
    items: [
        { title: 'Ayuda' },
      ],
  }),

  methods: {
    forceRerender() {
      this.componentKey += 1
      this.model = 'home'
    },

    changeTab(tab) {
      this.model = tab
    }
  }
};
</script>

<style scoped>
.left-marg {
  margin-left: 0.6rem;
}
.img-right {
  float: right;
  margin: 10px 10px 0 0;
}
.marb {
  margin-top: 10px;
}
.inteli {
  line-height: 5px;
}
</style>