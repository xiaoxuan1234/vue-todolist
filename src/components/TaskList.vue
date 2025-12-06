<script setup>
import { ref } from "vue";
import TaskItem from "./TaskItem.vue";

const props = defineProps({
  title: { type: String, required: true },
  tasks: { type: Array, required: true },
  searchResults: { type: Array, default: () => [] },
  filterKey: { type: String, default: "all" },
  sortKey: { type: String, default: "default" },
});

const emit = defineEmits([
  "addTask",
  "deleteTask",
  "update:filterKey",
  "update:sortKey",
  "toggleSidebar",
]);

const newTask = ref("");

const addTask = () => {
  const text = newTask.value?.trim();
  if (!text) return;
  emit("addTask", text);
  newTask.value = "";
};
</script>

<template>
  <div class="center">
    <div class="center-header">
      <svg
        @click="emit('toggleSidebar')"
        class="more-icon"
        viewBox="0 0 1024 1024"
        width="20"
        height="20"
      >
        <path
          d="M66.5 211.8h891c28.2 0 51-22.2 51-49.6 0-27.4-22.8-49.7-51-49.7H66.5c-28.1 0-51 22.2-51 49.6s22.8 49.7 51 49.7zm891 248.2H66.5c-28.2 0-51 22.2-51 49.7s22.8 49.6 51 49.6h891c28.2 0 51-22.2 51-49.6 0-27.4-22.9-49.7-51-49.7zm0 351.6H66.5c-28.1 0-51 22.2-51 49.7s22.8 49.6 51 49.6h891c28.2 0 51-22.2 51-49.6 0-27.4-22.8-49.7-51-49.7z"
        />
      </svg>
      <div class="title">{{ searchResults.length ? "搜索结果" : title }}</div>
    </div>
    <div class="center-content">
      <div class="addTask">
        <svg class="icon" fill="currentColor" width="20" height="20" viewBox="0 0 20 20">
          <path
            d="M10 2.5a.5.5 0 00-1 0V9H2.5a.5.5 0 000 1H9v6.5a.5.5 0 001 0V10h6.5a.5.5 0 000-1H10V2.5z"
          />
        </svg>
        <input
          type="text"
          placeholder="添加任务"
          v-model="newTask"
          @keydown.enter="addTask"
        />
      </div>
      <div class="toolbar">
        <div class="filter-group">
          <div
            class="filter-btn"
            :class="{ active: filterKey === 'all' }"
            @click="emit('update:filterKey', 'all')"
          >
            全部
          </div>
          <div
            class="filter-btn"
            :class="{ active: filterKey === 'undone' }"
            @click="emit('update:filterKey', 'undone')"
          >
            未完成
          </div>
          <div
            class="filter-btn"
            :class="{ active: filterKey === 'done' }"
            @click="emit('update:filterKey', 'done')"
          >
            已完成
          </div>
        </div>
        <div class="sort-group">
          <span class="sort-label">排序:</span>
          <select
            class="sort-select"
            :value="sortKey"
            @change="emit('update:sortKey', $event.target.value)"
          >
            <option value="default">默认</option>
            <option value="priority">优先级</option>
            <option value="date">截止日期</option>
          </select>
        </div>
      </div>

      <!-- 搜索结果 -->
      <div class="search-results" v-if="searchResults.length">
        <div class="task-item" v-for="t in searchResults" :key="t.id">
          <input type="checkbox" v-model="t.done" />
          <div class="search1">
            <span :class="{ done: t.done }">{{ t.text }}</span>
            <span class="list-tag">{{ t.listTitle }}</span>
          </div>
          <span class="delect" @click="emit('deleteTask', t)">
            <svg class="icon" viewBox="0 0 1024 1024" width="20" height="20">
              <path
                d="M254.4 804.7l-.03-4.79zM614.2 259c-22.1 0-40 17.9-40 40l.37 502.2c0 22.1 17.9 40 40 40s40-18 40-40l-.35-502.2c0-22.1-17.9-40-40-40zm279 -118.9l-318.9.15-.18-41.4c0-22.1-17.9-40-40-40-7.3 0-14.1 2-20 5.4-5.9-3.4-12.7-5.4-20-5.4-22.1 0-40 17.9-40 40l.19 41.5-230.1.1c-3.2-.8-6.6-1.3-10.1-1.3s-6.9.5-10.1 1.3l-73.1.03c-22.1 0-40 18-40 40s17.9 40 40 40l43.2-.02.29 617.9.06 0v42.6c0 44.2 35.9 80 80 80h40v.3l440.5-.26v-.05h40c43.7 0 79.2-35 80-78.4l-.33-583.4c0-22.1-17.9-40-40-40s-40 17.9-40 40l.29 509.3c-1.4 60.3-18.6 71.4-80 71.4l41.6 1-454.4.27 52.4-1.3c-62.7 0-79.3-11.5-80.1-75.3l.5 76.6h-.54l-.31-660.8 236.8-.11c1.2.1 2.4.17 3.6.17 1.2 0 2.4-.06 3.6-.17l32.6-.02c1.3.12 2.5.19 3.8.19 1.3 0 2.6-.07 3.9-.19l354.9-.16c22.1 0 40-17.9 40-40s-17.9-40-40-40zM774.9 815.3l.04 65.7h-.46v-65.7zM414 259c-22.1 0-40 17.9-40 40l.37 502.2c0 22.1 17.9 40 40 40s40-18 40-40l-.37-502.2c0-22.1-17.9-40-40-40z"
                fill="#272636"
              />
            </svg>
          </span>
        </div>
      </div>

      <!-- 任务列表 -->
      <div class="task" v-else>
        <TaskItem
          v-for="task in tasks"
          :key="task.id"
          :task="task"
          @delete="emit('deleteTask', $event)"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.center {
  flex: 1 1 auto;
  display: flex;
  flex-direction: column;
}

