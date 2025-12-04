<template>
  <div class="page">
    <div class="card">
      <h1 class="title">To-Do List</h1>

      <!-- Add -->
      <div class="add-row">
        <input
          v-model="draftNew"
          type="text"
          placeholder="Add a new task"
          @keyup.enter="addTask"
        />
        <button class="btn primary" @click="addTask">Add</button>
      </div>

      <!-- Filters / counts -->
      <div class="toolbar">
        <div class="tabs">
          <button
            class="tab"
            :class="{ active: filter === 'all' }"
            @click="filter = 'all'"
          >
            All <span class="badge">{{ tasks.length }}</span>
          </button>
          <button
            class="tab"
            :class="{ active: filter === 'active' }"
            @click="filter = 'active'"
          >
            Active <span class="badge">{{ activeCount }}</span>
          </button>
          <button
            class="tab"
            :class="{ active: filter === 'completed' }"
            @click="filter = 'completed'"
          >
            Completed <span class="badge">{{ completedCount }}</span>
          </button>
        </div>

        <div class="right-tools">
          <span class="muted">{{ activeCount }} remaining</span>
          <button
            class="btn subtle"
            :disabled="completedCount === 0"
            @click="clearCompleted"
            title="Remove all completed tasks"
          >
            Clear Completed
          </button>
        </div>
      </div>

      <!-- Progress -->
      <div class="progress-wrap" v-if="tasks.length">
        <div class="progress">
          <div class="bar" :style="{ width: progressPct + '%' }" />
        </div>
        <span class="muted">{{ progressPct }}% done</span>
      </div>

      <!-- Empty state -->
      <div v-if="!filtered.length" class="empty">
        <div class="empty-emoji">🙌</div>
        <div class="empty-text">No tasks here</div>
      </div>

      <!-- List -->
      <div
        v-for="(t, i) in filtered"
        :key="t.id"
        class="item"
        :class="{ done: t.done }"
      >
        <input
          type="checkbox"
          class="check"
          v-model="t.done"
          @change="persist"
        />

        <!-- read mode -->
        <template v-if="!t.editing">
          <span class="text">{{ t.text }}</span>
          <div class="actions">
            <button class="btn subtle" @click="startEdit(t)">Edit</button>
            <button class="btn subtle danger" @click="removeTask(t.id)">Delete</button>
          </div>
        </template>

        <!-- edit mode -->
        <template v-else>
          <input
            class="edit-input"
            v-model="t.tmp"
            @keyup.enter="saveEdit(t)"
            @keyup.esc="cancelEdit(t)"
            autofocus
          />
          <div class="actions">
            <button class="btn primary" @click="saveEdit(t)">Save</button>
            <button class="btn subtle" @click="cancelEdit(t)">Cancel</button>
          </div>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "TodoPro",
  data() {
    return {
      draftNew: "",
      tasks: [], // { id, text, done, editing, tmp }
      filter: "all", // 'all' | 'active' | 'completed'
    };
  },
  computed: {
    filtered() {
      if (this.filter === "active") return this.tasks.filter((t) => !t.done);
      if (this.filter === "completed") return this.tasks.filter((t) => t.done);
      return this.tasks;
    },
    activeCount() {
      return this.tasks.filter((t) => !t.done).length;
    },
    completedCount() {
      return this.tasks.filter((t) => t.done).length;
    },
    progressPct() {
      if (!this.tasks.length) return 0;
      return Math.round((this.completedCount / this.tasks.length) * 100);
    },
  },
  created() {
    // load from localStorage
    try {
      const raw = localStorage.getItem("todo_pro_v1");
      if (raw) this.tasks = JSON.parse(raw);
    } catch {}
  },
  methods: {
    persist() {
      localStorage.setItem("todo_pro_v1", JSON.stringify(this.tasks));
    },
    addTask() {
      const text = this.draftNew.trim();
      if (!text) return;
      this.tasks.push({
        id: crypto.randomUUID?.() || Date.now() + Math.random(),
        text,
        done: false,
        editing: false,
        tmp: "",
      });
      this.draftNew = "";
      this.persist();
    },
    removeTask(id) {
      this.tasks = this.tasks.filter((t) => t.id !== id);
      this.persist();
    },
    startEdit(t) {
      t.tmp = t.text;
      t.editing = true;
    },
    saveEdit(t) {
      const val = (t.tmp || "").trim();
      if (!val) return;
      t.text = val;
      t.tmp = "";
      t.editing = false;
      this.persist();
    },
    cancelEdit(t) {
      t.tmp = "";
      t.editing = false;
    },
    clearCompleted() {
      this.tasks = this.tasks.filter((t) => !t.done);
      this.persist();
    },
  },
};
</script>

