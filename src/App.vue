<script setup>
import { onMounted, ref, computed, watch, nextTick } from "vue";

const newTask = ref();
const newListTitle = ref();
const showLeft = ref(true);
const selectedListId = ref("today");
const KEY = "todoData";
const filterKey = ref("all");
const editingId = ref(null);
const editText = ref("");
const keyword = ref("");
const ctxMenuShow = ref(false); // 菜单显示开关
const ctxMenuList = ref(null); // 当前右键的列表对象
const ctxX = ref(0); // 菜单定位
const ctxY = ref(0);
const renameId = ref(null);
const renameText = ref("");

let todoData = ref([
  { id: "today", title: "今天", tasks: [] },
  { id: "import", title: "重要", tasks: [] },
  { id: "tasks", title: "任务", tasks: [] },
  {
    id: "list_1",
    title: "新建列表",
    tasks: [
      {
        id: "task_1",
        text: "任务1",
        done: false,
        isImportant: false,
        dueDate: null,
        listId: "list_1",
      },
    ],
  },
]);

onMounted(() => {
  const local = loadTodos();
  if (local && Array.isArray(local)) {
    todoData.value = local;
  }
  resetTodayIfNeeded();
});

watch(todoData, () => saveTodos(todoData.value), { deep: true });

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

const presetNames = {
  today: "今天",
  import: "重要",
  tasks: "任务",
};

const currentListName = computed(() => {
  if (presetNames[selectedListId.value]) {
    return presetNames[selectedListId.value];
  }

  const list = todoData.value.find((l) => l.id === selectedListId.value);
  return list?.title || "未选择列表";
});

const getCount = (id) => {
  const list = todoData.value.find((item) => item.id === id);
  const count = list.tasks.filter((e) => !e.done);
  return count.length ? count.length : "";
};

const filteredTasks = computed(() => {
  const base = currentListTask.value;
  if (filterKey.value === "undone") return base.filter((t) => !t.done);
  if (filterKey.value === "done") return base.filter((t) => t.done);
  return base;
});

const currentListTask = computed(() => {
  const list = todoData.value.find((l) => l.id === selectedListId.value);
  console.log(list);
  return list ? list.tasks : [];
});

const allTasks = computed(() =>
  todoData.value.flatMap((list) =>
    list.tasks.map((task) => ({ ...task, listId: list.id }))
  )
);

const isToday = (dateStr) => {
  const today = new Date();
  const target = new Date(dateStr);
  return (
    today.getFullYear() === target.getFullYear() &&
    today.getMonth() === target.getMonth() &&
    today.getDate() === target.getDate()
  );
};
const selectList = (listId) => {
  selectedListId.value = listId;
};

const addList = () => {
  if (!newListTitle.value.trim()) return;
  const time = Date.now().toString();
  todoData.value.push({
    id: "List" + time,
    title: newListTitle.value,
    tasks: [],
  });
  newListTitle.value = "";
};

const addTask = () => {
  if (!newTask.value?.trim()) return;
  const list = todoData.value.find((l) => l.id === selectedListId.value);
  if (!list) return;

  const taskTpl = {
    id: "task" + Date.now().toString(),
    text: newTask.value.trim(),
    done: false,
    isImportant: false,
    dueDate: null,
    listId: list.id,
  };
  if (selectedListId.value === "import") {
    taskTpl.isImportant = true;
  }
  if (selectedListId.value === "today") {
    taskTpl.dueDate = new Date().toISOString().slice(0, 10);
  }

  list.tasks.push(taskTpl);
  newTask.value = "";
};

const deleteTask = (task) => {
  const list = todoData.value.find((l) => l.id === task.listId);
  if (!list) return;
  const idx = list.tasks.findIndex((t) => t.id === task.id);
  if (idx > -1) list.tasks.splice(idx, 1);
};

const startEdit = (task) => {
  editingId.value = task.id;
  editText.value = task.text;
  nextTick(() => document.querySelector(".edit-input")?.focus());
};

const saveEdit = (task) => {
  if (editText.value.trim()) task.text = editText.value.trim();
  editingId.value = null;
};

const cancelEdit = () => {
  editingId.value = null;
};

const searchResults = computed(() => {
  if (!keyword.value.trim()) return [];
  const kw = keyword.value.trim().toLowerCase();
  const res = [];
  todoData.value.forEach((list) =>
    list.tasks.forEach((t) => {
      if (t.text.toLowerCase().includes(kw)) {
        t.listTitle = list.title;
        res.push(t);
      }
    })
  );
  return res;
});

