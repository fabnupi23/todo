<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-sheet class="mx-auto" width="300">
          <v-form ref="form" lazy-validation class="text-center">
            <v-text-field
              v-model="newTodo.title"
              :counter="64"
              :rules="newTodo.titleRules"
              label="Title"
              required
            ></v-text-field>

            <v-select
              v-model="select"
              :items="items"
              :rules="[v => !!v || 'Item is required']"
              label="Item"
              required
            ></v-select>

            <v-textarea
              v-model="newTodo.description"
              autocomplete="Description"
              label="Description"
            ></v-textarea>

            <v-checkbox
              v-model="newTodo.is_complete"
              label="La tarea esta completa?"
            ></v-checkbox>

            <div class="d-flex flex-column">
              

              <v-btn
                class="mt-4"
                color="error"
                @click="reset"
              >
                Limpiar
              </v-btn>

              <v-btn
                class="mt-4"
                color="success"  
                :disabled="!newTodo.title"              
                @click="add"
              >
                Add
              </v-btn>

            </div>
          </v-form>
        </v-sheet>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" v-if="todoList.length">
        <h3>List:</h3>
        <v-list shaped>
          <v-list-item-group color="deep-purple" v-model="selected" multiple>
            <v-list-item v-for="item in todoList" :key="item.id" @click="updateStatus(item)">          
              <v-list-item-icon>
                {{ item.id }}
              </v-list-item-icon>  
              <v-list-item-content>
                <v-list-item-title>
                  <strong>{{ item.title }}</strong>
                </v-list-item-title>
                <v-list-item-subtitle>
                  {{ item.description }}
                </v-list-item-subtitle>
              </v-list-item-content>  

              <v-list-item-action>
                <v-checkbox
                  v-model="item.is_complete"
                  color="deep-purple"
                  @change="updateTodo(item)"
                ></v-checkbox>
              </v-list-item-action>  
            </v-list-item>
          </v-list-item-group>
        </v-list>        
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
export default {
  data: () => ({
    newTodo:{
      title: "",
      description: "",
      titleRules: [
        (v) => !!v || "Titulo requerido",
        (v) => (v && v.length <= 64) ||  "El título debe tener menos de 64 caracteres.",
      ],
      is_complete: false,
      user: 1,
    },    
    selected: [],
    todoList: [],
    url: "http://localhost:8000/api/todo/",
  }),

  mounted() {
    this.getTodos();
  },

  methods: {
    getTodos() {
      axios.get(`${this.url}?ordering=is_complete`).then((response) => {
        console.log(response);
        this.todoList = response.data;
        response.data.forEach((element, index) => {
          if (!element.is_complete) this.selected.push(index);
        })
      })        
    },

    reset(){
      this.$refs,form.reset();
    },
    add(){
      var data = this.newTodo;
      axios.post(this.url, data).then((response) => {
        console.log(response);
        this.getTodos();
      });
    },

    updateStatus(item){
      item.is_complete = !item.is_complete;

      const url = `${this.url}${item.id}/`;
      var data = {
        is_complete: item.is_complete,
      };
      axios.patch(url, data).then((response) => {
        console.log(response);
        this.getTodos();
      })
    }

  },
};
</script>
