<template>
<div class="myWrap">
  <div class="table">
    <v-toolbar color="white">
      <v-toolbar-title>My Employees</v-toolbar-title>
      <v-divider class="mx-2" inset vertical></v-divider>
      <v-spacer></v-spacer>
      <v-text-field v-model="search" append-icon="search" label="Search" single-line hide-details></v-text-field>
      <v-dialog v-model="dialog" max-width="500px">
        <template v-slot:activator="{ on }">
          <v-btn color="primary" dark class="mb-2" v-on="on">Add</v-btn>
        </template>
        <v-card>
          <v-card-title>
            <span class="headline">{{ formTitle }}</span>
          </v-card-title>

          <v-card-text>
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs6>
                  <v-text-field v-model="editedItem.surname" label="Фамилия"></v-text-field>
                </v-flex>
                <v-flex xs6>
                  <v-text-field v-model="editedItem.name" label="Имя"></v-text-field>
                </v-flex>
                <v-flex xs6>
                  <v-text-field v-model="editedItem.patronymic" label="Отчество"></v-text-field>
                </v-flex>
                <v-flex xs6>
                  <v-text-field v-model="editedItem.position" label="Должность"></v-text-field>
                </v-flex>
                <v-flex xs12>
                  <v-text-field v-model="editedItem.email" label="Почта"></v-text-field>
                </v-flex>
              </v-layout>
            </v-container>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" flat @click="close">Cancel</v-btn>
            <v-btn color="blue darken-1" flat @click="save">Save</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-toolbar>

    <v-data-table :headers="headers" :items="employees" :search="search" class="elevation-1" :rows-per-page-items="pagination">
      <template v-slot:items="props">
        <td>{{ props.item.surname }}</td>
        <td class="text-xs-left">{{ props.item.name }}</td>
        <td class="text-xs-left">{{ props.item.patronymic }}</td>
        <td class="text-xs-left">{{ props.item.position }}</td>
        <td class="text-xs-left">{{ props.item.email }}</td>
        <td class="justify-left layout px-0">
          <v-icon small class="mr-2" @click="editItem(props.item)">edit</v-icon>
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
        text: 'Фамилия',
        value: 'surname'
      },
      {
        text: 'Имя',
        value: 'name'
      },
      {
        text: 'Отчество',
        sortable: false,
        value: 'patronymic'
      },
      {
        text: 'Должность',
        value: 'position'
      },
      {
        text: 'Почта',
        sortable: false,
        value: 'email'
      },
      {
        text: 'Actions',
        value: 'surname',
        sortable: false
      }
    ],
    employees: [],
    editedIndex: -1,
    editedItem: {
      surname: '',
      name: '',
      patronymic: '',
      position: '',
      email: ''
    },
    defaultItem: {
      surname: '',
      name: '',
      patronymic: '',
      position: '',
      email: ''
    }
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
    }
  },

  watch: {
    dialog(val) {
      val || this.close()
    }
  },

  created() {
    let self = this;
    axios
      .get('http://localhost:2019/api/employees',)
      .then(function (res) {
        self.employees = res.data;
      })
      .catch(function (err) {
        // eslint-disable-next-line
        console.log(err.message);
      });
  },

  methods: {
    editItem(item) {
      this.editedIndex = this.employees.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem(item) {
      let self = this;
      const index = this.employees.indexOf(item)
      if(confirm('Вы точно хотите удалить?')){
        axios
          .delete('http://localhost:2019/api/employees/' + item._id,)
          .then(function () {
            self.employees.splice(index, 1)
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
        this.editedIndex = -1
      }, 300)
    },

    save() {
      let self = this;
      if (this.editedIndex > -1) {
        axios
          .put('http://localhost:2019/api/employees', self.editedItem)
          .then(function (res) {
            Object.assign(self.employees[self.editedIndex], res.data)
          })
          .catch(function (err) {
            // eslint-disable-next-line
            console.log(err.message);
          });
      } else {
        axios
          .post('http://localhost:2019/api/employees',self.editedItem)
          .then(function (res) {
            self.employees.push(res.data[0]);
          })
          .catch(function (err) {
            // eslint-disable-next-line
            console.log(err.message);
          });
      }
      this.close()
    }
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