const openCtxMenu = (e, list) => {
  e.stopPropagation();
  ctxMenuList.value = list;
  ctxX.value = e.clientX;
  ctxY.value = e.clientY;
  ctxMenuShow.value = true;
};

const startRename = (list) => {
  ctxMenuShow.value = false;
  renameId.value = list.id;
  renameText.value = list.title;
  nextTick(() => document.querySelector(".rename-input")?.focus());
};

const saveRename = (list) => {
  if (renameText.value.trim()) list.title = renameText.value.trim();
  renameId.value = null;
};

const deleteList = (id) => {
  ctxMenuShow.value = false;
  if (!confirm("确定删除该列表吗？")) return;
  todoData.value = todoData.value.filter((l) => l.id !== id);
  if (selectedListId.value === id) selectedListId.value = "today";
};

const LAST_VISIT_KEY = "lastVisitDate";

function shouldResetToday() {
  const last = localStorage.getItem(LAST_VISIT_KEY);
  if (!last) return true;

  const lastDate = new Date(last);
  const today = new Date();

  return (
    lastDate.getFullYear() !== today.getFullYear() ||
    lastDate.getMonth() !== today.getMonth() ||
    lastDate.getDate() !== today.getDate()
  );
}

function resetTodayIfNeeded() {
  if (shouldResetToday()) {
    const todayList = todoData.value.find((l) => l.id === "today");
    if (todayList) {
      todayList.tasks = [];
    }
    localStorage.setItem(LAST_VISIT_KEY, new Date().toISOString());
  }
}
</script>

