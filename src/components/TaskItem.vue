<script setup>
import { ref, nextTick } from "vue";

const props = defineProps({
  task: { type: Object, required: true },
});

const emit = defineEmits(["delete", "update"]);

// 预设标签颜色
const TAG_COLORS = [
  { name: "红色", color: "#ef4444" },
  { name: "橙色", color: "#f97316" },
  { name: "黄色", color: "#eab308" },
  { name: "绿色", color: "#22c55e" },
  { name: "蓝色", color: "#3b82f6" },
  { name: "紫色", color: "#a855f7" },
];

// 优先级选项
const PRIORITIES = [
  { value: "high", label: "高", color: "#ef4444" },
  { value: "medium", label: "中", color: "#f97316" },
  { value: "low", label: "低", color: "#22c55e" },
];

const editingId = ref(null);
const editText = ref("");
const showTagPicker = ref(false);
const showPriorityPicker = ref(false);
const justCompleted = ref(false);

const startEdit = () => {
  editingId.value = props.task.id;
  editText.value = props.task.text;
  nextTick(() => document.querySelector(".edit-input")?.focus());
};

const saveEdit = () => {
  const text = editText.value?.trim();
  if (text) {
    props.task.text = text;
  }
  editingId.value = null;
};

const cancelEdit = () => {
  editingId.value = null;
};

const toggleImportant = () => {
  props.task.isImportant = !props.task.isImportant;
};

const setDueDate = (date) => {
  props.task.dueDate = date || null;
};

const setTag = (color) => {
  props.task.tag = color;
  showTagPicker.value = false;
};

const removeTag = () => {
  props.task.tag = null;
  showTagPicker.value = false;
};

const setPriority = (value) => {
  props.task.priority = value;
  showPriorityPicker.value = false;
};

const getPriorityInfo = (value) => {
  return PRIORITIES.find((p) => p.value === value);
};

const handleCheckChange = () => {
  if (props.task.done) {
    justCompleted.value = true;
    setTimeout(() => {
      justCompleted.value = false;
    }, 600);
  }
};
</script>

<template>
  <div
    class="task-item"
    :class="{ 'task-completed': justCompleted }"
    :style="task.tag ? { borderLeft: `4px solid ${task.tag}` } : {}"
  >
    <input
      type="checkbox"
      v-model="task.done"
      @change="handleCheckChange"
      class="task-checkbox"
    />
    <div class="task-content">
      <div class="task-main">
        <span
          v-if="editingId !== task.id"
          :class="{ done: task.done }"
          @dblclick="startEdit"
        >
          {{ task.text }}
        </span>
        <input
          v-else
          class="edit-input"
          v-model="editText"
          @keydown.enter="saveEdit"
          @blur="saveEdit"
          @keydown.esc="cancelEdit"
        />
      </div>
      <div class="task-meta" v-if="task.priority || task.dueDate">
        <span
          v-if="task.priority"
          class="priority-badge"
          :style="{ background: getPriorityInfo(task.priority)?.color }"
        >
          {{ getPriorityInfo(task.priority)?.label }}
        </span>
        <span v-if="task.dueDate" class="due-date-text">{{ task.dueDate }}</span>
      </div>
    </div>
    <div class="task-actions">
      <!-- 优先级选择 -->
      <div class="picker-wrapper">
        <span
          class="priority-btn"
          @click.stop="showPriorityPicker = !showPriorityPicker"
          title="设置优先级"
        >
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M4 15s1-1 4-1 5 2 8 2 4-1 4-1V3s-1 1-4 1-5-2-8-2-4 1-4 1z"/>
            <line x1="4" y1="22" x2="4" y2="15"/>
          </svg>
        </span>
        <div v-if="showPriorityPicker" class="picker-dropdown" @click.stop>
          <div
            v-for="p in PRIORITIES"
            :key="p.value"
            class="picker-item"
            @click="setPriority(p.value)"
          >
            <span class="priority-dot" :style="{ background: p.color }"></span>
            {{ p.label }}优先级
          </div>
          <div class="picker-item" @click="setPriority(null)">无优先级</div>
        </div>
      </div>

      <!-- 标签选择 -->
      <div class="picker-wrapper">
        <span
          class="tag-btn"
          :style="task.tag ? { color: task.tag } : {}"
          @click.stop="showTagPicker = !showTagPicker"
          title="设置标签"
        >
          <svg width="18" height="18" viewBox="0 0 24 24" fill="currentColor">
            <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2z"/>
          </svg>
        </span>
        <div v-if="showTagPicker" class="picker-dropdown" @click.stop>
          <div
            v-for="tag in TAG_COLORS"
            :key="tag.color"
            class="picker-item"
            @click="setTag(tag.color)"
          >
            <span class="tag-dot" :style="{ background: tag.color }"></span>
            {{ tag.name }}
          </div>
          <div class="picker-item" @click="removeTag">移除标签</div>
        </div>
      </div>

      <input
        type="date"
        class="date-picker"
        :value="task.dueDate"
        @change="setDueDate($event.target.value)"
        title="设置截止日期"
      />
      <span
        class="star-btn"
        :class="{ important: task.isImportant }"
        @click="toggleImportant"
        title="标记为重要"
      >
        <svg class="icon" width="20" height="20" viewBox="0 0 20 20">
          <path
            v-if="task.isImportant"
            d="M9.1 2.9a1 1 0 011.8 0l1.93 3.91 4.31.63a1 1 0 01.56 1.7l-3.12 3.05.73 4.3a1 1 0 01-1.45 1.05L10 15.51l-3.86 2.03a1 1 0 01-1.45-1.05l.74-4.3L2.3 9.14a1 1 0 01.56-1.7l4.31-.63L9.1 2.9z"
            fill="#f5a623"
          />
          <path
            v-else
            d="M9.1 2.9a1 1 0 011.8 0l1.93 3.91 4.31.63a1 1 0 01.56 1.7l-3.12 3.05.73 4.3a1 1 0 01-1.45 1.05L10 15.51l-3.86 2.03a1 1 0 01-1.45-1.05l.74-4.3L2.3 9.14a1 1 0 01.56-1.7l4.31-.63L9.1 2.9zm.9.44L8.07 7.25a1 1 0 01-.75.55L3 8.43l3.12 3.04a1 1 0 01.3.89l-.75 4.3 3.87-2.03a1 1 0 01.93 0l3.86 2.03-.74-4.3a1 1 0 01.29-.89L17 8.43l-4.32-.63a1 1 0 01-.75-.55L10 3.35z"
            fill="#999"
          />
        </svg>
      </span>
      <span class="delect" @click="emit('delete', task)">
        <svg class="icon" viewBox="0 0 1024 1024" width="20" height="20">
          <path
            d="M254.4 804.7l-.03-4.79zM614.2 259c-22.1 0-40 17.9-40 40l.37 502.2c0 22.1 17.9 40 40 40s40-18 40-40l-.35-502.2c0-22.1-17.9-40-40-40zm279-118.9l-318.9.15-.18-41.4c0-22.1-17.9-40-40-40-7.3 0-14.1 2-20 5.4-5.9-3.4-12.7-5.4-20-5.4-22.1 0-40 17.9-40 40l.19 41.5-230.1.1c-3.2-.8-6.6-1.3-10.1-1.3s-6.9.5-10.1 1.3l-73.1.03c-22.1 0-40 18-40 40s17.9 40 40 40l43.2-.02.29 617.9.06 0v42.6c0 44.2 35.9 80 80 80h40v.3l440.5-.26v-.05h40c43.7 0 79.2-35 80-78.4l-.33-583.4c0-22.1-17.9-40-40-40s-40 17.9-40 40l.29 509.3c-1.4 60.3-18.6 71.4-80 71.4l41.6 1-454.4.27 52.4-1.3c-62.7 0-79.3-11.5-80.1-75.3l.5 76.6h-.54l-.31-660.8 236.8-.11c1.2.1 2.4.17 3.6.17 1.2 0 2.4-.06 3.6-.17l32.6-.02c1.3.12 2.5.19 3.8.19 1.3 0 2.6-.07 3.9-.19l354.9-.16c22.1 0 40-17.9 40-40s-17.9-40-40-40zM774.9 815.3l.04 65.7h-.46v-65.7zM414 259c-22.1 0-40 17.9-40 40l.37 502.2c0 22.1 17.9 40 40 40s40-18 40-40l-.37-502.2c0-22.1-17.9-40-40-40z"
            fill="#272636"
          />
        </svg>
      </span>
    </div>
  </div>
