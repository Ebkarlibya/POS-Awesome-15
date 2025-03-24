<template>
  <div class="item-group-selector">
    <v-label v-if="label">{{ label }}</v-label>
    <div class="button-group">
      <v-btn
        v-for="groupName in itemGroups"
        :key="groupName"
        variant="outlined"
        size="small"
        :class="{ 'active-group': selectedGroup === groupName }"
        @click="handleGroupClick(groupName)"
        class="ms-2 mb-2"
      >
        {{ groupName }}
      </v-btn>
    </div>
  </div>
</template>

<script>
export default {
  name: "ItemGroupMultiSelect",
  props: {
    itemGroups: {
      type: Array,
      required: true
    },
    label: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      selectedGroup: null
    }
  },
  methods: {
    handleGroupClick(groupName) {
      this.selectedGroup = this.selectedGroup === groupName ? null : groupName
      this.$emit('group-selected', groupName)
    }
  }
}
</script>

<style scoped>
.item-group-selector {
  padding: 8px;
}

.button-group {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.active-group {
  background-color: #ffd700 !important;
  border-color: #ffa500 !important;
}
</style>