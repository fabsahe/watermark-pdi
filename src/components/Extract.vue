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
        div(class="canvascon")
          canvas(id="imgsource3" ref="imgsource3" class="imgcanvas3")

    v-row
      v-col(cols="12" md="4")
        v-form(v-model="validpass2")
          v-text-field(
            v-model="password"
            :append-icon="show2 ? 'mdi-eye' : 'mdi-eye-off'"
            :rules="[rules.required, rules.min, rules.number]"
            :type="show2 ? 'text' : 'password'"
            name="input-10-2"
            label="Contraseña"
            hint="6 caracteres requeridos"
            counter
            @click:append="show2 = !show2"
          )
    v-row
      v-col(cols="12" md="12")
        v-btn(color="error" 
              :disabled="!isReady" 
              :dark="isReady" 
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
        canvas(id="imgdest2" ref="imgdest2" class="imgcanvas3" v-show="finished")

      v-col(cols="12" md="6")
        img(src="" ref="imgd3" class="img-base")
        canvas(id="imgdest3" ref="imgdest3" class="imgcanvas" v-show="finished")

    v-row
      v-col(cols="12" md="12")
        v-btn(color="success" 
              :disabled="!finished" 
              dark
              @click="downloadImg()"
              large) 
          v-icon mdi-download
          span Descargar

    v-row
      div(class="bottom-space")

</template>

<script>
  import * as cv from 'opencv.js'
  import * as fft from 'fft-js'

  export default {
    name: 'ImageTest',

    data: () => ({
      imgLoadedE: false,
      validpass2: false,
      finished: false,

      show2: false,
      password: '',
      rules: {
        required: value => !!value || 'Campo obligatorio',
        min: v => v.length == 6 || '6 caracteres requeridos',
        number: v=> !isNaN(v) || 'Sólo se permiten números',
      },
    }),

    computed: {
      isReady: function() {
        return this.imgLoadedE && this.validpass2
      }
    },

    methods: {

      newImage(file) {
        let imgurl = URL.createObjectURL(file)
        let img = this.$refs.imgs3

        img.src = imgurl
        this.imgLoadedE = true
        this.loadImg()
      },

      loadImg() {
        let src, img, h, w

        src = this.$refs.imgsource3
        img = this.$refs.imgs3

        let ctx = src.getContext('2d')

        // img.onload = function() {
        //   if( img.width>550 ) {
        //     h = img.height*(550/img.width)
        //     w = 550
        //     src.height = h
        //     src.width = w          
        //   }
        //   else {
        //     h = img.height
        //     w = img.width
        //     src.height = h
        //     src.width = w    
        //   }
        //   ctx.drawImage(img, 0, 0, w, h)
        // }

        img.onload = function() {
          h = img.height
          w = img.width
          src.height = h
          src.width = w          
          ctx.drawImage(img, 0, 0, w, h)
        }

      },

      genPass(k,h){
        k = parseInt(k);
        var Sv = new Array(h);
        var s = new Array(h);
        Sv[0] = k%Math.PI;
        var sign = 0;
        for (let i = 1; i < h-1; i++) {
            if (Math.abs(Sv[i-1])  > (Math.PI/2)) {
                sign = 1;
            }else if (Math.abs(Sv[i-1])  <= (Math.PI/2)){
                sign = -1;
            }
            Sv[i]=  sign * (Sv[i-1]+k+i)%Math.PI;     
        }

        for (let i = 0; i < h-1; i++) {
           if (Sv[i]>=0) {
               s[i] = 1;
           } 
           else{
               s[i]= 0;
           }
        }
        return s;
      },

      micoef(coef) {
        var f = 0.2589700;
        return  Math.cos(f * coef);
      },

      creaMatriz(w,h){
        var matrix = new Array(w);
        for(var i=0; i<w; i++) {
          matrix[i] = new Array(h);
        }
        return matrix;
      },

      mat2Array(mat,w,h){
        let a= this.creaMatriz(w,h);
        let cont = 0;
        for(let i=0; i<w; i++){
          for (let j=0; j < h; j++){
            a[i][j] = mat[cont];
            cont++;
          }
        } 
        return a;      
      },      

      start() {

        let medic = cv.imread(this.$refs.imgsource3)

        cv.cvtColor(medic, medic, cv.COLOR_RGBA2GRAY, 0); 

        let pass = this.password;
        let w = medic.rows;
        let h = medic.cols;

        let img = medic;


        let S = this.genPass(pass,(w/2)*(h/2))
        let imgEnc = new Array(h*w);
        for (var i = 0; i < h*w; i++){
          imgEnc[i] = img.data[i];
        }
      
        let bloqE =new cv.MatVector();
        imgEnc = this.mat2Array(imgEnc,w,h);

        for (let i = 0; i < w; i+=2) {
          for (let j = 0; j < h; j+=2) {
            let b = new cv.Mat(2,2,cv.CV_32F);
            b.data32F[0] = imgEnc[i][j];
            b.data32F[1] = imgEnc[i][j+1];
            b.data32F[2] = imgEnc[i+1][j];
            b.data32F[3] = imgEnc[i+1][j+1];
            bloqE.push_back(b);
            b.delete();
          }          
        }

        let rec = new Array((h/2)*(w/2));
        let contR= 1;
        //Aplicando algoritmo de extraccion
        //console.log(S);
        for (let i=1; i<bloqE.size();i++){
          let bi = bloqE.get(i);
          let ba = bloqE.get(i-1);
          let Vi = [bi.data32F[0], bi.data32F[1], bi.data32F[2],bi.data32F[3]];
          let Va = [ba.data32F[0], ba.data32F[1],ba.data32F[2],ba.data32F[3]];
          let fftI = fft.fft(Vi);
          let fftA = fft.fft(Va); 
          let r = this.micoef(fftI[0][0]-fftA[0][0]);

          r = r>=0?1:0;
          r = r^S[contR];
          r = r>0?255:0;
        
          rec[contR] = r;
          contR++;            
          }
        
        let R = cv.matFromArray(w/2,h/2,cv.CV_8U,rec)
   
        cv.imshow(this.$refs.imgdest2, R)
        this.finished = true 
      },

      nameId() {
        return '_' + Math.random().toString(36).substr(2, 9);
      },

      downloadImg() {
        let canvas = this.$refs.imgdest2
        let image = canvas.toDataURL("image/jpg").replace("image/jpg", "image/octet-stream")
        var link = document.createElement('a')
        let filename = "report_"
        link.download = filename.concat( this.nameId(), '.jpg' )
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
.bottom-space {
  height: 5rem;
}
.imgcanvas3 {
  width: 500px;
  height: 500px;
}

</style>