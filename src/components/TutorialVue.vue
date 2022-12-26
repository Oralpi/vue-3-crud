<template>
  <div v-if="currentTutorial" class="edit-form">
    <h4>Tutorial</h4>
    <form>
      <div class="form-group">
        <label for="title">Title</label>
        <input
          type="text"
          class="form-control"
          id="title"
          v-model="currentTutorial.title"
        />
      </div>
      <div class="form-group">
        <label for="description">Description</label>
        <input
          type="text"
          class="form-control"
          id="description"
          v-model="currentTutorial.description"
        />
      </div>
      <div class="form-group">
        <label>
          <strong>Status:</strong>
        </label>
        {{ currentTutorial.published ? "Published" : "Pending" }}
      </div>
    </form>
    <button
      class="badge badge-primary mr-2"
      v-if="currentTutorial.published"
      @click="updatePublished(false)"
    >
      UnPublish
    </button>
    <button
      v-else
      class="badge badge-primary mr-2"
      @click="updatePublished(true)"
    >
      Publish
    </button>
    <button class="badge badge-danger mr-2" @click="deleteTutorial()">
      Delete
    </button>
    <button type="submit" class="badge badge-success" @click="updateTutorial()">
      Update
    </button>
    <p>{{ message }}</p>
  </div>
  <div v-else>
    <br />
    <p>Please click on a Tutorial...</p>
  </div>
</template>

<script setup>
import TutorialDataService from "@/services/TutorialDataService";
import { ref, onMounted } from "vue";
import { useRouter, useRoute } from "vue-router";

const router = useRouter();
const route = useRoute();
const currentTutorial = ref(null);
const message = ref("");

function getTutorial(id) {
  TutorialDataService.get(id)
    .then((response) => {
      currentTutorial.value = response.data;

      console.log(response.data);
    })
    .catch((e) => {
      console.log(e);
    });
}
function updatePublished(status) {
  let data = {
    id: currentTutorial.value.id,
    title: currentTutorial.value.title,
    description: currentTutorial.value.description,
    published: status,
  };

  TutorialDataService.update(currentTutorial.value.id, data)
    .then((response) => {
      console.log(response.data);

      currentTutorial.value.published = status;
      message.value = "The status was updated successfully!";
    })
    .catch((e) => {
      console.log(e);
    });
}
function updateTutorial() {
  TutorialDataService.update(currentTutorial.value.id, currentTutorial.value)
    .then((response) => {
      console.log(response.data);

      message.value = "The tutorial was updated successfully!";
    })
    .catch((e) => {
      console.log(e);
    });
}
function deleteTutorial() {
  TutorialDataService.delete(currentTutorial.value.id).then((response) => {
    console.log(response.data);

    router.push({
      name: "tutorials",
    });
  });
}

onMounted(() => {
  message.value = "";

  getTutorial(route.params.id);
});
</script>

<style></style>
