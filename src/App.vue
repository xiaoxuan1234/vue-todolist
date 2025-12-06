<script setup>
import { onMounted, ref, computed, watch } from "vue";
import Sidebar from "./components/Sidebar.vue";
import TaskList from "./components/TaskList.vue";

// 常量
const KEY = "todoData";
const LAST_VISIT_KEY = "lastVisitDate";
const PRESET_NAMES = {
  today: "今天",
  import: "重要",
};

// UI 状态
const showLeft = ref(true);
const selectedListId = ref("today");
const filterKey = ref("all");
const keyword = ref("");
const sortKey = ref("default"); // default, priority, date

// 数据
const todoData = ref([
  { id: "today", title: "今天", tasks: [] },
  { id: "import", title: "重要", tasks: [] },
  { id: "tasks", title: "任务", tasks: [] },
]);

// 生命周期
onMounted(() => {
  const local = loadTodos();
  if (local && Array.isArray(local)) {
    todoData.value = local;
  }
  resetTodayIfNeeded();
});

watch(todoData, () => saveTodos(todoData.value), { deep: true });

// 存储函数
function loadTodos() {
  try {
    const raw = localStorage.getItem(KEY);
    return raw ? JSON.parse(raw) : null;
  } catch {
    return null;
  }
}

function saveTodos(data) {
  localStorage.setItem(KEY, JSON.stringify(data));
}

// 计算属性
const currentList = computed(() =>
  todoData.value.find((l) => l.id === selectedListId.value)
);

const currentListName = computed(() =>
  PRESET_NAMES[selectedListId.value] || currentList.value?.title || "未选择列表"
);

const currentListTask = computed(() => {
  if (selectedListId.value === "today") {
    const todayStr = new Date().toISOString().slice(0, 10);
    const tasks = [];
    todoData.value.forEach((list) => {
      list.tasks.forEach((task) => {
        if (task.dueDate === todayStr) tasks.push(task);
      });
    });
    return tasks;
  }
  if (selectedListId.value === "import") {
    const tasks = [];
    todoData.value.forEach((list) => {
      list.tasks.forEach((task) => {
        if (task.isImportant) tasks.push(task);
      });
    });
    return tasks;
  }
  return currentList.value?.tasks || [];
});

const filteredTasks = computed(() => {
  let base = [...currentListTask.value];
  
  // 筛选
  if (filterKey.value === "undone") base = base.filter((t) => !t.done);
  if (filterKey.value === "done") base = base.filter((t) => t.done);
  
  // 排序
  if (sortKey.value === "priority") {
    const priorityOrder = { high: 0, medium: 1, low: 2 };
    base.sort((a, b) => {
      const pa = a.priority ? priorityOrder[a.priority] : 3;
      const pb = b.priority ? priorityOrder[b.priority] : 3;
      return pa - pb;
    });
  } else if (sortKey.value === "date") {
    base.sort((a, b) => {
      if (!a.dueDate && !b.dueDate) return 0;
      if (!a.dueDate) return 1;
      if (!b.dueDate) return -1;
      return new Date(a.dueDate) - new Date(b.dueDate);
    });
  }
  
  return base;
});

const customLists = computed(() =>
  todoData.value.filter((l) => !["today", "import", "tasks"].includes(l.id))
);

const searchResults = computed(() => {
  if (!keyword.value.trim()) return [];
  const kw = keyword.value.trim().toLowerCase();
  const results = [];
  todoData.value.forEach((list) => {
    list.tasks.forEach((task) => {
      if (task.text.toLowerCase().includes(kw)) {
        results.push({ ...task, listTitle: list.title });
      }
    });
  });
  return results;
});