</template>

<style scoped>
.task-item {
  align-items: center;
  padding: 16px 20px;
  display: flex;
  gap: 12px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  border-left: 4px solid transparent;
}

.task-item:hover {
  background: rgba(0, 0, 0, 0.02);
}

.task-completed {
  animation: completeAnim 0.6s ease;
}

@keyframes completeAnim {
  0% {
    background: transparent;
  }
  30% {
    background: rgba(34, 197, 94, 0.2);
    transform: scale(1.01);
  }
  100% {
    background: transparent;
    transform: scale(1);
  }
}

.task-checkbox {
  width: 18px;
  height: 18px;
  cursor: pointer;
  accent-color: rgb(100, 100, 241);
}

.task-content {
  flex: 1;
  min-width: 0;
}

.task-main span {
  word-break: break-all;
}

.task-meta {
  display: flex;
  gap: 8px;
  margin-top: 4px;
  align-items: center;
}

.priority-badge {
  font-size: 11px;
  color: #fff;
  padding: 2px 6px;
  border-radius: 4px;
}

.due-date-text {
  font-size: 12px;
  color: #888;
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
  width: 100%;
}

.task-actions {
  display: flex;
  align-items: center;
  gap: 6px;
  flex-shrink: 0;
}

.picker-wrapper {
  position: relative;
}

.priority-btn,
.tag-btn {
  cursor: pointer;
  display: flex;
  align-items: center;
  color: #999;
  padding: 4px;
  border-radius: 4px;
  transition: all 0.2s;
}

.priority-btn:hover,
.tag-btn:hover {
  background: rgba(0, 0, 0, 0.05);
  color: #666;
}

.picker-dropdown {
  position: absolute;
  top: 100%;
  right: 0;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  z-index: 100;
  min-width: 120px;
  padding: 4px 0;
}

.picker-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  cursor: pointer;
  font-size: 13px;
  white-space: nowrap;
}

.picker-item:hover {
  background: #f5f5f5;
}

.priority-dot,
.tag-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
}

.date-picker {
  border: none;
  outline: none;
  background: transparent;
  color: #666;
  font-size: 14px;
  cursor: pointer;
  padding: 4px;
  width: 24px;
  opacity: 0.6;
}

.date-picker::-webkit-calendar-picker-indicator {
  cursor: pointer;
}

.date-picker:hover {
  opacity: 1;
}

.star-btn {
  cursor: pointer;
  display: flex;
  align-items: center;
  padding: 4px;
}

.star-btn:hover svg path {
  fill: #f5a623;
}

.delect {
  cursor: pointer;
  display: flex;
  align-items: center;
  padding: 4px;
}

.delect:hover {
  opacity: 0.7;
}
</style>
