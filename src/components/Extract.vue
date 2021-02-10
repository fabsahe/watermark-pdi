<template lang="pug">
  v-container
    v-row(class="file-inputs")
      v-col(cols="12" md="6" class="file-ind")
        v-file-input(
          color="cyan darken-2"
          accept="image/jpeg, image/jpg"
          prepend-icon="mdi-camera"
          label="Selecciona una imagen marcada"
          @change="newImage($event)"
        )


    v-row
      v-col(cols="12" md="6")
        img(src="" ref="imgs3" class="img-base")
        canvas(id="imgsource3" ref="imgsource3" class="imgcanvas")

    v-row
      v-col(cols="12" md="4")
        v-text-field(
            v-model="password2"
            :append-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'"
            :rules="[rules.required, rules.min]"
            :type="show1 ? 'text' : 'password'"
            name="input-10-1"
            label="Contraseña"
            hint="6 caracteres minimo"
            counter
            @click:append="show2 = !show2"
        )

    v-row
      v-col(cols="12" md="12")
        v-btn(color="error" 
              :disabled="imgLoadedE==0" 
              :dark="imgLoadedE==1" 
              @click="start()"
              large) Extraer

    v-row
      div(class="bottom-space")
      v-divider

    v-row(style="margin-top: -3rem")
      v-col(cols="12" md="6")
        v-chip(
          class="ma-2"
          color="green"
          text-color="white"
          align="left"
        ) Resultados

    v-row
      v-col(cols="12" md="6")
        img(src="" ref="imgd2" class="img-base")
        canvas(id="imgdest2" ref="imgdest2" class="imgcanvas")

      v-col(cols="12" md="6")
        img(src="" ref="imgd3" class="img-base")
        canvas(id="imgdest3" ref="imgdest3" class="imgcanvas")

    v-row
      div(class="bottom-space")

</template>

<script>
  import * as cv from 'opencv.js'

  export default {
    name: 'ImageTest',

    data: () => ({
      imgLoadedE: 0,

      show2: false,
      password2: '',
      rules: {
        required: value => !!value || 'Campo obligatorio',
        min: v => v.length >= 6 || '6 caracteres min',
      },
    }),

    methods: {

      newImage(file) {
        let imgurl = URL.createObjectURL(file)
        let img = this.$refs.imgs3

        img.src = imgurl
        this.imgLoadedE = 1
        this.loadImg()
      },

      loadImg() {
        let src, img, h, w

        src = this.$refs.imgsource3
        img = this.$refs.imgs3

        let ctx = src.getContext('2d')

        img.onload = function() {
          if( img.width>550 ) {
            h = img.height*(550/img.width)
            w = 550
            src.height = h
            src.width = w          
          }
          else {
            h = img.height
            w = img.width
            src.height = h
            src.width = w    
          }
          ctx.drawImage(img, 0, 0, w, h)
        }

      },

      start() {
        let src = cv.imread(this.$refs.imgsource3)
        let dst2 = new cv.Mat()
        
        let dst3 = new cv.Mat();
        let ksize = new cv.Size(5, 5);


        cv.cvtColor(src, src, cv.COLOR_RGB2GRAY, 0);
        // You can try more different parameters
        cv.Laplacian(src, dst2, cv.CV_8U, 1, 1, 0, cv.BORDER_DEFAULT);

        // You can try more different parameters
        cv.GaussianBlur(src, dst3, ksize, 0, 0, cv.BORDER_DEFAULT);

        cv.imshow(this.$refs.imgdest2, dst2)
        cv.imshow(this.$refs.imgdest3, dst3)
        src.delete();        
        dst2.delete()  
        dst3.delete()    

      }
      // fin de los metodos      
    }
  }
</script>

<style scoped>
.file-inputs {
  margin-top: 0;
}
.file-ind {
  padding-right: 2em;
}
.img-base {
  display: none;
}
.bottom-space {
  height: 5rem;
}
</style>