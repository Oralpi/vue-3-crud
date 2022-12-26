<template>
  <div class="list row">
    <div class="col-md-8">
      <div class="input-group mb-3">
        <input
          type="text"
          class="form-control"
          placeholder="Search by title"
          v-model="title"
        />
        <div class="input-group-append">
          <button
            class="btn btn-outline-secondary"
            type="button"
            @click="searchTitle()"
          >
            Search
          </button>
        </div>
      </div>
    </div>
    <div class="col-md-6">
      <h4>Tutorials List</h4>
      <ul class="list-group">
        <li
          class="list-group-item"
          :class="{ active: index === currentIndex }"
          v-for="(tutorial, index) in tutorials"
          :key="index"
          @click="setActiveTutorial(tutorial, index)"
        >
          {{ tutorial.title }}
        </li>
      </ul>
      <button class="m-3 btn btn-sm btn-danger" @click="removeAllTutorials()">
        Remove All
      </button>
    </div>
    <div class="col-mb-6">
      <div v-if="currentTutorial">
        <h4>Tutorial</h4>
        <div>
          <label>
            <strong>Title:</strong>
          </label>
          {{ currentTutorial.title }}
        </div>
        <div>
          <label>
            <strong>Description:</strong>
          </label>
          {{ currentTutorial.description }}
        </div>
        <div>
          <label>
            <strong>Status:</strong>
          </label>
          {{ currentTutorial.published ? "Published" : "Pending" }}
        </div>
        <router-link
          :to="'/tutorials/' + currentTutorial.id"
          class="badge badge-warning"
          >Edit</router-link
        >
      </div>
      <div v-else>
        <br />
        <p>Please click on a Tutorial...</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import TutorialDataService from "@/services/TutorialDataService";
import { onMounted, ref } from "vue";

let tutorials = ref([]);
let currentTutorial = ref(null);
let currentIndex = ref(-1);

const title = ref("");

function retrieveTutorials() {
  TutorialDataService.getAll()
    .then((response) => {
      tutorials.value = response.data;

      console.log(response.data);
    })
    .catch((e) => {
      console.log(e);
    });
}
function refreshList() {
  retrieveTutorials();

  currentTutorial.value = null;
  currentIndex.value = -1;
}
function setActiveTutorial(tutorial, index) {
  currentTutorial.value = tutorial;
  currentIndex.value = tutorial ? index : -1;
}
function removeAllTutorials() {
  TutorialDataService.deleteAll()
    .then((response) => {
      console.log(response.data);

      refreshList();
    })
    .catch((e) => {
      console.log(e);
    });
}
function searchTitle() {
  TutorialDataService.findByTitle(title)
    .then((response) => {
      tutorials.value = response.data;

      setActiveTutorial(null);
      console.log(response.data);
    })
    .catch((e) => {
      console.log(e);
    });
}

onMounted(() => {
  retrieveTutorials();
});
</script>

<style>
.list {
  text-align: left;
  max-width: 750px;
  margin: auto;
}
</style>

<!-- <template>
  <v-row align="center" class="list px-3 mx-auto">
    <v-col cols="12" sm="8">
      <v-text-field label="Search by Title" v-model="searchTitle" />
    </v-col>
    <v-col cols="12" sm="4">
      <v-btn
        small
        @click="
          page = 1;
          retrieveTutorials();
        "
        >Search</v-btn
      >
    </v-col>
    <v-col cols="12" sm="12">
      <v-row>
        <v-col cols="4" sm="3">
          <v-select
            label="Items per Page"
            v-model="pageSize"
            :items="pageSizes"
            @change="handlePageSizeChange()"
          />
        </v-col>
        <v-col cols="12" sm="9">
          <v-pagination
            total-visible="7"
            next-icon="mdi-menu-right"
            prev-icon="mdi-menu-left"
            v-model="page"
            :length="totalPages"
            @input="handlePageChange()"
          />
        </v-col>
      </v-row>
    </v-col>
    <v-col cols="12" sm="12">
      <v-card class="mx-auto" tile>
        <v-card-title>Tutorials</v-card-title>
        <v-data-table
          disable-pagination
          :headers="headers"
          :items="tutorials"
          :hide-default-footer="true"
        >
          <template v-slot:[`item.actions`]="{ item }">
            <v-icon small class="mr-2" @click="editTutorial(item.id)"
              >mdi-pencil</v-icon
            >
            <v-icon small @click="deleteTutorial(item.id)">mdi-delete</v-icon>
          </template>
        </v-data-table>
      </v-card>
    </v-col>
  </v-row>
</template>

<script setup>
import TutorialDataService from "@/services/TutorialDataService";
import { ref, onMounted } from "vue";

const tutorials = ref([]);
const searchTitle = ref("");
const headers = ref([
  {
    text: "Title",
    align: "start",
    sortable: false,
    value: "title",
  },
  {
    text: "Description",
    sortable: false,
    value: "description",
  },
  {
    text: "Status",
    sortable: false,
    value: "status",
  },
  {
    text: "Actions",
    sortable: false,
    value: "actions",
  },
]);
const page = ref(1);
const totalPages = ref(0);
const pageSize = ref(3);
const pageSizes = ref([3, 6, 9]);

function getRequestParams(searchTitle, page, pageSize) {
  let params = {};

  if (searchTitle) {
    params["title"] = searchTitle;
  }
  if (page) {
    params["page"] = page - 1;
  }
  if (pageSize) {
    params["size"] = pageSize;
  }

  return params;
}
function retrieveTutorials() {
  const params = getRequestParams(
    searchTitle.value,
    page.value,
    pageSize.value
  );

  TutorialDataService.getAll(params)
    .then((response) => {
      const { tutorials, totalPages } = response.data;
      console.log({ tutorials, totalPages });

      tutorials.value = tutorials.map(getDisplayTutorial());
      totalPages.value = totalPages;

      console.log(response.data);
    })
    .catch((e) => {
      console.log(e);
    });
}
function handlePageChange(value) {
  page.value = value;

  retrieveTutorials();
}
function handlePageSizeChange(size) {
  pageSize.value = size;
  page.value = 1;

  retrieveTutorials();
}
function getDisplayTutorial(tutorial) {
  return {
    id: tutorial.id,
    title: tutorial.title.substr(0, 20) + "...",
    description: tutorial.description.substr(0, 20) + "...",
    status: tutorial.published ? "Published" : "Pending",
  };
}

onMounted(() => {
  retrieveTutorials();
});
</script> -->