<template>
  <div class="todolist" @click="ctxMenuShow = false" @contextmenu.prevent>
    <div
      class="Left-open"
      v-show="showLeft"
      @click="showLeft = !showLeft"
    ></div>
    <div class="Left" v-show="showLeft">
      <div class="Left-body">
        <div class="Left-header">
          <svg
            @click="showLeft = !showLeft"
            t="1764658081387"
            class="Left-icon"
            viewBox="0 0 1024 1024"
            version="1.1"
            xmlns="http://www.w3.org/2000/svg"
            p-id="4725"
            width="20"
            height="20"
          >
            <path
              d="M66.488889 211.781818h891.022222c28.198788 0 50.980202-22.238384 50.980202-49.648485 0-27.397172-22.768485-49.648485-50.980202-49.648485H66.488889C38.341818 112.484848 15.508687 134.723232 15.508687 162.133333s22.833131 49.648485 50.980202 49.648485z m891.009293 248.242424H66.488889C38.277172 460.024242 15.508687 482.262626 15.508687 509.672727s22.768485 49.648485 50.980202 49.648485h891.022222c28.198788 0 50.980202-22.238384 50.980202-49.648485-0.012929-27.410101-22.923636-49.648485-50.993131-49.648485z m0 351.63798H66.488889c-28.134141 0-50.980202 22.238384-50.980202 49.648485s22.833131 49.648485 50.980202 49.648485h891.022222c28.198788 0 50.980202-22.238384 50.980202-49.648485-0.012929-27.397172-22.781414-49.648485-50.993131-49.648485z m0 0"
              p-id="4726"
            ></path>
          </svg>
        </div>
        <div class="Left-content">
          <div class="Left-search">
            <svg
              t="1764660333997"
              class="icon"
              viewBox="0 0 1024 1024"
              version="1.1"
              xmlns="http://www.w3.org/2000/svg"
              p-id="12429"
              width="20"
              height="18"
            >
              <path
                d="M499.4 837.2c-207.7 0-376.7-169-376.7-376.7s169-376.7 376.7-376.7 376.7 169 376.7 376.7c0 95.5-35.8 186.6-100.8 256.5-71.1 76.3-171.6 120.2-275.9 120.2z m0-690.1C326.6 147.1 186 287.7 186 460.5s140.6 313.4 313.4 313.4c86.7 0 170.4-36.5 229.5-100 54.1-58.2 83.8-133.9 83.8-213.3 0.1-172.9-140.5-313.5-313.3-313.5z"
                fill="#666666"
                p-id="12430"
              ></path>
              <path
                d="M891.4 916.5c-8.4 0-16.8-3.3-23.1-10L719 747.9c-12-12.7-11.4-32.8 1.3-44.8 12.8-12 32.8-11.4 44.8 1.3l149.4 158.7c12 12.7 11.4 32.8-1.3 44.8-6.2 5.7-14 8.6-21.8 8.6z"
                fill="#666666"
                p-id="12431"
              ></path>
            </svg>
            <input
              class="search"
              type="text"
              placeholder="搜索"
              v-model="keyword"
            />
          </div>
          <div
            class="Left-listinner"
            @click="selectList('today')"
            :class="{ 'Selected-list': selectedListId === 'today' }"
          >
            <svg
              class="icon"
              aria-label=""
              fill="currentColor"
              aria-hidden="true"
              width="20"
              height="20"
              viewBox="0 0 20 20"
              xmlns="http://www.w3.org/2000/svg"
              focusable="false"
            >
              <path
                d="M10 2c.28 0 .5.22.5.5v1a.5.5 0 01-1 0v-1c0-.28.22-.5.5-.5zm0 12a4 4 0 100-8 4 4 0 000 8zm0-1a3 3 0 110-6 3 3 0 010 6zm7.5-2.5a.5.5 0 000-1h-1a.5.5 0 000 1h1zM10 16c.28 0 .5.22.5.5v1a.5.5 0 01-1 0v-1c0-.28.22-.5.5-.5zm-6.5-5.5a.5.5 0 000-1H2.46a.5.5 0 000 1H3.5zm.65-6.35c.2-.2.5-.2.7 0l1 1a.5.5 0 11-.7.7l-1-1a.5.5 0 010-.7zm.7 11.7a.5.5 0 01-.7-.7l1-1a.5.5 0 01.7.7l-1 1zm11-11.7a.5.5 0 00-.7 0l-1 1a.5.5 0 00.7.7l1-1a.5.5 0 000-.7zm-.7 11.7a.5.5 0 00.7-.7l-1-1a.5.5 0 00-.7.7l1 1z"
                fill="currentColor"
              ></path>
            </svg>
            <span>今天</span>
            <div class="list-count">{{ getCount("today") }}</div>
          </div>
          <div
            class="Left-listinner"
            @click="selectList('import')"
            :class="{ 'Selected-list': selectedListId === 'import' }"
          >
            <svg
              class="icon"
              aria-label=""
              fill="currentColor"
              aria-hidden="true"
              width="20"
              height="20"
              viewBox="0 0 20 20"
              xmlns="http://www.w3.org/2000/svg"
              focusable="false"
            >
              <path
                d="M9.1 2.9a1 1 0 011.8 0l1.93 3.91 4.31.63a1 1 0 01.56 1.7l-3.12 3.05.73 4.3a1 1 0 01-1.45 1.05L10 15.51l-3.86 2.03a1 1 0 01-1.45-1.05l.74-4.3L2.3 9.14a1 1 0 01.56-1.7l4.31-.63L9.1 2.9zm.9.44L8.07 7.25a1 1 0 01-.75.55L3 8.43l3.12 3.04a1 1 0 01.3.89l-.75 4.3 3.87-2.03a1 1 0 01.93 0l3.86 2.03-.74-4.3a1 1 0 01.29-.89L17 8.43l-4.32-.63a1 1 0 01-.75-.55L10 3.35z"
                fill="currentColor"
              ></path>
            </svg>
            <span>重要</span>
            <div class="list-count">{{ getCount("import") }}</div>
          </div>
          <div class="partition"></div>
          <div class="Left-List">
            <div
              class="Left-List-box"
              v-for="value in todoData.filter(
                (l) => !['today', 'import', 'tasks'].includes(l.id)
              )"
              :key="value.id"
              @click="selectList(value.id, value.title)"
              @contextmenu.prevent="openCtxMenu($event, value)"
              :class="{ 'Selected-list': selectedListId === value.id }"
            >
              <svg
                class="icon"
                aria-label=""
                fill="currentColor"
                aria-hidden="true"
                width="20"
                height="20"
                viewBox="0 0 20 20"
                xmlns="http://www.w3.org/2000/svg"
              >
                <path
                  d="M3 6.5a1 1 0 100-2 1 1 0 000 2zm3-1c0-.28.22-.5.5-.5h11a.5.5 0 010 1h-11a.5.5 0 01-.5-.5zm0 5c0-.28.22-.5.5-.5h11a.5.5 0 010 1h-11a.5.5 0 01-.5-.5zm.5 4.5a.5.5 0 000 1h11a.5.5 0 000-1h-11zm-2.5.5a1 1 0 11-2 0 1 1 0 012 0zm-1-4a1 1 0 100-2 1 1 0 000 2z"
                  fill="currentColor"
                ></path>
              </svg>
              <span v-if="renameId !== value.id">{{ value.title }}</span>
              <input
                v-else
                class="rename-input"
                v-model="renameText"
                @keydown.enter="saveRename(value)"
                @blur="saveRename(value)"
                @keydown.esc="renameId = null"
              />
              <div class="list-count">
                {{ getCount(value.id) }}
              </div>
            </div>
          </div>
          <div class="Left-AddList">
            <svg
              class="icon"
              aria-label=""
              fill="currentColor"
              aria-hidden="true"
              width="20"
              height="20"
              viewBox="0 0 20 20"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M10 2.5a.5.5 0 00-1 0V9H2.5a.5.5 0 000 1H9v6.5a.5.5 0 001 0V10h6.5a.5.5 0 000-1H10V2.5z"
                fill="currentColor"
              ></path>
            </svg>
            <input
              type="text"
              placeholder="添加列表"
              v-model="newListTitle"
              @keydown.enter="addList"
              maxlength="60"
            />
          </div>
        </div>
      </div>
    </div>
    <div class="center">
      <div class="center-header">
        <svg
          @click="showLeft = !showLeft"
          t="1764658081387"
          class="more-icon"
          viewBox="0 0 1024 1024"
          version="1.1"
          xmlns="http://www.w3.org/2000/svg"
          p-id="4725"
          width="20"
          height="20"
        >
          <path
            d="M66.488889 211.781818h891.022222c28.198788 0 50.980202-22.238384 50.980202-49.648485 0-27.397172-22.768485-49.648485-50.980202-49.648485H66.488889C38.341818 112.484848 15.508687 134.723232 15.508687 162.133333s22.833131 49.648485 50.980202 49.648485z m891.009293 248.242424H66.488889C38.277172 460.024242 15.508687 482.262626 15.508687 509.672727s22.768485 49.648485 50.980202 49.648485h891.022222c28.198788 0 50.980202-22.238384 50.980202-49.648485-0.012929-27.410101-22.923636-49.648485-50.993131-49.648485z m0 351.63798H66.488889c-28.134141 0-50.980202 22.238384-50.980202 49.648485s22.833131 49.648485 50.980202 49.648485h891.022222c28.198788 0 50.980202-22.238384 50.980202-49.648485-0.012929-27.397172-22.781414-49.648485-50.993131-49.648485z m0 0"
            p-id="4726"
          ></path>
        </svg>
        <div class="title">
          {{ searchResults.length ? "搜索结果" : currentListName }}
        </div>
      </div>
      <div class="center-content">
        <div class="addTask">
          <svg
            class="icon"
            aria-label=""
            fill="currentColor"
            aria-hidden="true"
            width="20"
            height="20"
            viewBox="0 0 20 20"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              d="M10 2.5a.5.5 0 00-1 0V9H2.5a.5.5 0 000 1H9v6.5a.5.5 0 001 0V10h6.5a.5.5 0 000-1H10V2.5z"
              fill="currentColor"
            ></path>
          </svg>
          <input
            type="text"
            placeholder="添加任务"
            v-model="newTask"
            @keydown.enter="addTask"
          />
        </div>
        <div class="center-select-show">
          <div
            class="all-task"
            :class="{ active: filterKey === 'all' }"
            @click="filterKey = 'all'"
          >
            全部
          </div>
          <div
            class="no-done-task"
            :class="{ active: filterKey === 'undone' }"
            @click="filterKey = 'undone'"
          >
            未完成
          </div>
          <div
            class="done-task"
            :class="{ active: filterKey === 'done' }"
            @click="filterKey = 'done'"
          >
            已完成
          </div>
        </div>
        <div class="search-results" v-if="searchResults.length">
          <div class="task-item" v-for="t in searchResults" :key="t.id">
            <input type="checkbox" v-model="t.done" />
            <div class="search1">
              <span :class="{ done: t.done }">{{ t.text }}</span>
              <span class="list-tag">{{ t.listTitle }}</span>
            </div>
            <span
              class="isimport"
              v-show="!t.isImportant"
              @click="t.isImportant = !t.isImportant"
            >
              <svg
                t="1764769692696"
                class="icon"
                viewBox="0 0 1024 1024"
                version="1.1"
                xmlns="http://www.w3.org/2000/svg"
                p-id="4841"
                width="20"
                height="20"
              >
                <path
                  d="M908.1 353.1l-253.9-36.9L540.7 86.1c-3.1-6.3-8.2-11.4-14.5-14.5-15.8-7.8-35-1.3-42.9 14.5L369.8 316.2l-253.9 36.9c-7 1-13.4 4.3-18.3 9.3-12.3 12.7-12.1 32.9 0.6 45.3l183.7 179.1-43.4 252.9c-1.2 6.9-0.1 14.1 3.2 20.3 8.2 15.6 27.6 21.7 43.2 13.4L512 754l227.1 119.4c6.2 3.3 13.4 4.4 20.3 3.2 17.4-3 29.1-19.5 26.1-36.9l-43.4-252.9 183.7-179.1c5-4.9 8.3-11.3 9.3-18.3 2.7-17.5-9.5-33.7-27-36.3zM664.8 561.6l36.1 210.3L512 672.7 323.1 772l36.1-210.3-152.8-149L417.6 382 512 190.7 606.4 382l211.2 30.7-152.8 148.9z"
                  p-id="4842"
                ></path>
              </svg>
            </span>
            <span
              class="isimport"
              v-show="t.isImportant"
              @click="t.isImportant = !t.isImportant"
            >
              <svg
                t="1764769826201"
                class="icon"
                viewBox="0 0 1024 1024"
                version="1.1"
                xmlns="http://www.w3.org/2000/svg"
                p-id="5090"
                width="20"
                height="20"
              >
                <path
                  d="M949.12 386.592c-4.864-15.008-17.856-25.952-33.44-28.192l-256.992-37.344-114.944-232.896c-6.976-14.144-21.376-23.104-37.152-23.104-15.776 0-30.176 8.96-37.152 23.104l-114.944 232.896L97.472 358.4c-15.616 2.272-28.576 13.184-33.44 28.192s-0.8 31.456 10.496 42.464l185.984 181.28-43.904 255.968c-2.656 15.552 3.712 31.264 16.48 40.544 12.768 9.28 29.664 10.496 43.648 3.136l229.888-120.864 229.856 120.864c6.048 3.168 12.672 4.768 19.264 4.768 8.576 0 17.152-2.656 24.352-7.904 12.768-9.28 19.136-24.992 16.48-40.544l-43.904-255.968 185.984-181.28C949.92 418.048 953.984 401.6 949.12 386.592z"
                  fill="#1296db"
                  p-id="5091"
                ></path>
              </svg>
            </span>
            <span class="delect" @click="deleteTask(t)">
              <svg
                t="1764770107982"
                class="icon"
                viewBox="0 0 1024 1024"
                version="1.1"
                xmlns="http://www.w3.org/2000/svg"
                p-id="6128"
                width="20"
                height="20"
              >
                <path
                  d="M254.398526 804.702412l-0.030699-4.787026C254.367827 801.546535 254.380106 803.13573 254.398526 804.702412zM614.190939 259.036661c-22.116717 0-40.047088 17.910928-40.047088 40.047088l0.37146 502.160911c0 22.097274 17.930371 40.048111 40.047088 40.048111s40.048111-17.950837 40.048111-40.048111l-0.350994-502.160911C654.259516 276.948613 636.328122 259.036661 614.190939 259.036661zM893.234259 140.105968l-318.891887 0.148379-0.178055-41.407062c0-22.13616-17.933441-40.048111-40.067554-40.048111-7.294127 0-14.126742 1.958608-20.017916 5.364171-5.894244-3.405563-12.729929-5.364171-20.031219-5.364171-22.115694 0-40.047088 17.911952-40.047088 40.048111l0.188288 41.463344-230.115981 0.106424c-3.228531-0.839111-6.613628-1.287319-10.104125-1.287319-3.502777 0-6.89913 0.452301-10.136871 1.296529l-73.067132 0.033769c-22.115694 0-40.048111 17.950837-40.048111 40.047088 0 22.13616 17.931395 40.048111 40.048111 40.048111l43.176358-0.020466 0.292666 617.902982 0.059352 0 0 42.551118c0 44.233434 35.862789 80.095199 80.095199 80.095199l40.048111 0 0 0.302899 440.523085-0.25685 0-0.046049 40.048111 0c43.663452 0 79.146595-34.95 80.054267-78.395488l-0.329505-583.369468c0-22.135136-17.930371-40.047088-40.048111-40.047088-22.115694 0-40.047088 17.911952-40.047088 40.047088l0.287549 509.324054c-1.407046 60.314691-18.594497 71.367421-79.993892 71.367421l41.575908 1.022283-454.442096 0.26606 52.398394-1.288343c-62.715367 0-79.305207-11.522428-80.0645-75.308173l0.493234 76.611865-0.543376 0-0.313132-660.818397 236.82273-0.109494c1.173732 0.103354 2.360767 0.166799 3.561106 0.166799 1.215688 0 2.416026-0.063445 3.604084-0.169869l32.639375-0.01535c1.25355 0.118704 2.521426 0.185218 3.805676 0.185218 1.299599 0 2.582825-0.067538 3.851725-0.188288l354.913289-0.163729c22.115694 0 40.050158-17.911952 40.050158-40.047088C933.283394 158.01792 915.349953 140.105968 893.234259 140.105968zM774.928806 815.294654l0.036839 65.715701-0.459464 0L774.928806 815.294654zM413.953452 259.036661c-22.116717 0-40.048111 17.910928-40.048111 40.047088l0.37146 502.160911c0 22.097274 17.931395 40.048111 40.049135 40.048111 22.115694 0 40.047088-17.950837 40.047088-40.048111l-0.37146-502.160911C454.00054 276.948613 436.069145 259.036661 413.953452 259.036661z"
                  fill="#272636"
                  p-id="6129"
                ></path>
              </svg>
            </span>
          </div>
        </div>
        <div class="task" v-else>
          <div class="task-item" v-for="task in filteredTasks" :key="task.id">
            <input type="checkbox" v-model="task.done" />
            <span
              v-if="editingId !== task.id"
              :class="{ done: task.done }"
              @dblclick="startEdit(task)"
            >
              {{ task.text }}
            </span>
            <input
              v-else
              class="edit-input"
              v-model="editText"
              @keydown.enter="saveEdit(task)"
              @blur="saveEdit(task)"
              @keydown.esc="cancelEdit"
            />
            <span
              class="isimport"
              v-show="!task.isImportant"
              @click="task.isImportant = !task.isImportant"
            >
              <svg
                t="1764769692696"
                class="icon"
                viewBox="0 0 1024 1024"
                version="1.1"
                xmlns="http://www.w3.org/2000/svg"
                p-id="4841"
                width="20"
                height="20"
              >
                <path
                  d="M908.1 353.1l-253.9-36.9L540.7 86.1c-3.1-6.3-8.2-11.4-14.5-14.5-15.8-7.8-35-1.3-42.9 14.5L369.8 316.2l-253.9 36.9c-7 1-13.4 4.3-18.3 9.3-12.3 12.7-12.1 32.9 0.6 45.3l183.7 179.1-43.4 252.9c-1.2 6.9-0.1 14.1 3.2 20.3 8.2 15.6 27.6 21.7 43.2 13.4L512 754l227.1 119.4c6.2 3.3 13.4 4.4 20.3 3.2 17.4-3 29.1-19.5 26.1-36.9l-43.4-252.9 183.7-179.1c5-4.9 8.3-11.3 9.3-18.3 2.7-17.5-9.5-33.7-27-36.3zM664.8 561.6l36.1 210.3L512 672.7 323.1 772l36.1-210.3-152.8-149L417.6 382 512 190.7 606.4 382l211.2 30.7-152.8 148.9z"
                  p-id="4842"
                ></path>
              </svg>
            </span>
            <span
              class="isimport"
              v-show="task.isImportant"
              @click="task.isImportant = !task.isImportant"
            >
              <svg
                t="1764769826201"
                class="icon"
                viewBox="0 0 1024 1024"
                version="1.1"
                xmlns="http://www.w3.org/2000/svg"
                p-id="5090"
                width="20"
                height="20"
              >
                <path
                  d="M949.12 386.592c-4.864-15.008-17.856-25.952-33.44-28.192l-256.992-37.344-114.944-232.896c-6.976-14.144-21.376-23.104-37.152-23.104-15.776 0-30.176 8.96-37.152 23.104l-114.944 232.896L97.472 358.4c-15.616 2.272-28.576 13.184-33.44 28.192s-0.8 31.456 10.496 42.464l185.984 181.28-43.904 255.968c-2.656 15.552 3.712 31.264 16.48 40.544 12.768 9.28 29.664 10.496 43.648 3.136l229.888-120.864 229.856 120.864c6.048 3.168 12.672 4.768 19.264 4.768 8.576 0 17.152-2.656 24.352-7.904 12.768-9.28 19.136-24.992 16.48-40.544l-43.904-255.968 185.984-181.28C949.92 418.048 953.984 401.6 949.12 386.592z"
                  fill="#1296db"
                  p-id="5091"
                ></path>
              </svg>
            </span>
            <span class="delect" @click="deleteTask(task)">
              <svg
                t="1764770107982"
                class="icon"
                viewBox="0 0 1024 1024"
                version="1.1"
                xmlns="http://www.w3.org/2000/svg"
                p-id="6128"
                width="20"
                height="20"
              >
                <path
                  d="M254.398526 804.702412l-0.030699-4.787026C254.367827 801.546535 254.380106 803.13573 254.398526 804.702412zM614.190939 259.036661c-22.116717 0-40.047088 17.910928-40.047088 40.047088l0.37146 502.160911c0 22.097274 17.930371 40.048111 40.047088 40.048111s40.048111-17.950837 40.048111-40.048111l-0.350994-502.160911C654.259516 276.948613 636.328122 259.036661 614.190939 259.036661zM893.234259 140.105968l-318.891887 0.148379-0.178055-41.407062c0-22.13616-17.933441-40.048111-40.067554-40.048111-7.294127 0-14.126742 1.958608-20.017916 5.364171-5.894244-3.405563-12.729929-5.364171-20.031219-5.364171-22.115694 0-40.047088 17.911952-40.047088 40.048111l0.188288 41.463344-230.115981 0.106424c-3.228531-0.839111-6.613628-1.287319-10.104125-1.287319-3.502777 0-6.89913 0.452301-10.136871 1.296529l-73.067132 0.033769c-22.115694 0-40.048111 17.950837-40.048111 40.047088 0 22.13616 17.931395 40.048111 40.048111 40.048111l43.176358-0.020466 0.292666 617.902982 0.059352 0 0 42.551118c0 44.233434 35.862789 80.095199 80.095199 80.095199l40.048111 0 0 0.302899 440.523085-0.25685 0-0.046049 40.048111 0c43.663452 0 79.146595-34.95 80.054267-78.395488l-0.329505-583.369468c0-22.135136-17.930371-40.047088-40.048111-40.047088-22.115694 0-40.047088 17.911952-40.047088 40.047088l0.287549 509.324054c-1.407046 60.314691-18.594497 71.367421-79.993892 71.367421l41.575908 1.022283-454.442096 0.26606 52.398394-1.288343c-62.715367 0-79.305207-11.522428-80.0645-75.308173l0.493234 76.611865-0.543376 0-0.313132-660.818397 236.82273-0.109494c1.173732 0.103354 2.360767 0.166799 3.561106 0.166799 1.215688 0 2.416026-0.063445 3.604084-0.169869l32.639375-0.01535c1.25355 0.118704 2.521426 0.185218 3.805676 0.185218 1.299599 0 2.582825-0.067538 3.851725-0.188288l354.913289-0.163729c22.115694 0 40.050158-17.911952 40.050158-40.047088C933.283394 158.01792 915.349953 140.105968 893.234259 140.105968zM774.928806 815.294654l0.036839 65.715701-0.459464 0L774.928806 815.294654zM413.953452 259.036661c-22.116717 0-40.048111 17.910928-40.048111 40.047088l0.37146 502.160911c0 22.097274 17.931395 40.048111 40.049135 40.048111 22.115694 0 40.047088-17.950837 40.047088-40.048111l-0.37146-502.160911C454.00054 276.948613 436.069145 259.036661 413.953452 259.036661z"
                  fill="#272636"
                  p-id="6129"
                ></path>
              </svg>
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
  <Teleport to="body">
    <div
      v-if="ctxMenuShow"
      class="ctx-menu"
      :style="{ left: ctxX + 'px', top: ctxY + 'px' }"
      @click.stop
    >
      <div @click="startRename(ctxMenuList)">重命名</div>
      <div @click="deleteList(ctxMenuList.id)">删除</div>
    </div>
  </Teleport>
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

