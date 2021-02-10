<template lang="pug">
  v-container
    v-row(class="file-inputs")
      v-col(cols="12" md="6" class="file-ind")
        v-file-input(
          color="cyan darken-2"
          accept="image/jpeg, image/jpg, .dcm"
          prepend-icon="mdi-camera"
          label="Selecciona una imagen médica"
          @change="newImageMed($event)"
        )
      v-col(cols="12" md="6" class="file-ind")
        v-file-input(
          color="cyan darken-2"
          accept="image/jpeg, image/jpg"
          prepend-icon="mdi-camera"
          label="Selecciona la imagen del reporte"
          @change="newImageRep($event)"
        )

    v-row
      v-col(cols="12" md="6" align="center")
        img(src="" ref="imgs1" class="img-base")
        canvas(id="imgsource1" ref="imgsource1" class="imgcanvas" v-show="isjpg")


        div(id="dcmsource1" ref="dcmsource1" class="dcmc")
          canvas(class="cornerstone-canvas" ref="canvas1" id="canvas1")

      v-col(cols="12" md="6" align="center")
        img(src="" ref="imgs2" class="img-base")
        canvas(id="imgsource2" ref="imgsource2" class="imgcanvas")

    v-row
      v-col(cols="12" md="4")
        v-text-field(
            v-model="password"
            :append-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'"
            :rules="[rules.required, rules.min]"
            :type="show1 ? 'text' : 'password'"
            name="input-10-1"
            label="Contraseña"
            hint="6 caracteres minimo"
            counter
            @click:append="show1 = !show1"
        )

    v-row
      v-col(cols="12" md="12")
        v-btn(color="error" 
              :disabled="!isLoaded" 
              :dark="isLoaded" 
              @click="start()"
              large) Insertar

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
      div(class="bottom-space")
      v-divider

    v-row
      v-col(cols="12" md="6")
        v-chip(
          class="ma-2"
          color="green"
          text-color="white"
          align="left"
        ) Resultados

    v-row
      v-col(cols="12" md="6" align="center")
        img(src="" ref="imgd1" class="img-base")
        canvas(id="imgdest1" ref="imgdest1" class="imgcanvas")

    v-row
      v-col(cols="12" md="12")
        v-btn(color="success" 
              :disabled="!finished" 
              :dark="finished" 
              @click="downloadImg()"
              large) 
          v-icon mdi-download
          span Descargar

    v-row
      div(class="bottom-space")


</template>

<script>
  import dicomParser from "dicom-parser"
  import * as cornerstone from "cornerstone-core"
  import * as cornerstoneWADOImageLoader from "cornerstone-wado-image-loader"
  import * as cv from 'opencv.js'

  // Externals
  cornerstoneWADOImageLoader.external.cornerstone = cornerstone;
  cornerstoneWADOImageLoader.external.dicomParser = dicomParser;

  // CodeSandbox live updates components in an odd way.
  // We do this to protect ourselves from duplicate initializations
  // if (!cornerstoneWADOImageLoader.initialized) {
  //   // WadoImageLoader Registration/Config
  //   const config = {
  //     webWorkerPath: "/codecs/cornerstoneWADOImageLoaderWebWorker.js",
  //     taskConfiguration: {
  //       decodeTask: {
  //         codecsPath: "/codecs/cornerstoneWADOImageLoaderCodecs.js"
  //       }
  //     }
  //   };
  //   cornerstoneWADOImageLoader.webWorkerManager.initialize(config)
  //   cornerstoneWADOImageLoader.initialized = true
  // }

  export default {
    name: 'ImageTest',

    data: () => ({
      iload1: false,
      iload2: false,
      isdcm: false,
      isjpg: false,
      dialog1: false,
      finished: false,

      show1: false,
      password: '',
      rules: {
        required: value => !!value || 'Campo obligatorio',
        min: v => v.length >= 6 || '6 caracteres min',
      },
    }),

    computed: {
      isLoaded: function() {
        return this.iload1 && this.iload2
      }
    },

    methods: {

      newImageMed(file) {

        cornerstone.disable(this.$refs.dcmsource1)

        // Comprobar si la imagen es DICOM
        if( file.name.indexOf(".dcm")>=0 ) {
          this.isdcm = true
          this.isjpg = false

          // Enable Canvas
          this.canvas = this.$refs.dcmsource1

          cornerstone.enable(this.canvas, {
            renderer: "webgl"
          })

          // Load Image
          const imageId = cornerstoneWADOImageLoader.wadouri.fileManager.add(file)
          cornerstone.loadImage(imageId).then(image => {
            cornerstone.displayImage(this.canvas, image)
          });

          this.iload1 = true

          console.log( this.isdcm, this.isjpg )

        }
        // Si no es DICOM es jpg
        else {
          this.isdcm = false
          this.isjpg = true

          let imgurl = URL.createObjectURL(file)          
          let img = this.$refs.imgs1

          this.iload1 = true
          
          img.src = imgurl

          this.loadImg(1)
        }

      },

      newImageRep(file) {
          let imgurl = URL.createObjectURL(file)          
          let img = this.$refs.imgs2

          this.iload2 = true
          
          img.src = imgurl

          this.loadImg(2)  
      },

      loadImg(nim) {
        let src, img, h, w

        if( nim==1 ) {
          src = this.$refs.imgsource1
          img = this.$refs.imgs1          
        }
        if( nim==2 ) {
          src = this.$refs.imgsource2
          img = this.$refs.imgs2          
        }

        let ctx = src.getContext('2d')
        let wmax = 500

        img.onload = function() {
          if( img.width>wmax ) {
            h = img.height*(wmax/img.width)
            w = wmax
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
        let src1 
        if( this.isdcm )
          src1 = cv.imread(this.$refs.canvas1)
        else
          src1 = cv.imread(this.$refs.imgsource1)

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

        this.finished = true 
      },

      downloadImg() {
        let canvas = this.$refs.imgdest1
        let image = canvas.toDataURL("image/jpg").replace("image/jpg", "image/octet-stream")
        var link = document.createElement('a')
        link.download = "watermarked.jpg"
        link.href = image;
        link.click();
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
.dcmcpre {
  display: none;
}
.dcmc {
  width: 500px;
  height: 500px;
}
.bottom-space {
  height: 2rem;
}
</style>