.center-header {
  display: flex;
  gap: 10px;
  width: 100%;
  padding: 20px;
  padding-top: 25px;
  font-size: 20px;
  align-items: center;
}

.title {
  max-width: 80%;
  display: inline;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.more-icon {
  flex: 0 0 auto;
  cursor: pointer;
}

.addTask {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 98%;
  padding: 0 15px;
  margin: 0 auto;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.08);
  color: rgb(100, 100, 241);
  border-radius: 12px;
}

.addTask input {
  width: 100%;
  height: 8px;
  padding: 30px 0;
  border: none;
  outline: none;
  background-color: #f8f8f8;
  font-size: 16px;
}

.addTask input::placeholder {
  color: rgb(100, 100, 241);
}

.addTask input:focus::placeholder {
  color: #b6b4b4;
}

.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  flex-wrap: wrap;
  gap: 10px;
}

.filter-group {
  display: flex;
  gap: 8px;
}

.filter-btn {
  padding: 8px 12px;
  cursor: pointer;
  border-radius: 6px;
  font-size: 14px;
  transition: all 0.2s;
}

.filter-btn:hover {
  background-color: #73a8f850;
}

.filter-btn.active {
  background-color: #73a8f850;
  color: rgb(100, 100, 241);
  font-weight: 500;
}

.sort-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.sort-label {
  font-size: 14px;
  color: #666;
}

.sort-select {
  padding: 6px 10px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
  outline: none;
  cursor: pointer;
  background: #fff;
}

.sort-select:focus {
  border-color: rgb(100, 100, 241);
}

.center-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  margin-bottom: 30px;
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.center-content::-webkit-scrollbar {
  display: none;
}

.task {
  flex: 1;
  margin: 0 10px;
  overflow-y: auto;
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.task::-webkit-scrollbar {
  display: none;
}

.search-results {
  flex: 1;
  margin: 0 10px;
  overflow-y: auto;
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.search-results::-webkit-scrollbar {
  display: none;
}

.task-item {
  align-items: center;
  justify-content: center;
  padding: 20px;
  display: flex;
  gap: 10px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.267);
}

.done {
  text-decoration: line-through;
  color: #999;
}

.list-tag {
  font-size: 12px;
  color: #999;
}

.search1 {
  display: flex;
  flex-direction: column;
}

.delect {
  margin-left: auto;
  cursor: pointer;
}
</style>
