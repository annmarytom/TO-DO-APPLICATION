<template>
    <div class="page">
        <div class="todo-card">
            <h1 class="title">To-Do List App</h1>
            <div class="input-row">
                <input v-model="newTask" type="text" placeholder="Add a new task" @keyup.enter="addTask" />
                <button @click="addTask">Add</button>
            </div>

            <!-- tasks list -->
            <!-- <div v-for="(task, index) in tasks" :key="index" class="task-row">
        <span class="task-text">{{ task }}</span>
        <div class="task-row-div">
          <button class="delete-btn" @click="removeTask(index)">Delete</button>
          <button class="edit-btn" @click="editTask(index)">Edit</button>
        </div>
      </div> -->

            <!-- List -->
            <div v-for="(task, index) in tasks" :key="task.id" class="task-row">
                <!-- READ MODE -->
                <template v-if="!task.editing">
                    <span class="task-text">{{ task.text }}</span>
                    <div class="task-row-div">
                        <button class="delete-btn" @click="removeTask(index)">Delete</button>
                        <button class="edit-btn" @click="startEdit(index)">Edit</button>
                    </div>
                </template>

                <!-- EDIT MODE -->
                <template v-else>
                    <input v-model="task.draft" class="edit-input" @keyup.enter="saveEdit(index)"
                        @keyup.esc="cancelEdit(index)" autofocus />
                    <div class="task-row-div">
                        <button class="delete-btn" @click="saveEdit(index)">Save</button>
                        <button class="edit-btn" @click="cancelEdit(index)">Cancel</button>
                    </div>
                </template>
            </div>
            <p v-if="tasks.length === 0" class="empty-msg">
                <img :src="noTaskImage" width="100" height="100" />
            <h4>No Tasks</h4>
            </p>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            newTask: "",
            tasks: [],
            noTaskImage: "/images/Man-Gesturing-No-3d-Medium-icon.png",
        };
    },
    methods: {
        addTask() {
            const text = this.newTask.trim();
            if (!text) return;

            this.tasks.push({
                id: crypto.randomUUID?.() || Date.now() + Math.random(),
                text,
                editing: false,
                draft: "",
            });

            this.newTask = "";
        },
        removeTask(index) {
            this.tasks.splice(index, 1);
        },
        startEdit(index) {
            const t = this.tasks[index];
            t.draft = t.text;     // pre-fill with current text
            t.editing = true;     // switch row to edit mode
        },
        saveEdit(index) {
            const t = this.tasks[index];
            const val = (t.draft || "").trim();
            if (!val) return;     // ignore empty saves
            t.text = val;
            t.draft = "";
            t.editing = false;    // back to read mode
        },
        cancelEdit(index) {
            const t = this.tasks[index];
            t.draft = "";
            t.editing = false;    // discard changes
        },
    },
};
</script>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

.page {
    height: auto;
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 0;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
        sans-serif;
    background: linear-gradient(to right, #376848, #2a8342, #1ba386);
}

.todo-card {
    width: 600px;
    max-width: 90%;
    padding: 30px 40px;
    border-radius: 22px;
    background: rgba(255, 255, 255, 0.18);
    box-shadow: 0 18px 45px rgba(0, 0, 0, 0.25);
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    text-align: center;
}

.title {
    margin: 0 0 25px;
    font-size: 28px;
    font-weight: 700;
    color: #ffffff;
}

.input-row {
    display: flex;
    gap: 10px;
    margin-bottom: 16px;
}

.input-row input {
    flex: 1;
    padding: 10px 12px;
    border-radius: 10px;
    border: none;
    outline: none;
    font-size: 14px;
}

.input-row button {
    padding: 0 20px;
    border-radius: 10px;
    border: none;
    cursor: pointer;
    font-weight: 600;
    background: #cbead7;
    color: #155e37;
    transition: background 0.2s, transform 0.1s;
}

.task-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: rgba(255, 255, 255, 0.25);
    border-radius: 10px;
    padding: 8px 12px;
    margin-bottom: 8px;
    max-width: 100%;
}

.task-text {
    color: #ffffff;
    text-align: left;
    flex: 1;
    margin-right: 10px;
    white-space: normal;
    overflow-wrap: anywhere;


}

.edit-input {
    flex: 1;
    padding: 8px 10px;
    border-radius: 8px;
    border: none;
    outline: none;
    font-size: 14px;
}

.delete-btn {
    padding: 6px 16px;
    border-radius: 8px;
    border: none;
    cursor: pointer;
    font-weight: 600;
    background: #e0f1e4;
    color: #155e37;
    transition: background 0.2s, transform 0.1s;
}

.input-row button:hover,
.delete-btn:hover {
    background: #b6dcc6;
}

.input-row button:active,
.delete-btn:active {
    transform: scale(0.97);
}

.empty-msg {
    margin-top: 10px;
    font-size: 14px;
    opacity: 0.7;
    color: white;
}

.edit-btn {
    padding: 6px 16px;
    border-radius: 8px;
    border: none;
    cursor: pointer;
    font-weight: 600;
    background: #e0f1e4;
    color: #155e37;
    transition: background 0.2s, transform 0.1s;
}

.task-row-div {
    display: flex;
    justify-content: space-between;
    gap: 10px;
}
</style>