/* 左边区域 */
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

.Left-body {
  height: 100%;
  margin: 0px 10px;
}

.Left-icon {
  margin: 28px 0px 0px 10px;
}

.Left-content {
  display: flex;
  flex-direction: column;
  margin-top: 20px;
}

.Left-List {
  max-height: 58vh;
  overflow-y: auto;
}

.Left-listinner {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px;
}

.Left-search {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px;
  color: rgb(100, 100, 241);
}

.search {
  width: 100%;
  outline: none;
  border: none;
  font-size: 16px;
}

.search::placeholder {
  color: rgb(100, 100, 241);
}

.search:focus::placeholder {
  color: #b6b4b4;
}

.Left-Listitem {
  padding-bottom: 10px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.267);
}

.Left-List-body {
  margin-top: 15px;
  list-style: none;
  overflow-y: auto;
}

.Left-List-box {
  width: 100%;
  display: flex;
  align-items: center;
  padding: 10px;
  gap: 12px;
}

.Left-List-box span {
  width: 75%;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.list-count {
  margin-left: auto;
}

.Left-AddList {
  display: flex;
  align-items: center;
  padding: 10px;
  gap: 12px;
  color: rgb(100, 100, 241);
}

.Left-AddList input {
  width: 100%;
  border: none;
  outline: none;
  font-size: 16px;
}

.Left-AddList input::placeholder {
  color: rgb(100, 100, 241);
}

.Left-AddList input:focus::placeholder {
  color: #b6b4b4;
}

/* 选择列表样式 */
.Left-listinner.Selected-list,
.Left-List-box.Selected-list {
  background-color: #eff6fc;
  padding-left: 7px !important;
  border-left: 3px solid rgb(100, 100, 241);
}

/* 中间内容区域 */
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
  padding-top: 15px;
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

.addTask {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 98%;
  padding: 0px 15px;
  margin: 0 auto;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.08);
  color: rgb(100, 100, 241);
  border-radius: 12px;
}

