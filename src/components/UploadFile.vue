<template>
  <div id="dropContainer">
    Suelta aquí los archivos
    <label for="fileInput" id="clickHere">
      O seleccione aquí
      <input type="file" name="file" id="fileInput" v-on:change="loadFile($event.target.value)"/>
    </label>
  </div>
</template>

<script>
export default {
  name: 'upload-file',
  props: {
    modelJsonsource: Array
  },
  data() {
      return {
        fileData: Array,
        dropContainer: null,
        fileInput: null,
        listPossibleTypes: [
          "text/plain",
          "text/json",
        ]
      }
  },
  mounted() {
    this.fileData = this.modelJsonsource
    this.dropContainer = document.getElementById("dropContainer")
    this.fileInput = document.getElementById("fileInput")
    this.load();
  },
  emits: ['update:modelJsonsource'],
  methods: {
    loadFile: function loadFile(file) {
      console.log(file)
    },
    managmentFileData(items) {

      const that = this

      that.getFileText(items).then(text => {

        text = text.trim()
        // Remove the las ','
        if (text.endsWith(','))
        {
          let posLastCom = text.lastIndexOf(',')
          text = text.substring(0, posLastCom)
        }

        // Add the brackets if don't have
        if (!text.startsWith('[') && !text.endsWith(']'))
        {
          text = `[${text}]`
        }

        that.checkIfFileJsonIsValid(text, (json) => {

          that.fileData = json
          that.$emit('update:modelJsonsource', that.fileData)
          alert("Texto leído correctamente")
          // that.jsonsource = JSON.parse(text)
          
        }, (error, json) => {console.log(error, json)} )
      }, (error) => {console.log(error)})
    },
    /**
     * Method that chapture the dropzone file
     */
    async load() 
    {
      this.dropContainer.ondragover = this.dropContainer.ondragenter = function(evt) {
        evt.preventDefault();
      };

      const that = this
      // Dropdown event
      this.dropContainer.ondrop = function(evt) {
        // pretty simple -- but not for IE :(
        // this.fileInput.files = evt.dataTransfer.files;

        // If you want to use some of the dropped files
        const dT = new DataTransfer();

        if (!evt.dataTransfer.files && evt.dataTransfer.files.constructor != Array && evt.dataTransfer.files.length < 1)
        {
          return;
        }
        
        dT.items.add(evt.dataTransfer.files[0]);

        that.managmentFileData(dT.items)

        /* if (evt.dataTransfer.files && evt.dataTransfer.files.constructor == Array && evt.dataTransfer.files.length > 2)
        {
          dT.items.add(evt.dataTransfer.files[3]);
        } */
        /* this.fileInput.files = dT.files; */

        evt.preventDefault();
      };
    },
    /**
     * 
     * @param {*} text 
     * @param {*} callback 
     * @param {*} error 
     */
    checkIfFileJsonIsValid: function checkIfFileJsonIsValid(text, callback = () => {return void(0)}, error = () => {return void(0)}) { 
      let json = null;
      
      try {
        // json = JSON.stringify(text);
        console.log(text)
        json = JSON.parse(text)
        
        let isTrue = true

        Object.values(json).forEach(e => {
          if (
            !Object.hasOwn(e, "type") || 
            !Object.hasOwn(e, "url") || 
            !Object.hasOwn(e, "finded") || 
            !Object.hasOwn(e, "titn")
          ){
            // console.log("El json no tiene el formato adecuado")
            isTrue = false
          }
        })
        if (!isTrue) {
          error("Json no válido", json)
          alert("Json no válido") 
        }
        callback(json)
        return isTrue
        
      } catch (e) { 
        error("Json no válido: " + e, json)
        alert("Json no válido: " + e) 
      }
      error("Json no válido", json)
      return false
    },
    /**
     * 
     * @param {*} items 
     */
    getFileText: function getFileText(items) {

      const promise = new Promise((resolve, reject) => {
        
        // The necesary consts
        const that = this
        // We inizialize the FileReader object
        const reader = new FileReader();
        
        Object.values(items).forEach(it => {

          if (!that.listPossibleTypes.includes(it.type))
          {
            console.log("Tipo de archivo no permitido")
            alert("Tipo de archivo no permitido")
            reject("Tipo de archivo no permitido")
          }
          try {
            // We convert the DataTranfer/File object to a File object
            const file = it.getAsFile()
            // We a FileRead object to read the text
            // The file will start to read (Async)
            reader.readAsText(file);

          } catch(error) {
            console.log("error: " + error)
            alert("error: " + error)
            reject("error: " + error)
          }
          // let textFile = it.prototype.getAsString()
          
        });

        // Event handle when the file has been rode
        reader.addEventListener(
          "load",
          () => {
            // this will then display a text file
            // content.innerText = reader.result;
            console.log("Texto leído: " + reader.result)
            let text = reader.result
            if (!text) {
              alert("Error, the file is empty")
              console.log("Error, the file is empty")
              reject("Error, el archivo está vacío")
            }
            resolve(text)

          },
          false,
        );
      });

      return promise

    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

  #dropContainer {
    width: 40%;
    height: 40%;
    left: 50%;
    top: 50%;
    background-color: rgb(255 255 255);
    transform: translate(-50%, -50%);
    border: 2px dashed #42b983;
    border-radius: 20px;
    font-family: Arial;
    text-align: center;
    position: relative;
    line-height: 180px;
    font-size: 20px;
    color: rgb(0 0 0);
    transition: all ease-in-out .4s;

    &::drop
    {
      border: 2px dashed #42b983;
    }

  }

  #dropContainer input {
    /*Important*/
    position: absolute;
    /*Important*/
    cursor: pointer;
    left: 0px;
    top: 0px;
    /*Important This is only comment out for demonstration purposes. */
    opacity:0; 
  }

  /*Important*/
  #dropContainer.mouse-over {
    border: 2px dashed rgba(0,0,0,.5);
    color: rgba(0,0,0,.5);
  }

  /*If you dont want the button*/
  #clickHere {
    position: absolute;
    cursor: pointer;
    left: 50%;
    top: 50%;
    margin-top: 20px;
    line-height: 31px;
    color: white;
    font-size: .8rem;
    width: auto;
    text-transform: uppercase;
    height: auto;
    padding: 5px 20px;
    border-radius: 4px;
    transition: background-color ease .4s;
    background-color: #42b983;
    transform: translate(-50% );
    cursor: pointer;
  }

  #clickHere:hover {
    background-color: #20583f;
  }

</style>
