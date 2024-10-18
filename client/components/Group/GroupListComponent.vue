<script setup lang="ts">
import CreateGroupForm from "@/components/Group/CreateGroupForm.vue";
import EditGroupForm from "@/components/Group/EditGroupForm.vue";
import GroupComponent from "@/components/Group/GroupComponent.vue";
import { useUserStore } from "@/stores/user";
import { fetchy } from "@/utils/fetchy";
import { storeToRefs } from "pinia";
import { onBeforeMount, ref } from "vue";

const { isLoggedIn } = storeToRefs(useUserStore());

const loaded = ref(false);
let groups = ref<Array<Record<string, string>>>([]);
let editing = ref("");
let searchOwner = ref("");

async function getGroups(owner?: string) {
  let query: Record<string, string> = owner !== undefined ? { owner } : {};
  let groupResults;
  try {
    groupResults = await fetchy("/api/groups", "GET", { query });
  } catch (_) {
    return;
  }
  searchOwner.value = owner ? owner : "";
  groups.value = groupResults;
}

function updateEditing(id: string) {
  editing.value = id;
}

onBeforeMount(async () => {
  await getGroups();
  loaded.value = true;
});
</script>

<template>
  <section v-if="isLoggedIn">
    <h2>Create a group:</h2>
    <CreateGroupForm @refreshGroups="getGroups" />
  </section>
  <div class="row">
    <h2 v-if="!searchOwner">Groups:</h2>
    <h2 v-else>Groups by {{ searchOwner }}:</h2>
    <SearchGroupForm @getGroupsByOwner="getGroups" />
  </div>
  <section class="groups" v-if="loaded && groups.length !== 0">
    <article v-for="group in groups" :key="group._id">
      <GroupComponent v-if="editing !== group._id" :group="group" @refreshGroups="getGroups" @editGroup="updateEditing" />
      <EditGroupForm v-else :group="group" @refreshGroups="getGroups" @editGroup="updateEditing" />
    </article>
  </section>
  <p v-else-if="loaded">No groups found</p>
  <p v-else>Loading...</p>
</template>
