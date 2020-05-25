<template>
  <v-data-table
    :headers="headers"
    :items="desserts"
    class="elevation-1"
    loading="true"
    loading-text='Data is loading, please wait...'
    dense
  >
      <template v-slot:headers.name="{ header }">
        {{ header.text.toUpperCase() }}
      </template>
    <template v-slot:top>
      <v-toolbar flat color="white">
        <v-toolbar-title>Orders</v-toolbar-title>
        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on }">
            <v-btn color="primary" dark class="mb-2" v-on="on">New Item</v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.order_id" label="Order Id"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.order_shipment_id" label="Order Shipment Id"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.shipping_name" label="Shipping Carrier"></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="save">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon
        small
        class="mr-2"
        @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        small
        @click="deleteItem(item)"
      >
        mdi-delete
      </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="initialize">Reset</v-btn>
    </template>
  </v-data-table>
</template>

<script>
 import axios from 'axios';
 
  export default {
    data: () => ({
      dialog: false,
      headers: [],
      desserts: [],
      editedIndex: -1,
      editedItem: {
        order_id: '',
        order_shipment_id: '',
        shipping_name: ''
      },
      defaultItem: {
        name: '',
        calories: 0,
        fat: 0,
        carbs: 0,
        protein: 0,
      },
    }),

    computed: {
      formTitle () {
        return 'New Item'
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
    },

    created () {
      this.initialize()
    },

    methods: {
      initialize () {
        const columnsHeader = [];
        let columnsData = [];
        let dataArray = [];

        setTimeout(() => {
          axios.get('./data.json')
          .then(resp => {
            dataArray = resp.data

            const createColumnsHeader = (data, mainField) => {  
              if (Array.isArray(data)) {
                createColumnsHeader(data[0], mainField)
              } else if (typeof data === 'object' && data !== null) {
                Object.keys(data).map((key) => {
                  const field = mainField ? mainField + '-' + key : key;

                  if (typeof data[key] !== 'object' || data[key] === null) {
                    columnsHeader.push({value: field, text: key, sortable: false})
                  } else {
                    createColumnsHeader(data[key], key)
                  }
                })
              } else {
                columnsHeader.push({field: data, label: data})
              }
              return columnsHeader;
            };

            const createColumnsData = (data, mainField, index) => {
              if (index >= 0) {
                columnsData = [...columnsData, {}]
              }
              if (Array.isArray(data)) {
                data.map((item, i) => {
                  if (i > 0) {
                    columnsData = [...columnsData, {}]
                  }
                  createColumnsData(item, mainField)
                })
              } else if (typeof data === 'object' && data !== null) {
                Object.keys(data).map(key => {
                  const field = mainField ? mainField + '-' + key : key;
                  const value = data[key] === null || data[key] === '' ? '-' : data[key];
                  const lastObject = columnsData.length - 1;

                  if (data[key] === null || typeof data[key] !== 'object') {
                    columnsData[lastObject][field] = value;
                  } else {
                    createColumnsData(data[key], key)
                  }
                })
              }
              return columnsData;
            };

            let columns = createColumnsHeader(dataArray[0]);
            columns.push({value: 'actions', text: 'Actions', sortable: false})

            this.headers = columns;
            this.desserts = dataArray.map((item, index) => createColumnsData(item, null, index))[dataArray.length - 1];
          })
          .catch(er => console.log('error', er))
        }, 1500);

        
      },

      editItem (item) {
        this.editedIndex = this.desserts.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem (item) {
        const index = this.desserts.indexOf(item)
        confirm('Are you sure you want to delete this item?') && this.desserts.splice(index, 1)
      },

      close () {
        this.dialog = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      save () {
        if (this.editedIndex > -1) {
          Object.assign(this.desserts[this.editedIndex], this.editedItem)
        } else {
          this.desserts.push(this.editedItem)
        }
        this.close()
      },
    },
  }
</script>

<style lang="scss">
  @import "./table.scss";
</style>
