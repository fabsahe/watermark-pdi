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
        v-form(v-model="validpass")
          v-text-field(
            v-model="password"
            :append-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'"
            :rules="[rules.required, rules.min, rules.number]"
            :type="show1 ? 'text' : 'password'"
            name="input-10-1"
            label="Contraseña"
            hint="6 caracteres requeridos"
            counter
            @click:append="show1 = !show1"
          )

    v-row
      v-col(cols="12" md="12")
        v-btn(color="error" 
              :disabled="!isReady" 
              :dark="isReady" 
              @click="start()"
              large) Insertar

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
  import * as fft from 'fft-js'

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
    name: 'Insert',

    data: () => ({
      iload1: false,
      iload2: false,
      isdcm: false,
      isjpg: false,
      dialog1: false,
      finished: false,
      validpass: false,

      show1: false,
      password: '',
      rules: {
        required: value => !!value || 'Campo obligatorio',
        min: v => v.length == 6 || '6 caracteres requeridos',
        number: v=> !isNaN(v) || 'Sólo se permiten números',
      },
    }),

    computed: {
      isReady: function() {
        return this.iload1 && this.iload2 && this.validpass
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

      funM(coef, w,imp){
            let l=w==1?0:1;

            let f =  0.2589700;
            let t = (f*coef)/(2*3.1416);
            let p = Math.round(t);
            let Mf = (3.1416/f)*(l+2*p);

            if(imp==1){
              // console.log(f);
              // console.log(t);
              // console.log(p);
              // console.log(Mf);
            }
            return Mf;
      },

      micoef(coef) {
        var f = 0.2589700;
        return  Math.cos(f * coef);
      },

      resizeImg(s,w,h){
        let nuevo = s; 
        let dsize = new cv.Size(h,w);
        cv.resize(nuevo,nuevo, dsize, 0, 0, cv.INTER_AREA);
        return nuevo;
      },

      reporteBin(rep,w,h) {
        let tmp = new Array(w*h);
        let contT=0;          
        for (let i = 0; i < w; i++){
          for (let j = 0; j <h; j++){

            let rs =  rep.ucharPtr(i,j)[0];
            if(rs>=127){
              rs = 1;
            }else{
              rs =0;
            }
            
            tmp[contT] = rs;
            contT++;
            }
        }

        return tmp;
      },

      creaMatriz(w,h){
        var matrix = new Array(w);
        for(var i=0; i<w; i++) {
          matrix[i] = new Array(h);
        }
        return matrix;
      },

      toArray(mat,w,h, min){
        
        let arr = new Array(w*h);
        let cont=0;
        for(let i=0; i<w; i++) {
          for(let j=0; j < h; j++){
            let r =Math.round( mat[i][j]- min);
            arr[cont]=r>255?255:r;
            cont +=1;
          }
        }
        return arr;
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
        setTimeout( this.run(), 0 )
      },

      run() {

        let medic
        if( this.isdcm )
          medic = cv.imread(this.$refs.canvas1)
        else
          medic = cv.imread(this.$refs.imgsource1)

        let report = cv.imread(this.$refs.imgsource2)

        cv.cvtColor(medic, medic, cv.COLOR_RGBA2GRAY, 0); 
        cv.cvtColor(report, report, cv.COLOR_RGBA2GRAY, 0); 
        cv.threshold(report, report, 127, 255, cv.THRESH_TOZERO);

        let w = report.rows;
        let h = report.cols;

        medic =  this.resizeImg(medic,w*2,h*2);
        w = medic.rows;
        h = medic.cols;

        report = this.resizeImg(report,w/2,h/2);
        let T = this.reporteBin(report,w/2,h/2);
        let S = this.genPass(this.password,(w/2)*(h/2));

        //Pasar img a un array 
        let imgA = new Array(h*w);
        for (var i = 0; i < h*w; i++){
          imgA[i] = medic.data[i];
        }
        //Pasar array a matriz
        let img = this.mat2Array(imgA,w,h);

        //Obtener bloques
        let bloques =new cv.MatVector();

        for (let i = 0; i < w; i+=2) {
          for (let j = 0; j < h; j+=2) {
            let b = new cv.Mat(2,2,cv.CV_32F);
            b.data32F[0] = img[i][j];
            b.data32F[1] = img[i][j+1];
            b.data32F[2] = img[i+1][j];
            b.data32F[3] = img[i+1][j+1];
           
            bloques.push_back(b);
            b.delete();
          }          
        }

        let im=0;

        //Aplicando el algoritmo del
        let DcW = new cv.MatVector(); 
        let mp = new cv.Mat.zeros(2,2,cv.CV_32F);
        let conT = 1;
        DcW.push_back(mp);
        for(let i =1;i<bloques.size(); i++){
          let Dc = bloques.get(i);
          let DcWi= DcW.get(i-1);
          let tDc = [Dc.data32F[0],Dc.data32F[1],Dc.data32F[2],Dc.data32F[3]]
          let tDcW = [DcWi.data32F[0], DcWi.data32F[1],DcWi.data32F[2],DcWi.data32F[3]];
          let fftDc = fft.fft(tDc);
          let fftDcW = fft.fft(tDcW);  
          let ac = fftDc[0][0];
          let an = fftDcW[0][0];
          let res = ac-an;

          let tmp = [[0,0],[0,0],[0,0],[0,0]];
          tmp[0][0] = Math.round(an + (this.funM(res,T[conT]^S[conT],im))); 
          let iff = fft.ifft(tmp);
          let be = new cv.Mat(2,2,cv.CV_32F);
          be.data32F[0] =iff[0][0];
          be.data32F[1] =iff[1][0];
          be.data32F[2] =iff[2][0];
          be.data32F[3] =iff[3][0];

          conT++;
          DcW.push_back(be);
          Dc.delete();
          DcWi.delete();

        }

        //Recuperando img desde bloques
        
        let imR = this.creaMatriz(w,h);


        let menor = 100;
        let mayor = -100;
        let contB=0;
        for (let i = 0; i <w; i+=2){
          for (let j = 0; j <h;j+=2){
            let r = DcW.get(contB);
            imR[i][j] = r.data32F[0];
            imR[i][j+1] = r.data32F[1];
            imR[i+1][j] = r.data32F[2];
            imR[i+1][j+1] = r.data32F[3];

            menor = menor>=imR[i][j]?imR[i][j]:menor;
            menor = menor>=imR[i][j+1]?imR[i][j+1]:menor;
            menor = menor>=imR[i+1][j]?imR[i+1][j]:menor;
            menor = menor>=imR[i+1][j+1]?imR[i+1][j+1]:menor;

            mayor = mayor<=imR[i][j]?imR[i][j]:mayor;
            mayor = mayor<=imR[i][j+1]?imR[i][j+1]:mayor;
            mayor = mayor<=imR[i+1][j]?imR[i+1][j]:mayor;
            mayor = mayor<=imR[i+1][j+1]?imR[i+1][j+1]:mayor;

            contB+=1;
          }
        }

        let imA = this.toArray(imR,w,h,menor);
        let CV = cv.matFromArray(w,h,cv.CV_8U,imA)

        // cv.imshow(this.$refs.imgsource1, Medica);
        // cv.imshow(this.$refs.imgsource2, CV);
        let Des = this.desencripta(CV,w,h);
        let R = cv.matFromArray(w/2,h/2,cv.CV_8U,Des)
        cv.imshow(this.$refs.imgdest1, R);

        this.finished = true 
      },

      desencripta(img,w,h) {

        let S = this.genPass(this.pass,(w/2)*(h/2))
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

        let Rec = new Array((h/2)*(w/2));
        let contR= 1;
        //Aplicando algoritmo de extraccion

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
        
          Rec[contR] = r;
          contR++;            
        }
        return Rec;     
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