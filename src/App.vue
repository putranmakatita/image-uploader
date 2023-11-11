<template>
  <div id="uploadingAlert" class="alert alert-primary" role="alert" hidden>
    uploading...
  </div>
  <div>
    <div class="d-flex justify-content-center mb-5">
      <h3>Image Upload App</h3>
      <!-- <a href="https://github.com/azeemade/image-upload-vue" class="btn border-bottom border-primary border-0">
        <i class="bi bi-github"></i>
      </a> -->
    </div>
    <div class="container">
      <div class="row mb-5">
        <div class="col-md-4 mb-3">
          <h5>Uploader</h5>
          <image-upload action="create" title="Choose image" :image_style="{
            border: 'rounded-circle',
            height: '100',
            width: '100'
          }" />
          <button type="submit" @click="uploadCreateSingle" class="btn btn-sm btn-success">Upload</button>
        </div>
        <!-- <div class="col-md-8 d-grid justify-content-center">
          <h5>Multi create</h5>
          <image-upload :upload_type="'multi'" action="create" />
          <button type="submit" @click="uploadCreateMulti" class="btn btn-sm btn-success">Upload</button>
        </div> -->
      </div>
      <div class="row mb-3">
        <!-- <div class="col-md-4 mb-3">
          <h5>Single edit</h5>
          <image-upload action="edit" :url="singImg" title="Change image" :image_style="{
            height: '100',
            width: '100'
          }" />
          <button type="submit" @click="uploadEditSingle" class="btn btn-sm btn-success">Upload</button>
        </div> -->
        <div class="col-md-8 d-grid justify-content-center">
          <h5>Gallery</h5>
          <image-upload :upload_type="'multi'" action="edit" :urls="images" :image_style="{
            height: '100',
            width: '100'
          }" />
          <!-- <button type="submit" @click="uploadEditMulti" class="btn btn-sm btn-success">Upload</button> -->
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import imageUpload from './components/image-upload.vue'
import { mapGetters } from 'vuex'
import axios from "axios"

export default {
  components: { imageUpload },
  name: "App",
  data() {
    return {
      singImg: localStorage.getItem('singleImg') ? localStorage.getItem('singleImg') : "https://github.com/azeemade/image-upload-vue/blob/main/src/assets/default.png?raw=true",
      images: JSON.parse(localStorage.getItem('images')) || []
    }
  },

  methods: {
    uploadCreateSingle() {
      if (!this.createSingle) {
        console.error('No file selected');
        return;
      }

      // Create a FileReader
      const reader = new FileReader();

      // Define a callback function to handle the FileReader's result
      reader.onload = () => {
        // Get the base64 string
        const base64String = reader.result.split(',')[1];

        // Send the base64 string to ImgBB
        this.sendToImgBB(base64String);
      };

      // Read the content of the selected file as a data URL
      reader.readAsDataURL(this.createSingle);
    },

    sendToImgBB(base64String) {
      let alert = document.getElementById('uploadingAlert');
      alert.removeAttribute('hidden');
      let formData = new FormData();
      formData.append('image', base64String);

      axios.post('https://api.imgbb.com/1/upload', formData, {
        headers: {
          'Content-Type': 'multipart/form-data',
        },
        params: {
          key: 'd810bd47f24cc3aa488206f99be4cdb7', // Replace with your actual API key
        },
      })
        .then(response => {
          console.log('Image uploaded successfully:', response.data);
          // alert.removeAttribute('hidden');
          let multiStore = JSON.parse(localStorage.getItem('images')) || [];
          // Add the new image URL to the array
          multiStore.push(response.data.data.url);
          localStorage.setItem('images', JSON.stringify(multiStore));
          localStorage.setItem('singleImg', response.data.data.url);
          location.reload();
        })
        .catch(error => {
          console.error('Error uploading image:', error);
          // Handle the error as needed
        });
    },


    uploadCreateMulti() {
      if (!this.createMulti.length < 1) {
        console.error('No file selected');
        return;
      }

      let formData = new FormData();
      for (const i of Object.keys(this.createMulti)) {
        formData.append(`createMulti`, this.createMulti[i]);

        // Create a FileReader
        const reader = new FileReader();

        // Define a callback function to handle the FileReader's result
        reader.onload = () => {
          // Get the base64 string
          const base64String = reader.result.split(',')[1];

          // Send the base64 string to ImgBB
          this.sendToImgBB(base64String);
        };

        // Read the content of the selected file as a data URL
        reader.readAsDataURL(this.createMulti[i]);
      }
    },

    uploadEditSingle() {
      let formData = new FormData();
      formData.append("updateSingle", this.updateSingle);
      /*axios.post(``, formData, {headers: {'Content-Type': 'multipart/form-data'}})
      .then(response => {
          console.log(response.data);
      })*/
    },

    uploadEditMulti() {
      let formData = new FormData();
      for (const i of Object.keys(this.updateMulti)) {
        formData.append(`updateMulti`, this.updateMulti[i]);
      }
      /*axios.post(``, formData, {headers: {'Content-Type': 'multipart/form-data'}})
      .then(response => {
          console.log(response.data);
      })*/
    }
  },

  computed: {
    ...mapGetters([
      "createSingle", "updateSingle", "createMulti", "updateMulti"
    ]
    )
  }
};
</script>

<style>
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
