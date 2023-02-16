<template>
  <v-container fill-height>
    <v-row>
      <v-col>
        <!-- <v-skeleton-loader v-if="!isFetched" type="list-item-avatar,list-item-avatar,list-item-avatar,list-item-avatar,list-item-avatar"></v-skeleton-loader> -->
        <v-skeleton-loader v-if="!isFetched" type="table"></v-skeleton-loader>
        <div v-else>
          <p class="text-h4 font-weight-light">User list</p>
          <v-data-table :headers="headers" :items="users" :search="search" sort-by="calories" class="elevation-1"
            :page.sync="page" :items-per-page="itemsPerPage" hide-default-footer :loading="!isFetched && isFetching">
            <template v-slot:[`item.avatar`]="{ item }">
              <v-avatar class="my-2">
                <v-img :src="item.img"></v-img>
              </v-avatar>
            </template>
            <template v-slot:top>
              <v-toolbar flat>
                <v-text-field v-model="search" filled dense rounded flat append-icon="mdi-magnify"
                  label="Search for users..." single-line hide-details class="rounded"
                  style="max-width: 400px"></v-text-field>
                <v-spacer></v-spacer>
                <v-dialog v-model="dialog" max-width="500px">
                  <template v-slot:activator="{ on, attrs }">
                    <v-btn v-if="$vuetify.breakpoint.xs" class="ml-6" color="primary" dark fab v-bind="attrs" v-on="on">
                      <div class="d-flex justify-space-between">
                        <v-icon small class="d-inline-flex mr-1">mdi-plus</v-icon>
                      </div>
                    </v-btn>
                    <v-btn v-else color="primary" dark class="text-capitalize rounded-pill px-4 ml-6 ml-sm-0" v-bind="attrs"
                      v-on="on">
                      <div class="d-flex justify-space-between">
                        <v-icon small class="d-inline-flex mr-1">mdi-plus</v-icon>
                        <span>Add user</span>
                      </div>
                    </v-btn>
                  </template>
                  <v-card>
                    <v-card-title>
                      <span class="text-h5">{{ formTitle }}</span>
                    </v-card-title>

                    <v-card-text>
                      <v-container>
                        <v-row>
                          <v-col cols="12" sm="6" md="4">
                            <v-text-field v-model="editedItem.name" label="Full name"></v-text-field>
                          </v-col>
                          <v-col cols="12" sm="6" md="4">
                            <v-text-field v-model="editedItem.img" label="Avatar link"></v-text-field>
                          </v-col>
                        </v-row>
                      </v-container>
                    </v-card-text>

                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn color="blue darken-1" text @click="close">
                        Cancel
                      </v-btn>
                      <v-btn color="blue darken-1" text @click="save">
                        Save
                      </v-btn>
                    </v-card-actions>
                  </v-card>
                </v-dialog>
                <v-dialog v-model="dialogDelete" max-width="500px">
                  <v-card>
                    <v-card-title class="text-h5">Are you sure you want to delete this item?</v-card-title>
                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
                      <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
                      <v-spacer></v-spacer>
                    </v-card-actions>
                  </v-card>
                </v-dialog>
              </v-toolbar>
            </template>
            <template v-slot:[`item.actions`]="{ item }">
              <v-icon small class="mr-2" @click="editItem(item)">
                mdi-square-edit-outline
              </v-icon>
              <v-icon small @click="deleteItem(item)">
                mdi-delete
              </v-icon>
            </template>
            <template v-slot:no-data>
              No data to display...
            </template>
          </v-data-table>
          <div class="text-center pt-2 float-left">
            <v-pagination v-model="page" :length="pageCount"></v-pagination>
          </div>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  data: () => ({
    page: 1,
    itemsPerPage: 8,
    isFetched: false,
    isFetching: false,
    search: '',
    dialog: false,
    dialogDelete: false,
    headers: [
      { text: '', value: 'avatar', sortable: false, width: '10%' },
      {
        text: 'Full Name',
        align: 'start',
        sortable: true,
        value: 'name',
      },
      { text: 'Actions', value: 'actions', sortable: false, width: '10%' },
    ],
    users: [],
    editedIndex: -1,
    editedItem: {
      name: '',
      img: ''
    },
    defaultItem: {
      name: '',
      img: ''
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'New user' : 'Edit user'
    },
    pageCount(){
      return this.$vuetify.breakpoint.xs ? 5 : 10
    }
  },

  watch: {
    dialog(val) {
      val || this.close()
    },
    dialogDelete(val) {
      val || this.closeDelete()
    },
    page(v) {
      console.log(v)
      this.fetchPage()
    },
    itemsPerPage(v) {
      console.log('itemsPerPage', v)
    },
    users(v) {
      console.log(v)
    }
  },

  created() {
    this.initialize()
  },

  methods: {
    initialize() {
      this.isFetching = true
      fetch(`https://reqres.in/api/users?page=${this.page}&per_page=${this.itemsPerPage}`)
        .then(res => {
          console.log(`https://reqres.in/api/users?page=${this.page}&per_page=${this.itemsPerPage}`)
          return res.json()
        })
        .then(res => {
          this.users = res.data.map(({ first_name, last_name, avatar }) => ({ name: `${first_name} ${last_name}`, img: avatar }))
          this.isFetched = true
          this.isFetching = false
        })
    },

    fetchPage() {
      this.isFetching = true
      fetch(`https://reqres.in/api/users?page=${this.page}&per_page=${this.itemsPerPage}`)
        .then(res => {
          console.log(`https://reqres.in/api/users?page=${this.page}&per_page=${this.itemsPerPage}`)
          return res.json()
        })
        .then(res => {
          this.users = res.data.map(({ first_name, last_name, avatar }) => ({ name: `${first_name} ${last_name}`, img: avatar }))
          this.isFetched = true
          this.isFetching = false
        })
    },


    editItem(item) {
      this.editedIndex = this.users.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem(item) {
      this.editedIndex = this.users.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogDelete = true
    },

    deleteItemConfirm() {
      this.users.splice(this.editedIndex, 1)
      this.closeDelete()
    },

    close() {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    closeDelete() {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.users[this.editedIndex], this.editedItem)
      } else {
        this.users.push(this.editedItem)
      }
      this.close()
    },
  },
}
</script>
<style>
.v-pagination__item {}
</style>