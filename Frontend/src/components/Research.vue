<template>
  <div class="main">
    <div class="instance-block">
      <div class="tags">
        <div id="search">
          <scrolly class="foo" :style="{ width: '100%', height: '100%'}">
            <scrolly-viewport>
              <v-combobox v-model="tags" label="Tags" chips clearable solo multiple>
                <template v-slot:selection="data">
                  <v-chip :selected="data.selected" color="light-green" text-color="white" close @input="remove(data.item)">
                    <strong>{{ data.item }}</strong>&nbsp;
                  </v-chip>
                </template>
              </v-combobox>
            </scrolly-viewport>
            <scrolly-bar axis="y"></scrolly-bar>
          </scrolly>
        </div>
        <div id="button">
          <v-btn id="searchButton" depressed color="success" @click="search">Search</v-btn>
        </div>
      </div>

      <scrolly class="foo" :style="{ width: '100%', height: '80%', border: '1px solid white'}">
        <scrolly-viewport>
          <v-flex v-for="(item, index) in instancesIDS" :key="index" xs12 class="card">
            <v-card flat color="cyan darken-2" class="white--text">
              <v-layout id="card" row wrap class="instanceCard">
                <v-flex xs11 @click="showInfo(item.instanceID, index)">
                  <v-card-title primary-title>
                    <div>
                      <div class="headline">{{item.instanceID}}</div>
                      <v-chip disabled v-for="item in item.tags" :key="item.id" color="light-green lighten-5" text-color="black">{{ item }}</v-chip>
                    </div>
                  </v-card-title>
                  <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="white" depressed @click="editItem(item,index)">Edit</v-btn>
                  <v-btn color="white" depressed @click="deleteItem(item, index)">Delete</v-btn>
                </v-card-actions>
                </v-flex>
                <v-flex xs1>
                  <v-checkbox dark color="white" light v-model="selectedInstances" :value="instancesIDS[index].instanceID"></v-checkbox>
                </v-flex>
              </v-layout>
            </v-card>
          </v-flex>
        </scrolly-viewport>
        <scrolly-bar axis="y"></scrolly-bar>
      </scrolly>

      <v-dialog v-model="editDialog" max-width="500px">
        <v-card>
          <v-card-title>
            <span class="headline">Edit Tags</span>
          </v-card-title>

          <v-card-text>
            <v-container grid-list-md>
              <v-combobox v-model="selectedInstanceTags" label="Your tags" chips clearable solo multiple>
                <template v-slot:selection="data">
                  <v-chip :selected="data.selected" color="light-green" text-color="white" close @input="removeEditTag(data.item)">
                    <strong>{{ data.item }}</strong>&nbsp;
                  </v-chip>
                </template>
               </v-combobox>
            </v-container>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" flat @click="close">Cancel</v-btn>
            <v-btn color="blue darken-1" flat @click="save">Save</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </div>

    <div class="content-block">
      <div class="box">
        <v-flex sm12>
          <v-text-field :value="selectedPatient.name" label="Name" outline readonly></v-text-field>
        </v-flex>

        <v-flex sm12>
          <v-text-field :value="selectedPatient.id" label="ID" outline readonly></v-text-field>
        </v-flex>

        <v-flex sm12>
          <v-text-field :value="selectedPatient.sex" label="Sex" outline readonly></v-text-field>
        </v-flex>

        <v-flex sm12>
          <v-text-field :value="selectedPatient.instanceid" label="InstanceID" outline readonly></v-text-field>
        </v-flex>

        <v-flex sm12>
          <ul>
            <li v-for="(item, index) in selectedInstances" :key="index">{{ item }}</li>
          </ul>
        </v-flex>
      </div>
    </div>

    <div class="user-block">
      <scrolly class="foo" :style="{ width: '100%', height: '99%'}">
        <scrolly-viewport>
          <v-flex v-for="(item, index) in employees" :key="index" xs12 class="card">
            <v-card flat color="cyan darken-2" class="white--text">
              <v-layout>
                <v-flex xs12 class="userCard" id="card" @click="selectUser(index)">
                  <v-card-title primary-title>
                    <div>
                      <div class="headline">{{item.surname + ' ' + item.name}}</div>
                      <div>{{item.position}}</div>
                      <div>{{item.email}}</div>
                    </div>
                  </v-card-title>
                </v-flex>
              </v-layout>
            </v-card>
          </v-flex>
        </scrolly-viewport>
        <scrolly-bar axis="y"></scrolly-bar>
      </scrolly>
    </div>

    <div class="button-block">
      <v-dialog v-model="dialog" width="500">
        <template v-slot:activator="{ on }">
          <v-btn depressed color="success" block dark v-on="on">Explore</v-btn>
        </template>

        <v-card>
          <v-card-title class="headline grey lighten-2" primary-title>Подтверждение</v-card-title>
          <v-card-text>Вы уверены в выборе?</v-card-text>
          <v-divider></v-divider>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn v-if="loading" color="success" flat loading></v-btn>
            <v-btn v-else color="success" flat @click="sendMail()">Отправить</v-btn>
            <v-btn color="primary" flat @click="dialog = false">Закрыть</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { Scrolly, ScrollyViewport, ScrollyBar } from "vue-scrolly";
import Vue from "vue";
import VueNotification from "@kugatsu/vuenotification";

Vue.use(VueNotification, {
  position: "bottomRight"
});

