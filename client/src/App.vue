<template>
  <div>
    <h1>Search:</h1>
    <input type="search" v-model="searchTerm">
    <input type="button" value="GO" @click="search(searchTerm)">

    <div v-if="!isEditing">
      <div v-for="result in searchResults" :key="result.id" id="grid_parent">
        <div class="parent">
          <h1>{{ result.description.split(":")[0] }}</h1>
          <div class="flex">
            <input type="button" :value="'EDIT: ' + result.id.toString()" @click="editItem(result)">
            <img :src="'https://webapp.uibk.ac.at/ubi/cat/' + result.thumb" class="img">
            <p id="text">{{ result.description }}</p>
          </div>
        </div>
      </div>
    </div>

    <div v-else>
      <div class="edit-container">
        <h1>{{ editDes.split(":")[0] }}</h1>
        <div class="edit-content">
          <input type="button" value="SAVE" @click="saveChanges()">
          <img :src="editImg" class="edit-img">
          <textarea class="edit-text" rows="20" cols="200" v-model="editDes"></textarea>
        </div>
        <div class="history-container">
          <table>
            <tr v-for="(h, index) in editHistory" :key="h.id">
              <td>{{ h.datum }}</td>
              <td @click="setCurrentText(index)">{{ h.description }}</td>
            </tr>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

const api_url = "http://127.0.0.1:5000/";

async function api_search(query){
  const response = await axios.get( api_url + 'cat-search/' + query ,{});
  return response.data;
}

async function api_id_search(query){
  const response = await axios.get( api_url + 'cat-item/' + query , {});
  return response.data;
}
async function api_id_patch(query, description){
  const response = await axios.patch( api_url + 'cat-item-history/' + query , {'params':{'description':description}});
  return response.data;
}
async function api_id_history(q){
  try {
    const response = await axios.get(api_url + 'cat-item-history/' + q, {});
    return response.data;
  } catch (error) {
    console.error('Error fetching history:', error);
    return null;
  }
}



export default {
  name: 'App',
  data(){
    return{
      searchResults: Object,
      searchTerm: "",
      isEditing: false,
      editId: String,
      editDes: String,
      editImg: String,
      editHistory: Object
    }
  },
  methods:{
    async search(query){
      this.isEditing = false
      this.searchResults = await api_search(query)
    },
    editItem(result){
      this.editId = result.id.toString()
      this.editImg = 'https://webapp.uibk.ac.at/ubi/cat/' + result.thumb
      this.editDes = result.description
      this.isEditing = true
      this.loadEditHistory()
    },
    async saveChanges() {
      try {
        await api_id_patch(this.editId, this.editDes);
        this.loadEditHistory();
        this.finishEditing();
      } catch (error) {
        console.error("Error saving changes:", error);
      }
    },

    async loadEditHistory() {
      this.editHistory = await api_id_history(this.editId);
    },
    setCurrentText(index) {
      document.getElementById("text").value = this.editHistory[index].description;
    },
    finishEditing() {
      this.isEditing = false;
    }
  }
};
</script>

<style>
.result-container {
  padding: 1em; 
  border: 1px solid #ccc;
  margin-bottom: 1em;
}s

.result-content {
  display: flex;
  align-items: center;
}

.result-img {
  width: 100px;
  height: auto;
  margin: 0 1em;
}

.result-text {
  flex-grow: 1;
}

.edit-container {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 1em;
}

.edit-content {
  display: flex;
  align-items: center;
}

.edit-img {
  width: 100px;
  height: auto;
  margin: 0 1em;
}

.edit-text {
  flex-grow: 1;
}

.history-container {
  overflow-y: auto;
  max-height: 400px;
  border: 1px solid #ccc;
  padding: 1em;
}

</style>