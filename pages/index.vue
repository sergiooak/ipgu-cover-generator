<template>
  <div class="wrapper h-screen flex flex-col">
    <header>
      <div class="container mx-auto">
        <h1 class="text-center text-lg text-white py-8">Gere as thumbnails para cursos do IPGU facilmente</h1>
      </div>
    </header>
    <main class="flex-1">
      <div class="container mx-auto">
        <div class="bg-white rounded-lg p-4 shadow-lg flex flex-col md:flex-row-reverse">
          <form class="md:w-2/6 flex flex-col">
            <label class="block">
              <span class="text-gray-700">Número TSV:</span>
              <input class="form-input mt-1 block w-full" v-model="form.tsv">
            </label>
            <label class="block">
              <span class="text-gray-700">Título:</span>
              <textarea class="form-textarea mt-1 block w-full" rows="3" v-model="form.titulo"></textarea>
            </label>
            <label class="block">
              <span class="text-gray-700">Cidade:</span>
              <input class="form-input mt-1 block w-full" v-model="form.cidade">
            </label>
            <label class="block relative flex-1 flex flex-col">
              <span class="text-gray-700">Imagem:</span>
              <div class="relative mt-1 border-dashed border-2 p-4 px-16 flex justify-center items-center rounded-lg border-gray-300 flex-1 -mb-12">
                <svg class="h-20 text-gray-400 mr-2" viewBox="0 0 24 24" fill="currentColor" xmlns="http://www.w3.org/2000/svg"><path d="M11 14.9861C11 15.5384 11.4477 15.9861 12 15.9861C12.5523 15.9861 13 15.5384 13 14.9861V7.82831L16.2428 11.0711L17.657 9.65685L12.0001 4L6.34326 9.65685L7.75748 11.0711L11 7.82854V14.9861Z" fill="currentColor" /><path d="M4 14H6V18H18V14H20V18C20 19.1046 19.1046 20 18 20H6C4.89543 20 4 19.1046 4 18V14Z" fill="currentColor" /></svg>
                <span class="text-center text-gray-700 font-medium leading-tight">Arraste solte uma imagem, ou clique para procurar...</span>
              </div>
              <input type="file" class="dropzone mt-1 block w-full" style="height: 42px; padding: 6px;" @change="updateImage">
            </label>
          </form>
          <div class="canvasWrapper md:w-4/6 pt-4 md:pt-0 md:pr-4 border-t-2 md:border-t-0 md:border-r-2 border-gray-300 mt-4 md:mt-0 md:mr-4">
            <canvas class="w-full bg-gray-300" :width="canvas.width" :height="canvas.height" id="canvas" />
          </div>
        </div>
      </div>
    </main>
    <footer>
      <div class="container mx-auto">
        Tecnologia por Sergio Carvalho
      </div>
    </footer>
  </div>
</template>

<script>
export default {
  data: function (){
    return {
      form: {
        initialHeight: 212,
        tsv: 'TSV1',
        titulo: 'Energias',
        cidade: 'Uberlândia',
        imagem: null,
      },
      canvas: {
        ctx: null,
        width: 878,
        height: 500
      }
    }
  },
  computed: {
    tituloParsed: function () {
      return this.form.titulo.split('\n')
    },
    biggestWord(){
      let x = 0;
      for (var word of this.tituloParsed) {
        x = word.length > x ? word.length : x;
      }
      return x;
    },
    sizeTitulo: function () {
      if (this.biggestWord <= 8) {
        return 9;
      }else if(this.biggestWord <= 10) {
        return 7
      }else if(this.biggestWord <= 12) {
        return 6
      }else if(this.biggestWord <= 15) {
        return 5
      }else if(this.biggestWord <= 19) {
        return 4
      }else{
        return 3
      }
    },
    initialHeight(){
      if(this.tituloParsed.length <= 1){
        return 210;
      }else if(this.tituloParsed.length == 2){
        return 190;
      }else if(this.tituloParsed.length == 3){
        return 170;
      }else if(this.tituloParsed.length == 4){
        return 140;
      }else if(this.tituloParsed.length == 5){
        return 90;
      }else{
        return 50;
      }
    },
  },
  methods:{
    calcCrop(x,y){
      console.log(`x:${x} y:${y}`);
      let biggestSize = x > y ? 'x' : 'y';
      let sx = 0;
      let sy = 0;
      let scale = 0;
      if (biggestSize == 'x') {
        scale = y / this.canvas.height;
        sx = (x - 660 * scale) / 2;
        x = 660 * scale;
      }else{
        scale = x / 660;
        sy = (y - this.canvas.height * scale) / 2;
        y = this.canvas.height * scale;          
      }
      return {sx: sx, sy: sy, sWidth: x, sHeight: y};
    },
    reset(img) {
      let cx = this;
      let canvas = this.canvas;
      this.canvas.ctx.clearRect(0, 0, this.width, this.height);
      let i = new Image();
      i.onload = function(){
        let axis = cx.calcCrop(i.width, i.height);
        console.log(axis);
        canvas.ctx.drawImage(i, axis.sx, axis.sy, axis.sWidth, axis.sHeight, 0, 0, 660, canvas.height);
        cx.image = i;
        cx.applyBase();
      }
      i.src = img ? img : 'generator/temp.png';
    },
    applyBase(){
      let cx = this;
      let canvas = this.canvas
      let img = new Image();
      img.onload = function(){
        canvas.ctx.drawImage(img,0,0, canvas.width, canvas.height);
        cx.applyTexts();
      }
      img.src = 'generator/base.png';
    },
    applyTexts(){
      this.canvas.ctx.font = '48px MoolBoran';
      this.canvas.ctx.textAlign="left";
      this.canvas.ctx.fillStyle = '#C52528';
      this.canvas.ctx.strokeStyle = '#FFF';
      this.canvas.ctx.lineWidth = 5;
      this.canvas.ctx.strokeText(this.form.cidade, 35, 60);
      this.canvas.ctx.fillText(this.form.cidade, 35, 60);

      this.canvas.ctx.font = '48px MoolBoran';
      this.canvas.ctx.textAlign="right";
      this.canvas.ctx.fillStyle = '#C52528';
      this.canvas.ctx.fillText(this.form.tsv, 843, this.initialHeight);


      for (var i = 0; i < this.tituloParsed.length; i++) {
        let height = this.sizeTitulo;
        let y = (this.initialHeight + (height * 5)) + (i * (height * 5));
        this.canvas.ctx.font = `${height}ex MoolBoran`;
        this.canvas.ctx.fillStyle = '#004280';
        this.canvas.ctx.fillText(this.tituloParsed[i], 843, y);
      }
    },
    updateImage(e) {
      let cx = this;
      let canvas = this.canvas
      let reader = new FileReader();
      reader.onload = function(event){
        let img = new Image();
        cx.reset(event.target.result)
      }
      reader.readAsDataURL(e.target.files[0]);
    }
  },
  mounted() {
    let c = document.getElementById("canvas");
    let ctx = c.getContext("2d");
    this.canvas.ctx = ctx;

    this.reset();
  },
  watch: {
    form: {
      handler(){
        this.reset();
      },
    deep: true
    }
  }
}
</script>

<style>
  body{
    @apply bg-blue-800;
  }

  .dropzone{
    visibility: hidden;
  }

  .dropzone::before{
    visibility: visible;
    content: '';
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    cursor: pointer;
  }
</style>
