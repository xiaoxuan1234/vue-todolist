<script setup>
import { ref, nextTick } from "vue";

const props = defineProps({
  selectedListId: { type: String, required: true },
  customLists: { type: Array, required: true },
  keyword: { type: String, required: true },
  getCount: { type: Function, required: true },
});

const emit = defineEmits([
  "update:keyword",
  "selectList",
  "addList",
  "deleteList",
  "renameList",
  "toggleSidebar",
]);

const newListTitle = ref("");
const renameId = ref(null);
const renameText = ref("");
const ctxMenuShow = ref(false);
const ctxMenuList = ref(null);
const ctxX = ref(0);
const ctxY = ref(0);

const addList = () => {
  const title = newListTitle.value?.trim();
  if (!title) return;
  emit("addList", title);
  newListTitle.value = "";
};

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
  const text = renameText.value?.trim();
  if (text) {
    emit("renameList", list, text);
  }
  renameId.value = null;
};

const deleteList = (id) => {
  ctxMenuShow.value = false;
  emit("deleteList", id);
};

const closeCtxMenu = () => {
  ctxMenuShow.value = false;
};

defineExpose({ closeCtxMenu });
</script>

<template>
  <div class="Left-body" @click="closeCtxMenu">
    <div class="Left-header">
      <svg
        @click="emit('toggleSidebar')"
        class="Left-icon"
        viewBox="0 0 1024 1024"
        width="20"
        height="20"
      >
        <path
          d="M66.5 211.8h891c28.2 0 51-22.2 51-49.6 0-27.4-22.8-49.7-51-49.7H66.5c-28.1 0-51 22.2-51 49.6s22.8 49.7 51 49.7zm891 248.2H66.5c-28.2 0-51 22.2-51 49.7s22.8 49.6 51 49.6h891c28.2 0 51-22.2 51-49.6 0-27.4-22.9-49.7-51-49.7zm0 351.6H66.5c-28.1 0-51 22.2-51 49.7s22.8 49.6 51 49.6h891c28.2 0 51-22.2 51-49.6 0-27.4-22.8-49.7-51-49.7z"
        />
      </svg>
    </div>
    <div class="Left-content">
      <div class="Left-search">
        <svg class="icon" viewBox="0 0 1024 1024" width="20" height="18">
          <path
            d="M499.4 837.2c-207.7 0-376.7-169-376.7-376.7s169-376.7 376.7-376.7 376.7 169 376.7 376.7c0 95.5-35.8 186.6-100.8 256.5-71.1 76.3-171.6 120.2-275.9 120.2zm0-690.1C326.6 147.1 186 287.7 186 460.5s140.6 313.4 313.4 313.4c86.7 0 170.4-36.5 229.5-100 54.1-58.2 83.8-133.9 83.8-213.3 0.1-172.9-140.5-313.5-313.3-313.5z"
            fill="#666"
          />
          <path
            d="M891.4 916.5c-8.4 0-16.8-3.3-23.1-10L719 747.9c-12-12.7-11.4-32.8 1.3-44.8 12.8-12 32.8-11.4 44.8 1.3l149.4 158.7c12 12.7 11.4 32.8-1.3 44.8-6.2 5.7-14 8.6-21.8 8.6z"
            fill="#666"
          />
        </svg>
        <input
          class="search"
          type="text"
          placeholder="搜索"
          :value="keyword"
          @input="emit('update:keyword', $event.target.value)"
        />
      </div>

      <!-- 今天 -->
      <div
        class="Left-listinner"
        @click="emit('selectList', 'today')"
        :class="{ 'Selected-list': selectedListId === 'today' }"
      >
        <svg class="icon" fill="currentColor" width="20" height="20" viewBox="0 0 20 20">
          <path
            d="M10 2c.28 0 .5.22.5.5v1a.5.5 0 01-1 0v-1c0-.28.22-.5.5-.5zm0 12a4 4 0 100-8 4 4 0 000 8zm0-1a3 3 0 110-6 3 3 0 010 6zm7.5-2.5a.5.5 0 000-1h-1a.5.5 0 000 1h1zM10 16c.28 0 .5.22.5.5v1a.5.5 0 01-1 0v-1c0-.28.22-.5.5-.5zm-6.5-5.5a.5.5 0 000-1H2.46a.5.5 0 000 1H3.5zm.65-6.35c.2-.2.5-.2.7 0l1 1a.5.5 0 11-.7.7l-1-1a.5.5 0 010-.7zm.7 11.7a.5.5 0 01-.7-.7l1-1a.5.5 0 01.7.7l-1 1zm11-11.7a.5.5 0 00-.7 0l-1 1a.5.5 0 00.7.7l1-1a.5.5 0 000-.7zm-.7 11.7a.5.5 0 00.7-.7l-1-1a.5.5 0 00-.7.7l1 1z"
          />
        </svg>
        <span>今天</span>
        <div class="list-count">{{ getCount("today") }}</div>
      </div>

      <!-- 重要 -->
      <div
        class="Left-listinner"
        @click="emit('selectList', 'import')"
        :class="{ 'Selected-list': selectedListId === 'import' }"
      >
        <svg class="icon" fill="currentColor" width="20" height="20" viewBox="0 0 20 20">
          <path
            d="M9.1 2.9a1 1 0 011.8 0l1.93 3.91 4.31.63a1 1 0 01.56 1.7l-3.12 3.05.73 4.3a1 1 0 01-1.45 1.05L10 15.51l-3.86 2.03a1 1 0 01-1.45-1.05l.74-4.3L2.3 9.14a1 1 0 01.56-1.7l4.31-.63L9.1 2.9zm.9.44L8.07 7.25a1 1 0 01-.75.55L3 8.43l3.12 3.04a1 1 0 01.3.89l-.75 4.3 3.87-2.03a1 1 0 01.93 0l3.86 2.03-.74-4.3a1 1 0 01.29-.89L17 8.43l-4.32-.63a1 1 0 01-.75-.55L10 3.35z"
          />
        </svg>
        <span>重要</span>
        <div class="list-count">{{ getCount("import") }}</div>
      </div>

      <div class="partition"></div>

      <!-- 自定义列表 -->
      <div class="Left-List">
        <div
          class="Left-List-box"
          v-for="list in customLists"
          :key="list.id"
          @click="emit('selectList', list.id)"
          @contextmenu.prevent="openCtxMenu($event, list)"
          :class="{ 'Selected-list': selectedListId === list.id }"
        >
          <svg class="icon" fill="currentColor" width="20" height="20" viewBox="0 0 20 20">
            <path
              d="M3 6.5a1 1 0 100-2 1 1 0 000 2zm3-1c0-.28.22-.5.5-.5h11a.5.5 0 010 1h-11a.5.5 0 01-.5-.5zm0 5c0-.28.22-.5.5-.5h11a.5.5 0 010 1h-11a.5.5 0 01-.5-.5zm.5 4.5a.5.5 0 000 1h11a.5.5 0 000-1h-11zm-2.5.5a1 1 0 11-2 0 1 1 0 012 0zm-1-4a1 1 0 100-2 1 1 0 000 2z"
            />
          </svg>
          <span v-if="renameId !== list.id">{{ list.title }}</span>
          <input
            v-else
            class="rename-input"
            v-model="renameText"
            @keydown.enter="saveRename(list)"
            @blur="saveRename(list)"
            @keydown.esc="renameId = null"
          />
          <div class="list-count">{{ getCount(list.id) }}</div>
        </div>
      </div>

      <!-- 添加列表 -->
      <div class="Left-AddList">
        <svg class="icon" fill="currentColor" width="20" height="20" viewBox="0 0 20 20">
          <path
            d="M10 2.5a.5.5 0 00-1 0V9H2.5a.5.5 0 000 1H9v6.5a.5.5 0 001 0V10h6.5a.5.5 0 000-1H10V2.5z"
          />
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

  <!-- 右键菜单 -->
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
.Left-body {
  height: 100%;
  margin: 0 10px;
}

.Left-icon {
  margin: 28px 0 0 10px;
  cursor: pointer;
}

.Left-content {
  display: flex;
  flex-direction: column;
  margin-top: 20px;
}

.Left-List {
  max-height: 58vh;
  overflow-y: auto;
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.Left-List::-webkit-scrollbar {
  display: none;
}

.Left-listinner {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px;
  cursor: pointer;
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

.Left-List-box {
  width: 100%;
  display: flex;
  align-items: center;
  padding: 10px;
  gap: 12px;
  cursor: pointer;
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

.Left-listinner.Selected-list,
.Left-List-box.Selected-list {
  background-color: #eff6fc;
  padding-left: 7px !important;
  border-left: 3px solid rgb(100, 100, 241);
}

.partition {
  border: 1px solid rgba(0, 0, 0, 0.267);
  margin: 10px 0;
}

.rename-input {
  outline: none;
  border: none;
  font-size: 16px;
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
</style>
