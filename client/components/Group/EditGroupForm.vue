<script setup lang="ts">
import { ref } from "vue";
import { fetchy } from "../../utils/fetchy";
import { formatDate } from "../../utils/formatDate";

const props = defineProps(["group"]);
const newName = ref(props.group.name);

const emit = defineEmits(["updateGroup", "refreshGroups"]);

const updateGroup = async (newName: string) => {
  try {
    const currentName = props.group.name;
    await fetchy(`api/groups`, "PATCH", { body: { name: currentName, changeTo: newName } });
  } catch (error) {
    return;
  }
  emit("updateGroup");
  emit("refreshGroups");
};
</script>

<template>
  <form @submit.prevent="updateGroup(newName)">
    <textarea id="newName" v-model="newName" placeholder="Edit group name" required></textarea>
    <div class="base">
      <menu>
        <li><button type="submit">Save</button></li>
        <li><button @click="emit('updateGroup')">Cancel</button></li>
      </menu>
      <p v-if="props.group.dateCreated !== props.group.dateUpdated" class="timestamp">Updated on: {{ formatDate(props.group.dateUpdated) }}</p>
      <p v-else class="timestamp">Created on: {{ formatDate(props.group.dateCreated) }}</p>
    </div>
  </form>
</template>