<style scoped>
/* Page */
.page {
  min-height: 100vh;
  display: grid;
  place-items: center;
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  background: radial-gradient(1200px 600px at 20% 10%, #2ba169 0%, rgba(0,0,0,0) 60%),
              linear-gradient(95deg, #2f714c, #1e9b85);
}

/* Card */
.card {
  width: min(860px, 92vw);
  background: rgba(255,255,255,0.16);
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  padding: 28px 28px 22px;
  border-radius: 22px;
  box-shadow: 0 16px 44px rgba(0,0,0,.28);
}

.title {
  color: #fff;
  text-align: center;
  font-size: clamp(22px, 3.2vw, 34px);
  font-weight: 800;
  margin: 4px 0 18px;
}

/* Add row */
.add-row {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 12px;
  margin-bottom: 12px;
}
.add-row input {
  border: none;
  border-radius: 12px;
  padding: 12px 14px;
  font-size: 15px;
  outline: none;
}
.btn {
  border: none;
  border-radius: 12px;
  padding: 10px 16px;
  font-weight: 700;
  cursor: pointer;
  transition: transform .06s ease, filter .15s ease, opacity .15s ease;
}
.btn:active { transform: translateY(1px); }
.btn[disabled] { opacity: .55; cursor: not-allowed; }
.primary { background: #c9efd8; color: #14623c; }
.subtle  { background: #e7f3ec; color: #1f6b45; }
.danger  { background: #ffe7e7; color: #7b1d1d; }

/* Toolbar */
.toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  margin: 8px 0 10px;
}
.tabs {
  display: flex; gap: 6px; flex-wrap: wrap;
}
.tab {
  background: rgba(255,255,255,.25);
  color: #083a27;
  border: 1px solid rgba(0,0,0,.06);
  padding: 6px 10px;
  border-radius: 999px;
  font-weight: 600;
  cursor: pointer;
}
.tab.active { background: #d4f1e3; }
.badge {
  background: #ffffffa8;
  color: #0c5235;
  padding: 2px 6px;
  margin-left: 6px;
  border-radius: 999px;
  font-size: 12px;
}
.right-tools { display: flex; align-items: center; gap: 10px; }
.muted { color: #e8fff5cc; font-size: 14px; }

/* Progress */
.progress-wrap {
  display: flex; align-items: center; gap: 10px;
  margin-bottom: 8px;
}
.progress {
  flex: 1; height: 8px; border-radius: 999px;
  background: rgba(255,255,255,.25); overflow: hidden;
}
.progress .bar {
  height: 100%; background: #b7f0cf; width: 0%;
}

/* Empty state */
.empty {
  text-align: center; color: #fff; opacity: .9; padding: 24px 0;
}
.empty-emoji { font-size: 44px; line-height: 1; }
.empty-text { margin-top: 6px; }

/* Item */
.item {
  display: grid;
  grid-template-columns: auto 1fr auto;
  gap: 12px;
  align-items: center;
  background: rgba(255,255,255,.22);
  border-radius: 12px;
  padding: 10px 12px;
  margin: 10px 0;
}
.item.done .text { text-decoration: line-through; opacity: .7; }
.check { width: 18px; height: 18px; accent-color: #24b27a; cursor: pointer; }

.text {
  color: #fff; overflow-wrap: anywhere;
}
.actions { display: flex; gap: 8px; }

.edit-input {
  width: 100%;
  border: none; outline: none;
  border-radius: 10px; padding: 8px 10px;
  font-size: 15px;
}
</style>
