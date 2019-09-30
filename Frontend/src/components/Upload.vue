<template>
<div class="myWrap">
  <div class="tags">

    <div class="card">
      <div class="card-header">Выбранный файл</div>
      <div class="card-main">
        <div class="main-description" v-if="selectedFile">{{this.selectedFile.name}}</div>
      </div>
    </div>

    <v-btn class="button" depressed color="red accent-2" @click="$refs.inputUpload.click()">Upload</v-btn>
    <input id="file" name="avatar" v-show="false" ref="inputUpload" type="file" @change="onFileSelected">

    <v-combobox v-model="tags" label="Your tags" chips clearable solo multiple>
      <template v-slot:selection="data">
        <v-chip :selected="data.selected" color="light-green" text-color="white" close @input="remove(data.item)">
          <strong>{{ data.item }}</strong>&nbsp;
        </v-chip>
      </template>
    </v-combobox>

    <v-btn class="button" depressed color="success" @click="submitFile">Submit</v-btn>
    
  </div>
</div>
</template>

<script>
import axios from 'axios'
import Vue from 'vue'
import VueNotification from "@kugatsu/vuenotification";

Vue.use(VueNotification, {
  position: 'bottomRight'
});

export default {
  data() {
    return {
      tags: [],
      selectedFile: null,
    }
  },

  methods: {
    remove(item) {
      this.tags.splice(this.tags.indexOf(item), 1)
      this.tags = [...this.tags]
    },

    onFileSelected() {
      this.selectedFile = event.target.files[0]
    },

    submitFile() {
      let self = this;
      
      let formData = new FormData();
      formData.append('file', this.selectedFile);
      formData.append('tags', this.tags);

      axios
        .post('http://localhost:2019/api/instances',
            formData, {
              headers: {
                'Content-Type': 'multipart/form-data'
              }
            },
          )
        .then(function () {
          self.selectedFile = null;
          self.tags = [];
          self.$notification.success("Success");
        })
        .catch(function (error) {
          // eslint-disable-next-line
          console.log(error.message);
          self.$notification.error("Can't send file!");
        });
    }
  }
}
</script>

<style scoped>
.card {
  width: auto;
  display: flex;
  flex-direction: column;
  border: 1px solid #EF9A9A;
  border-radius: 4px;
  overflow: hidden;
}

.card-header {
  color: #D32F2F;
  text-align: center;
  font-size: 12px;
  font-weight: 600;
  border-bottom: 1px solid #EF9A9A;
  background-color: #FFEBEE;
  padding: 5px 10px;
}

.card-main {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 15px 0;
}

.main-description {
  color: #D32F2F;
  font-size: 16px;
  text-align: center;
}

.wrap {
  width: 100%;
  height: 100%;
}

.tags {
  position: relative;
  width: 50%;
  margin-left: 25%;
  margin-top: 1%;
}

.button {
  width: 100%;
  margin-left: 0;
  margin-bottom: 3%;
  padding-left: 0;
  color: white;
}
</style>
