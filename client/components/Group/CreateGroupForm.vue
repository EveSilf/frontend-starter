<script setup lang="ts">
import { ref } from "vue";
import { fetchy } from "../../utils/fetchy";

const name = ref("");

const emit = defineEmits(["refreshGroups"]);

const createGroup = async (name: string) => {
  try {
    await fetchy("api/groups", "POST", {
      body: {
        name: name,
      },
    });
  } catch (_) {
    return;
  }
  emit("refreshGroups");
  emptyForm();
};

const emptyForm = () => {
  name.value = "";
};
</script>

<template>
  <div class="text">
    <h2>create a group</h2>
    <form @submit.prevent="createGroup(name)">
      <label class="group-name" for="name">group name:</label>
      <textarea id="name" v-model="name" class="input"></textarea>
      <button type="submit" class="button pure-button-primary pure-button">Create Group.</button>
    </form>
  </div>
</template>
