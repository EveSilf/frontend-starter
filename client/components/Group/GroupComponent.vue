<script setup lang="ts">
import { onBeforeMount, ref } from "vue";
import { fetchy } from "../../utils/fetchy";

const props = defineProps(["group"]);
const emit = defineEmits(["editGroup", "refreshGroups"]);

const username = ref("");
const isLoaded = ref(false);
let groupList = ref<Array<Record<string, string & { members: string[] }>>>([]);
let searchQuery = ref("");
const selectedUsers = ref<Array<string>>([]);
const dropdownVisible = ref(false);

const resetForm = () => {
  username.value = "";
};

onBeforeMount(async () => {
  await getGroups();
  isLoaded.value = true;
});

async function getGroups(name?: string) {
  let query: Record<string, string> = name !== undefined ? { name } : {};
  let Groups;
  try {
    Groups = await fetchy("api/groups", "GET", { query });
  } catch (_) {
    return;
  }
  searchQuery.value = name ? name : "";
  groupList.value = Groups;
}

const deleteGroup = async () => {
  try {
    await fetchy(`api/groups/${props.group.name}`, "DELETE");
  } catch (e) {
    return console.error("Failed to delete group:", e);
  }
  emit("refreshGroups");
};

async function addUserToGroup(group: string, user: string) {
  try {
    await fetchy(`api/groups/members/${group}`, "POST", {
      body: {
        member: user,
      },
    });
  } catch (e) {
    return console.error("Failed to add member:", e);
  }
  resetForm();
  emit("refreshGroups");
}

async function removeSelectedUsers(group: string) {
  const users = selectedUsers.value;
  if (users !== undefined) {
    for (const user of users) {
      try {
        if (user) {
          await fetchy(`api/groups/members/${group}/${user}`, "DELETE");
        }
      } catch (e) {
        return console.error("Failed to delete member:", e);
      }
    }
  }
  emit("refreshGroups");
}
</script>
<template>
  <div class="text">
    <div class="group">
      <div class="group-header">
        <div class="header-left">
          <h2 class="group-name">{{ props.group.name }}</h2>
        </div>
        <div class="dropdown-container">
          <div v-if="dropdownVisible" class="dropdown">
            <ul>
              <button class="edit-button btn-small pure-button" @click="emit('editGroup', props.group._id)">Edit Name</button>
              <button class="delete-btn btn-small pure-button" @click="deleteGroup">Delete Group</button>
            </ul>
          </div>
        </div>
      </div>

      <!-- Members and Add Members form -->
      <div class="members-add-form-container">
        <!-- Add Members form -->
        <div class="add-member-form">
          <label for="username"></label>
          <textarea class="add-member-input" id="username" v-model="username" placeholder=" username" required></textarea>
          <button type="submit" class="add-button btn-small pure-button" @click="addUserToGroup(group.name, username)">Add Member.</button>
        </div>

        <div class="members-container">
          <h3>members</h3>
          <ul class="member-list">
            <label v-for="member in group.members" :key="member">
              <template v-if="member !== 'DELETED_USER'"> <input type="checkbox" v-model="selectedUsers" :value="member" /> {{ member }} </template>
            </label>
          </ul>
        </div>
        <button class="delete-button btn-small pure-button" @click="removeSelectedUsers(group.name)" v-if="group.members.length > 0">Delete Members</button>
      </div>
    </div>
  </div>
</template>