// 辅助函数
const getCount = (id) => {
  if (id === "today") {
    const todayStr = new Date().toISOString().slice(0, 10);
    let count = 0;
    todoData.value.forEach((list) => {
      list.tasks.forEach((task) => {
        if (task.dueDate === todayStr && !task.done) count++;
      });
    });
    return count || "";
  }
  if (id === "import") {
    let count = 0;
    todoData.value.forEach((list) => {
      list.tasks.forEach((task) => {
        if (task.isImportant && !task.done) count++;
      });
    });
    return count || "";
  }
  const list = todoData.value.find((item) => item.id === id);
  return list?.tasks.filter((t) => !t.done).length || "";
};

// 列表操作
const selectList = (listId) => {
  selectedListId.value = listId;
};

const addList = (title) => {
  todoData.value.push({
    id: `list_${Date.now()}`,
    title,
    tasks: [],
  });
};

const deleteList = (id) => {
  if (!confirm("确定删除该列表吗？")) return;
  todoData.value = todoData.value.filter((l) => l.id !== id);
  if (selectedListId.value === id) {
    selectedListId.value = "today";
  }
};

const renameList = (list, newTitle) => {
  list.title = newTitle;
};

// 任务操作
const addTask = (text) => {
  const list = currentList.value;
  if (!list) return;

  const task = {
    id: `task_${Date.now()}`,
    text,
    done: false,
    isImportant: selectedListId.value === "import",
    dueDate:
      selectedListId.value === "today"
        ? new Date().toISOString().slice(0, 10)
        : null,
    listId: list.id,
    priority: null,
    tag: null,
  };

  list.tasks.push(task);
};

const deleteTask = (task) => {
  const list = todoData.value.find((l) => l.id === task.listId);
  if (!list) return;
  const idx = list.tasks.findIndex((t) => t.id === task.id);
  if (idx > -1) list.tasks.splice(idx, 1);
};

const toggleSidebar = () => {
  showLeft.value = !showLeft.value;
};

// 今日列表重置逻辑
function resetTodayIfNeeded() {
  const lastVisit = localStorage.getItem(LAST_VISIT_KEY);
  const today = new Date().toDateString();

  if (!lastVisit || new Date(lastVisit).toDateString() !== today) {
    const todayList = todoData.value.find((l) => l.id === "today");
    if (todayList) {
      todayList.tasks = [];
    }
    localStorage.setItem(LAST_VISIT_KEY, new Date().toISOString());
  }
}
</script>

<template>
  <div class="todolist">
    <div class="Left-open" v-show="showLeft" @click="toggleSidebar"></div>
    <div class="Left" v-show="showLeft">
      <Sidebar
        :selectedListId="selectedListId"
        :customLists="customLists"
        :keyword="keyword"
        :getCount="getCount"
        @update:keyword="keyword = $event"
        @selectList="selectList"
        @addList="addList"
        @deleteList="deleteList"
        @renameList="renameList"
        @toggleSidebar="toggleSidebar"
      />
    </div>
    <TaskList
      :title="currentListName"
      :tasks="filteredTasks"
      :searchResults="searchResults"
      :filterKey="filterKey"
      :sortKey="sortKey"
      @update:filterKey="filterKey = $event"
      @update:sortKey="sortKey = $event"
      @addTask="addTask"
      @deleteTask="deleteTask"
      @toggleSidebar="toggleSidebar"
    />
  </div>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.todolist {
  display: flex;
  width: 100vw;
  height: 100vh;
  background-color: #f8f8f8;
}

.Left {
  width: 290px;
  min-width: 290px;
  height: 100%;
  background-color: #fff;
  z-index: 999;
}

.Left-open {
  display: none;
  width: 50%;
  height: 100%;
  z-index: 100;
  margin-left: auto;
}

@media (max-width: 435px) {
  .todolist {
    position: relative;
  }

  .Left {
    position: absolute;
    left: 0;
    top: 0;
    box-shadow: 2px 0 8px rgba(0, 0, 0, 0.15);
    width: 220px;
    min-width: 220px;
  }

  .Left-open {
    display: block;
    width: 100%;
    left: 200px;
    top: 0;
    position: absolute;
    background-color: rgba(222, 224, 224, 0.534);
  }
}
</style>
