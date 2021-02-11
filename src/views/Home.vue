<template>
  <div class="home">
    <ul>
      <li v-for="error in errors" :key="error">{{ error }}</li>
    </ul>
    <button v-on:click="showAdd = !showAdd">New Place</button>
    <br v-if="showAdd" />
    <input type="text" v-if="showAdd" v-model="name" placeholder="Name" />
    <input type="text" v-if="showAdd" v-model="address" placeholder="Address" />
    <button v-if="showAdd" v-on:click="addPlace">Add</button>

    <div v-for="place in places" :key="place.id">
      <h2>{{ place.name }}</h2>
      <p>{{ place.address }}</p>
      <button v-on:click="showInfo(place)">More Info</button>
    </div>

    <dialog>
      <form method="dialog">
        <h2>Address Info</h2>
        <p>
          Name:
          <input type="text" v-model="currentPlace.name" />
        </p>
        <p>
          Address:
          <input type="text" v-model="currentPlace.address" />
        </p>
        <button v-on:click="updatePlace(currentPlace)">Update Location</button>
        <button v-on:click="destroyPlace(currentPlace)">Delete Location</button>
        <button>Close</button>
      </form>
    </dialog>
  </div>
</template>
<style></style>
<script>
import axios from "axios";
export default {
  data: function() {
    return {
      places: [],
      currentPlace: {},
      showAdd: false,
      showUpdate: false,
      name: "",
      address: "",
      errors: [],
    };
  },
  created: function() {
    this.placesIndex();
  },
  methods: {
    showInfo: function(place) {
      console.log(place);
      this.currentPlace = place;
      document.querySelector("dialog").showModal();
    },
    placesIndex: function() {
      axios
        .get("/api/places")
        .then(response => {
          console.log("Success", response.data);
          this.places = response.data;
        })
        .catch(error => {
          error.response.data.errors;
          this.errors = error.response.data.errors;
        });
    },
    addPlace: function() {
      var params = {
        name: this.name,
        address: this.address,
      };
      axios
        .post("/api/places", params)
        .then(response => {
          console.log("Success", response.data);
          this.places.push(response.data);
          this.name = "";
          this.address = "";
        })
        .catch(error => {
          error.response.data.errors;
          this.errors = error.response.data.errors;
        });
    },
    updatePlace: function(place) {
      var params = {
        name: place.name,
        address: place.address,
      };
      axios
        .patch(`/api/places/${place.id}`, params)
        .then(response => {
          console.log("Success", response.data);
          this.name = "";
          this.address = "";
        })
        .catch(error => {
          console.log(error.response.data.errors);
          this.errors = error.response.data.errors;
        });
    },
    destroyPlace: function(place) {
      var confirmation = confirm(`Do you want to delete ${place.name}?`);
      if (confirmation) {
        axios.delete(`/api/places/${place.id}`).then(response => {
          console.log("Success", response.data);
          var index = this.places.indexOf(place);
          this.places.splice(index, 1);
        });
      } else {
        alert(`${place.name} not deleted`);
      }
    },
  },
};
</script>
