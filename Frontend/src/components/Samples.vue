<template>
<div class="myWrap">
  <div class="table">
    <v-toolbar color="white">
      <v-toolbar-title>Samples</v-toolbar-title>
      <v-divider class="mx-2" inset vertical></v-divider>
      <v-spacer></v-spacer>
      <v-text-field v-model="search" append-icon="search" label="Search" single-line hide-details></v-text-field>
      <v-dialog v-model="dialog" max-width="500px">
        <v-card>
          <v-card-title>
            <span class="headline">More information</span>
          </v-card-title>

          <v-card-text>
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex v-for="value in selectedSample" :key="value.id" xs12>
                    {{value}}
                </v-flex>
              </v-layout>
            </v-container>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" flat @click="close">Close</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-toolbar>

    <v-data-table :headers="headers" :items="samples" :search="search" class="elevation-1" :rows-per-page-items="pagination">
      <template v-slot:items="props">
        <td>{{ props.item.username }}</td>
        <td class="text-xs-left">{{ props.item.email }}</td>
        <td class="text-xs-left">{{ props.item.time }}</td>
        <td class="text-xs-left">{{ props.item._id }}</td>
        <td class="justify-left layout px-0">
          <v-icon small class="mr-2" @click="editItem(props.item)">info</v-icon>
          <v-icon small @click="deleteItem(props.item)">delete</v-icon>
        </td>
      </template>
    </v-data-table>
  </div>
</div>
</template>

<script>
import axios from 'axios'

export default {
  data: () => ({
    search: '',
    dialog: false,
    pagination: [10,25,{"text":"$vuetify.dataIterator.rowsPerPageAll","value":-1}],
    headers: [{
        text: 'ФИО',
        value: 'username'
      },
      {
        text: 'Почта',
        value: 'email'
      },
      {
        text: 'Дата/Время',
        value: 'time'
      },
      {
        text: 'ID',
        value: '_id'
      },
      {
        text: 'Actions',
        value: 'actions',
        sortable: false
      }
    ],
    samples: [],
    selectedSample: [],
  }),

  watch: {
    dialog(val) {
      val || this.close()
    }
  },

  created() {
    let self = this;
    axios
      .get('http://localhost:2019/api/samples',)
      .then(function (res) {
        self.samples = res.data;
        console.log(res.data);
      })
      .catch(function (err) {
        // eslint-disable-next-line
        console.log(err.message);
      });
  },

  methods: {
    editItem(item) {
      this.selectedSample = Object.assign({}, item.instances);
      this.dialog = true;
    },

    deleteItem(item) {
      let self = this;
      const index = this.samples.indexOf(item)
      if(confirm('Вы точно хотите удалить?')){
        axios
          .delete('http://localhost:2019/api/samples/' + item._id,)
          .then(function () {
            self.samples.splice(index, 1)
          })
          .catch(function (err) {
            // eslint-disable-next-line
            console.log(err.message);
          });
      }
    },

    close() {
      this.dialog = false
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
      }, 300)
    },
  }
}
</script>

<style>
  .table {
    margin-top: 1%;
    margin-left: 10%;
    width: 80%;
  }
</style>
