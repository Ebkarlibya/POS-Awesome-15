<template>
  <div class="item-group-selector">
    <v-label v-if="label" class="mb-2">{{ label }}</v-label>
    <div class="button-group">
      <v-btn
        v-for="groupName in itemGroups"
        :key="groupName"
        :style="{'background-color': selectedGroup === groupName ? 'orange' : '#1867c0'}"
        variant="contained"
        size="small"
        class="group-btn white--text transition-button"
        @click="handleGroupClick(groupName)"
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

.group-btn {
  min-width: 80px;
  text-align: center;
  font-size: 14px;
  font-weight: 800;
  border-radius: 8px;
}

/* تحريك التغييرات اللونية لإعطاء مظهر سلس */
.transition-button {
  transition: background-color 0.3s, color 0.3s, border-color 0.3s;
}

/* تأكيد بقاء النص أبيض دائمًا */
.white--text {
  color: #fff !important;
}
</style>