export default {
  components: {
    Scrolly,
    ScrollyViewport,
    ScrollyBar
  },
  data() {
    return {
      editDialog: false,
      dialog: false,
      loading: false,
      selectedInstances: [],
      selectedInstanceTags: null,
      selectedIndex: null,
      instancesIDS: null,
      selectedPatient: {
        name: "",
        id: "",
        sex: "",
        instanceid: ""
      },
      employees: null,
      selectedUserIndex: 0,
      selectedInstanceIndex: 0,
      selectedUserEmail: null,
      selectedUser: null,
      tags: [],
      sampleID: null
    };
  },

  created() {
    let self = this;
    axios
      .get("http://localhost:2019/api/instances/")
      .then(function(res) {
        self.instancesIDS = res.data;
      })
      .catch(function(err) {
        // eslint-disable-next-line
        console.log(err.message);
      });

    axios
      .get("http://localhost:2019/api/employees/")
      .then(function(res) {
        self.employees = res.data;
      })
      .catch(function(err) {
        // eslint-disable-next-line
        console.log(err.message);
      });
  },

  methods: {
    search() {
      let self = this;
      this.selectedInstanceIndex = 0;
      this.selectedInstances = [];
      this.instancesIDS = [];

      axios
        .post("http://localhost:2019/api/instances/tags", self.tags)
        .then(function(res) {
          // eslint-disable-next-line
          self.instancesIDS = res.data;
        })
        .catch(function(err) {
          // eslint-disable-next-line
          console.log(err.message);
        });
    },

    remove(item) {
      this.tags.splice(this.tags.indexOf(item), 1);
      this.tags = [...this.tags];
    },

    selectUser(index) {
      document.getElementsByClassName("userCard")[this.selectedUserIndex].style.background = "";
      document.getElementsByClassName("userCard")[index].style.background = "#4ab14f";
      this.selectedUserIndex = index;
      this.selectedUserEmail = this.employees[index].email;
      this.selectedUser = this.employees[index];
    },

    showInfo(id, index) {
      let self = this;
      if (document.getElementsByClassName("instanceCard")) {
        document.getElementsByClassName("instanceCard")[this.selectedInstanceIndex].style.background = "";
        document.getElementsByClassName("instanceCard")[index].style.background = "#4ab14f";
        this.selectedInstanceIndex = index;
      }

      axios
        .get("http://localhost:2019/api/instances/" + id)
        .then(function(res) {
          self.selectedPatient = {
            name: res.data.PatientName,
            id: res.data.PatientID,
            sex: res.data.PatientSex,
            instanceid: id
          };
        })
        .catch(function(err) {
          // eslint-disable-next-line
          console.log(err.message);
        });
    },

    sendMail() {
      if (this.selectedInstances.length == 0 || this.selectedUserEmail == null) {
        alert("Не выбрана болезнь(-ни) или исследователь!");
        this.dialog = false;
        return;
      }
      
      let self = this;
      this.loading = true;
      let name = this.selectedUser.surname + " " + this.selectedUser.name + " " + this.selectedUser.patronymic;
      
      axios
        .post("http://localhost:2019/api/samples", {
          username: name,
          instances: this.selectedInstances,
          email: self.selectedUserEmail
        })
        .then(() => {
          self.$notification.success("Success");
          self.dialog = false;
          self.loading = false;
        })
        .catch(() => {
          self.$notification.error("Can't send e-mail!");
          self.dialog = false;
          self.loading = false;
        });
    },

    deleteItem(item) {
      let self = this;

      if(confirm('Вы точно хотите удалить?')) {
      axios
        .delete("http://localhost:2019/api/instances/" + item._id)
        .then(function() {
          self.instancesIDS.splice(self.instancesIDS.indexOf(item), 1)
          self.instancesIDS = [...self.instancesIDS]
          self.$notification.success("Deleted");
        })
        .catch(function(err) {
          // eslint-disable-next-line
          console.log(err.message);
        });
      }
    },

    editItem(item,index) {
      this.selectedInstanceTags = item.tags;
      this.selectedIndex = index;
      this.editDialog = true;
    },
    
    close() {
      this.editDialog = false
    },

    save() {
      let self = this;
      let tmp = this.instancesIDS[this.selectedIndex];
      tmp.tags = this.selectedInstanceTags;

      axios
      .put("http://localhost:2019/api/instances/", tmp)
      .then(function(){
        self.instancesIDS[self.selectedIndex].tags = self.selectedInstanceTags;
        self.$notification.success("Edited");
        self.editDialog = false;
      })
      .catch(function(){
        // eslint-disable-next-line
        console.log(err.message);
        self.$notification.error("Can't edit");
      });
    },

    removeEditTag(item) {
      this.selectedInstanceTags.splice(this.selectedInstanceTags.indexOf(item), 1);
      this.selectedInstanceTags = [...this.selectedInstanceTags];
    }
  }
};
</script>

<style scoped>
html,
body {
  margin: 0;
  padding: 0;
}
.main {
  margin: 0 auto;
  width: 100%;
  height: 100%;
}
.main .instance-block,
.content-block,
.user-block {
  display: inline-block;
  vertical-align: top;
}
.instance-block {
  /* background:red; */
  width: 30%;
  height: 92%;
}
.content-block {
  /* background:green; */
  width: 40%;
  height: 92%;
}
.user-block {
  /* background:blue; */
  width: 30%;
  height: 92%;
}
.button-block {
  /* background:pink; */
  width: 100%;
  height: 5%;
}
.card {
  width: 90%;
  margin: auto;
  margin-bottom: 2%;
  margin-top: 2%;
}
.card #card:hover {
  background: #4ab14f;
}
.tags {
  width: 100%;
  height: 20%;
  /* background: black; */
}
.tags #search {
  width: 100%;
  height: 70%;
  /* background: yellow; */
}
.tags #button {
  width: 100%;
  height: 30%;
}
#button #searchButton {
  width: 100%;
  height: 100%;
  margin: 0;
}
.box {
  margin: 2%;
}
.foo {
  border-radius: 10px;
}
</style>
