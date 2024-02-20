<template>
  <div class="hello">
    <h1 class="subtitle">{{ msg }}</h1>
    <div class="container">
      <a class="btn btnUploadFile" href="javascript:void(0)" @click="mostrarUpload = !mostrarUpload">
        Leer nuevo archivo de logs (formato json)
      </a>
      <Transition>
        <div v-show="mostrarUpload" class="cont-upload-file">
          <!-- <UploadFile :model-jsonsource="jsonsource"  @update:model-jsonsource="newValue => jsonsource = newValue"></UploadFile> -->
          <UploadFile v-model:model-jsonsource="jsonsource"></UploadFile>
          <span class="close-upload" @click="mostrarUpload = !mostrarUpload"></span>
        </div>
      </Transition>
    </div>
    <div class="container">
      <div class="row">
        <div class="col-6 ">
          <div class="selstyle">
            <select @change="filter()" class="" v-model="selectedtyperesource" id="selecttyperesource">
              <option :value="-1">Tipo de elementos</option>
              <option :value="0">Pdf</option>
              <option :value="1">Imagen</option>
            </select>
          </div>
        </div>
        <div class="col-6">
          <div class="selstyle">
          <select @change="filter()" class="" v-model="selectedstate" id="selectedstate">
            <option :value="-1">Estado del recurso</option>
            <option :value="true">Correcto</option>
            <option :value="false">Incorrecto</option>
          </select>
        </div>
        </div>
      </div>
    </div>
    <h2 class="mt-4">
      Resultados ({{ jsonsourceshow.length }})
    </h2>
    <div>
      <div class="cont" v-if="jsonsourceshow != null" >
        <div class="item" :key="index" v-for="(item, index) in jsonsourceshow.slice(0, showNumItems)">
          <details>
            <summary :class="{ encontrado: item.finded }">
              <span class="type">{{ types[item.type] }}</span>, {{ item.titn }} 
            </summary>            
            <p class="body">
              <span class="titn"><span class="ttle">TITN:</span> <a :href="'https://opac.inap.es/opac/abnetcl.exe?TITN=' + item.titn" target="_blank">{{ item.titn }}</a> </span>
              <span class="finded"><span class="ttle">RECURSO ENCONTRADO:</span> {{ encontradoArr[item.finded] }}</span>
              <span class="url"><span class="ttle">URL:</span> <a :href="item.url" target="_blank">{{ item.url }}</a></span>
              <span class="type"><span class="ttle">TIPO:</span> {{ types[item.type] }}</span>
            </p>
          </details>
        </div>
        <div class="text-center">
          <button @click="showNumItems += showNumItemsIncrement" class="btn btn-loadmore">Cargar más</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import UploadFile from './UploadFile.vue';

