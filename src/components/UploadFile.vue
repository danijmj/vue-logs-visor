<template>
  <div id="dropContainer" class="dropzone">
    Suelta aquí los archivos
    <label for="fileInput" id="clickHere" class="labelClickHere">
      O seleccione aquí
      <input type="file" name="file" id="fileInput" v-on:change="loadFile($event.target.files)"/>
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
    /**
     * Method to load a file with the imput type file
     * @param {Files(s)} files, List of object with the imput files loaded with the input
     */
    loadFile: function loadFile(files) {

      // Be generate the DataTransfer object
      const dT = new DataTransfer();
      // We add the file(only the first item) into the object
      dT.items.add(files[0]);
      // Call the method to managment the file
      this.managmentFileData(dT.items)

    },
    /**
     * Method that start the managment of the upload file,
     * We have got the files, now we're goint to managment and read the file 
     * @param {DataTransfer.items} items, List (but only one) of files to load 
     */
    managmentFileData(items) {

      const that = this
      // We call the method that load the text
      that.getFileText(items).then(text => {

        // Remove the empty first, and last, empty spaces
        text = text.trim()
        // Remove the last ',' character
        if (text.endsWith(','))
        {
          let posLastCom = text.lastIndexOf(',')
          text = text.substring(0, posLastCom)
        }

        // Add the brackets if it don't have
        if (!text.startsWith('[') && !text.endsWith(']'))
        {
          text = `[${text}]`
        }
        // We convert (in the method) the text to a json object
        // After, we emit the change to the parent component
        that.checkIfFileJsonIsValid(text, (json) => {

          that.fileData = json
          // Emit the change
          that.$emit('update:modelJsonsource', that.fileData)
          alert("Texto leído correctamente")
          // that.jsonsource = JSON.parse(text)
          
        }, (error, json) => {console.log(error, json)} )
      }, (error) => {console.log(error)})

      // Remove the class 'dragover' ever
      this.dropContainer.classList.remove('dragover')
    },
    /**
     * Async Method that chapture the dropzone file
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

      // Events to detect if a file is over the area
      this.dropContainer.addEventListener("dragenter", (event) => {
        // highlight potential drop target when the draggable element enters it
        if (event.target.classList.contains("dropzone")) {
          event.target.classList.add("dragover");
        }
      });

      this.dropContainer.addEventListener("dragleave", (event) => {
        // reset background of potential drop target when the draggable element leaves it
        if (event.target.classList.contains("dropzone")) {
          event.target.classList.remove("dragover");
        }
      });

    },
    /**
     * Method that parse the json text into a json object and check if the json is a valid log  
     * @param {String} text, text with the (expected) json text
     * @param {Function} callback, callback to execute if the value is correct
     * @param {Function} error, callback to execute if the value is incorrect
     */
    checkIfFileJsonIsValid: function checkIfFileJsonIsValid(text, callback = () => {return void(0)}, error = () => {return void(0)}) { 
      let json = null;
      
      try {
        json = JSON.parse(text)
        
        let isTrue = true

        // Check if the object items has the structure
        Object.values(json).forEach(e => {
          if (
            !Object.hasOwn(e, "type") || 
            !Object.hasOwn(e, "url") || 
            !Object.hasOwn(e, "finded") || 
            !Object.hasOwn(e, "titn")
          ){
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
     * Methdo to load the files
     * @param {DataTransfer.items} items, List (but only one) of files to load 
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
            console.log("Texto leído")
            let text = reader.result
            if (!text) {
              alert("Error, el archivo está vacío")
              console.log("Error, el archivo está vacío")
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

  .dropzone {
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
    transition: border ease-in-out .4s, background-color ease-in-out .4s,;

  }
  .dropzone.dragover
  {
    border: 5px solid #42b983;
    /* transform: translate(-50%, -50%) scale(1.1, 1.1); */
    /* border-radius: 0; */
    background-color: rgb(233, 233, 233);
  }

  

  .dropzone input {
    /*Important*/
    position: absolute;
    /*Important*/
    cursor: pointer;
    left: 0px;
    top: 0px;
    /*Important This is only comment out for demonstration purposes. */
    opacity:0; 
  }

  @media screen and ( max-width: 750px ) {
    .dropzone {
      width: 80%;
      height: 80%;
      left: 10%;
      top: 10%;
      transform: none;
      font-size: 20px;
    }
  }

  /*Important*/
  .dropzone.mouse-over {
    border: 2px dashed rgba(0,0,0,.5);
    color: rgba(0,0,0,.5);
  }

  /*If you dont want the button*/
  .labelClickHere {
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

  .labelClickHere:hover {
    background-color: #20583f;
  }

</style>
