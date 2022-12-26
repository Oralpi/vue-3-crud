<template>
  <div class="submit-form">
    <div v-if="!submitted">
      <div class="form-group">
        <label for="title">Title</label>
        <input
          type="text"
          class="form-control"
          id="title"
          required
          v-model="tutorial.title"
          name="title"
        />
      </div>
      <div class="form-group">
        <label for="description">Description</label>
        <input
          class="form-control"
          id="description"
          required
          v-model="tutorial.description"
          name="description"
        />
      </div>
      <button @click="saveTutorial()" class="btn btn-success">Submit</button>
    </div>
    <div v-else>
      <h4>You submitted successfully!</h4>
      <button class="btn btn-success" @click="newTutorial()">Add</button>
    </div>
  </div>
</template>

<script setup>
import TutorialDataService from "@/services/TutorialDataService";
import { ref } from "vue";

const tutorial = ref({
  id: null,
  title: "",
  description: "",
  published: false,
});

let submitted = ref(false);

function saveTutorial() {
  let data = {
    title: tutorial.value.title,
    description: tutorial.value.description,
  };

  TutorialDataService.create(data)
    .then((response) => {
      tutorial.value.id = response.data.id;

      console.log(response.data);

      submitted.value = true;
    })
    .catch((e) => {
      console.log(e);
    });
}
function newTutorial() {
  submitted.value = false;
  tutorial.value = {};
}
</script>

<style>
.submit-form {
  max-width: 300px;
  margin: auto;
}
</style>