export default {
    name: 'check-items',
    props: {
      msg: String,
      subMsg: String
    },
    data() {
      return {
        jsonsource: [],
        jsonsourceshow: [],
        selectedtyperesource: -1,
        selectedstate: -1,
        types: {
          0: "Pdf",
          1: "Imagen"
        },
        encontradoArr: {
          false: "NO",
          true: "SI"
        },
        mostrarUpload: false,
        showNumItemsIncrement: 40,
        showNumItems: 40
      };
    },
    mounted() {
      // Load the file
      this.load();
      // Check if the user has clicked outside 
      this.checkOutsideUploadFileArea()
    },
    methods: {
      /**
       * Method to manage the click event to check if the user has clicked outside of the UploadFile area
       */
       checkOutsideUploadFileArea () {
        const that = this;
        let body = document.querySelector('body')
        let item = document.querySelector('.cont-upload-file')
        let btn = document.querySelector('.btnUploadFile')
        body.addEventListener('click', (ev) => {
          if (ev.target == item && ev.target != btn) {
            that.mostrarUpload = false
          }
        });
      },
      /**
       * Method to filter the data
       */
      filter: function filter() {
        this.showNumItems = this.showNumItemsIncrement
        // We check firt if the json has data
        if (this.jsonsource == null) { return; }
        // Load the original json data into the copy
        this.jsonsourceshow = [...this.jsonsource];
        // We filter the data
        if (this.selectedtyperesource != -1) {
          // console.log("this.jsonsource: " + this.jsonsource);
          this.jsonsourceshow = this.jsonsourceshow.filter(e => e.type === this.selectedtyperesource);
        }
        if (this.selectedstate != -1) {
          this.jsonsourceshow = this.jsonsourceshow.filter(e => e.finded === this.selectedstate);
        }

      },
      /**
       * Async method to load the current source file
       */
      async load() {
        await fetch("/sources.json")
          .then((res) => this.jsonsource = res.json())
          .then(res => { this.jsonsource = res; this.jsonsourceshow = res; })
          .catch((e) => console.error(e));
      }
    },
    components: { UploadFile },
    watch: {
      // watching top-level property is changed
      jsonsource: {
        handler(val) {
          // console.log('jsonsource changed')
          if (val && val.constructor === Array)
          {
            // console.log("jsonsource handler arrived")
            this.jsonsourceshow = [...val]
            this.mostrarUpload = false
            this.filter()
          }
        },
        deep: true
      }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

  /**
  Upload file
  */
  .close-upload {
    position: absolute;
    right: calc(30% - 45px);
    width: 45px;
    height: 45px;
    background: #42b983;
    transition: all ease .4s;
    border-radius: 50%;
    top: calc(30% - 45px);
    cursor: pointer;

    &:after {
      content: " ";
      display: block;
      width: 36px;
      height: 2px;
      position: relative;
      background: #fff;
      transform: rotate(45deg);
      top: 20px;
      right: -5px;
    }

    &::before {
      content: " ";
      display: block;
      width: 36px;
      height: 2px;
      position: relative;
      background: #fff;
      transform: rotate(-45deg);
      top: 22px;
      right: -5px;
    }

    &:hover {
      border-radius: 0;
      background: #20583f;
    }
  }
  @media screen and ( max-width: 750px ) {
    .close-upload {
      position: absolute;
      right: calc(10% - 20px);
      top: calc(10% - 20px);
      transform: scale(.5);

      &::after {
        transform: scale(.5);
      }

      &::before {
        transform: scale(.5);
      }

      &:hover {
        border-radius: 0;
        background: #20583f;
      }
    }
  }
  .cont-upload-file {
    position: fixed;
    width: 100svw;
    height: 100svh;
    display: block;
    top: 0;
    background-color: rgb(231 231 231 / 94%);
    overflow: hidden;
    z-index: 10;
  }

  .btn {
    padding: 20px;
    font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
    font-size: large;
    font-weight: 600;
    background-color: #e4e8f3;
    display: inline-block;
    clear: both;
    border: 1px solid transparent;
    color: #42b983;
    transition: all ease-in-out .4s;
    cursor: pointer;
    &:hover {
      color: #20583f;
      border: 1px solid #20583f;
      background-color: #c3c6cf;
    }
  }

  .btn-loadmore {
    margin: 30px 0;
  }

  .btnUploadFile {
    margin-bottom: 30px;
  }
  


  /** Others */
  .row {
    display: flex;
    flex-wrap: wrap;
    flex-direction: row;
    margin: 0 auto;
    max-width: 600px;
  }
  .col-6 {
    flex: 1 0 0%;
    padding: 0 10px;
    width: 100%;
    max-width: 100%;
  }
  *::after, *::before {
    box-sizing: border-box;
  }

  h3 {
    margin: 40px 0 0;
  }

  .subtitle {
    font-size: x-large;
    position: relative;
    margin: 20px auto 40px;

  }

  ul {
    list-style-type: none;
    padding: 0;
  }
  li {
    display: inline-block;
    margin: 0 10px;
  }
  a {
    color: #42b983;
    text-decoration: none;
  }
  .cont {
    padding: 20px 40px;
    text-align: left;

    .item {
      margin: 15px 0;

      summary {
        padding: 20px;
        font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
        font-size: large;
        font-weight: 600;
        background-color: #e4e8f3;
      }

      summary.encontrado {
      background-color: #9fd7be;
    }
      .body {
        font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
        font-size: medium;
        font-weight: normal;
        margin-top: 1px solid #c0c0c0;
        transition: all ease-in-out .5s;

        span:not([class="ttle"]) {
          display: block;
          padding: 5px;
          transition: all ease-in-out .5s;
          font-weight: 600;
          color: rgb(82, 82, 82);
          .ttle {
            color: rgb(82, 82, 82);
            margin-right: 5px;
            font-weight: normal;
          }
        }

        
        span:not([class="ttle"]):hover {
          background: #e6e6e6;
        }
      }
    }
  }



  details summary  {
    margin-bottom: -10px; /* for more prominent move */
    transition: margin 150ms ease-out;
  }

  details[open] summary {
    margin-bottom: 5px;
  }

  details[open] summary ~ * {
    /* animation: sweep .5s ease-in-out; */
    margin-bottom: 0px;
  }


  .selstyle {
    position: relative;
    margin: 5px 0;
  }

  .selstyle select {
    background-color: #42b983;
    color: white;
    padding: 12px 16px 12px;
    width: 250px;
    border: none;
    font-size: 20px;
    box-shadow: 0 5px 25px rgba(0, 0, 0, 0.2);
    -webkit-appearance: button;
    appearance: button;
    outline: none;
  }


  .selstyle:hover::before {
    color: rgba(255, 255, 255, 0.6);
    background-color: rgba(255, 255, 255, 0.2);
  }

  .selstyle select option {
    padding: 30px;
    border-radius: 0;
  }

  .mt-4 {
    margin-top: 60px;
  }
  
  @keyframes sweep {
    0%    {opacity: 0; margin-left: -10px}
    100%  {opacity: 1; margin-left: 0px}
  }

</style>

<style>
  .text-center {
    text-align: center;
  }
  .v-enter-active,
  .v-leave-active {
    transition: opacity 0.4s ease;
  }

  .v-enter-from,
  .v-leave-to {
    opacity: 0;
  }
</style>