.addTask input {
  width: 100%;
  height: 8px;
  padding: 30px 0px;
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

.center-select-show {
  display: flex;
  padding: 20px 20px 20px 15px;
  gap: 10px;
}

.no-done {
  font-size: 16px;
  margin-left: auto;
  flex: 0 0 100px;
  text-align: right;
}

.done-task,
.no-done,
.no-done-task,
.all-task {
  padding: 10px;
}

.more-icon {
  flex: 0 0 auto;
}

.done-task:hover,
.no-done-task:hover,
.all-task:hover {
  background-color: #73a8f850;
  border-radius: 5px;
}

.active {
  background-color: #73a8f850;
  border-radius: 5px;
}

.select-Task-show {
  background-color: #73a8f850;
  border-radius: 5px;
}

.center-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  margin-bottom: 30px;
}

.task {
  flex: 1;
  margin: 0px 10px;
  overflow-y: auto;
}

.task-item {
  align-items: center;
  justify-content: center;
  padding: 20px 20px 20px 20px;
  display: flex;
  gap: 10px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.267);
}

.task-item span {
  flex-wrap: wrap;
  word-break: break-all;
}

.delect {
  margin-left: auto;
}

.partition {
  border: 1px solid rgba(0, 0, 0, 0.267);
  margin: 10px 0px;
}

.done {
  text-decoration: line-through;
  color: #999;
}

.edit-input {
  border: none;
  outline: none;
  font-size: 16px;
  background-color: #f8f8f8;
}

.list-tag {
  font-size: 12px;
  color: #999;
}

.search1 {
  display: flex;
  flex-direction: column;
}

.ctx-menu {
  position: fixed;
  background: #fff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  border-radius: 4px;
  font-size: 14px;
  z-index: 9999;
}
.ctx-menu div {
  padding: 10px 15px;
  cursor: pointer;
}
.ctx-menu div:hover {
  background: #f5f5f5;
}

.rename-input {
  outline: none;
  border: none;
  font-size: 16px;
}

@media (max-width: 432px) {
  .todolist {
    position: relative;
  }

  .Left {
    position: absolute;
    left: 0;
    top: 0;
    box-shadow: 2px 0 8px rgba(0, 0, 0, 0.15);
    width: 220px;
  }

  .Left-open {
    display: block;
    width: 100%;
    left: 200px;
    top: 0;
    position: absolute;
    background-color: rgba(222, 224, 224, 0.534);
  }

  .center {
    width: 100%;
  }
}
</style>
