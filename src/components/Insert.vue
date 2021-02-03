<template lang="pug">
  v-container
    v-row(class="file-inputs")
      v-col(cols="12" md="6" class="file-ind")
        v-file-input(
          color="cyan darken-2"
          accept="image/jpeg, image/jpg"
          prepend-icon="mdi-camera"
          label="Selecciona una imagen médica"
          @change="newImage($event,1)"
        )
      v-col(cols="12" md="6" class="file-ind")
        v-file-input(
          color="cyan darken-2"
          accept="image/jpeg, image/jpg"
          prepend-icon="mdi-camera"
          label="Selecciona la imagen del reporte"
          @change="newImage($event,2)"
        )

    v-row
      v-col(cols="12" md="6")
        img(src="" ref="imgs1" class="img-base")
        canvas(id="imgsource1" ref="imgsource1" class="imgcanvas")

      v-col(cols="12" md="6")
        img(src="" ref="imgs2" class="img-base")
        canvas(id="imgsource2" ref="imgsource2" class="imgcanvas")

    v-row
      v-col(cols="12" md="12")
        v-btn(color="error" 
              :disabled="!isLoaded" 
              :dark="isLoaded" 
              @click="start()"
              large) Insert

        v-dialog(
          v-model="dialog1"
          max-width="340"
        )

          v-card
            v-card-title(class="headline") ERROR
            v-card-text Las imágenes deben ser del mismo tamaño
            v-card-actions
              v-spacer
                v-btn(
                color="green darken-1"
                text
                @click="dialog1 = false"
                ) Ok

    v-row
      v-col(cols="12" md="6")
        img(src="" ref="imgd1" class="img-base")
        canvas(id="imgdest1" ref="imgdest1" class="imgcanvas")

    v-row
      div(class="bottom-space")

</template>

<script>
  import * as cv from 'opencv.js'

  export default {
    name: 'ImageTest',

    data: () => ({
      iload1: false,
      iload2: false,
      dialog1: false,
    }),

    computed: {
      isLoaded: function() {
        return this.iload1 && this.iload2
      }
      
    },

    methods: {

      newImage(file, i) {
        let imgurl = URL.createObjectURL(file)
        let img

        if( i == 1 ) {
          img = this.$refs.imgs1
          this.iload1 = true
        }
        else {
          img = this.$refs.imgs2
          this.iload2 = true
        }

        img.src = imgurl

        this.loadImg()
      },

      loadImg() {
        let src, img, h, w

        if( this.iload1 ) {
          src = this.$refs.imgsource1
          img = this.$refs.imgs1          
        }
        if( this.iload2 ) {
          src = this.$refs.imgsource2
          img = this.$refs.imgs2          
        }

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
        let src1 = cv.imread(this.$refs.imgsource1)
        let src2 = cv.imread(this.$refs.imgsource2)
        let dst = new cv.Mat()
        let mask = new cv.Mat();
        let dtype = -1;

        if( src1.cols != src2.cols || src1.rows != src2.rows  ) {
            this.dialog1 = true
            return
        }

        cv.cvtColor(src1, src1, cv.COLOR_RGB2GRAY, 0);
        cv.cvtColor(src2, src2, cv.COLOR_RGB2GRAY, 0);

        cv.add(src1, src2, dst, mask, dtype);

        cv.imshow(this.$refs.imgdest1, dst)
        src1.delete() 
        src2.delete() 
        dst.delete()
        mask.delete()        
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