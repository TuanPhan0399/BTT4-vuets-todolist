<template>
  <div class="hello">
    <header class="header">
      <h1>My To Do list of Tuan Phan</h1>
    </header>
    <div class="wrapper">
      <div class="task-input">
        <i
          ref="iElement"
          v-show="todos.length"
          class="fa-solid fa-angle-down"
          v-bind:class="{ tickAll: countFun === 0 }"
          @click="toggleAll()"
        >
        </i>
        <input
          type="text"
          v-model="newTodo"
          @keyup.enter="addTodo()"
          placeholder="What needs to be done?"
        />
      </div>
      <ul
        v-for="(todo, index) in filtersTodos"
        :key="todo.idU"
        class="task-box"
      >
        <li class="task">
          <div>
            <input type="checkbox" v-model="todo.status" />
            <input
              ref="valueInput"
              @dblclick="editTask(index)"
              type="text"
              :value="todo.name"
              :class="{
                checked: todo.status === true,
              }"
              @blur="doneEdit(todo, index)"
              @keyup.enter="doneEdit(todo, index)"
              readOnly
            />
          </div>
          <div class="task-close">
            <i @click="deleteTask(index)" class="fa-solid fa-xmark"></i>
          </div>
        </li>
      </ul>
      <div v-show="todos.length" class="controls">
        <span>
          <strong class="count">{{ countFun }}</strong> items left
        </span>
        <div class="filters">
          <span
            id="all"
            :class="{ active: visibility === 'all' }"
            @click="setFilterType('all')"
          >
            All
          </span>
          <span
            id="pending"
            :class="{ active: visibility === 'pending' }"
            @click="setFilterType('pending')"
          >
            Active
          </span>
          <span
            id="completed"
            :class="{ active: visibility === 'completed' }"
            @click="setFilterType('completed')"
          >
            Completed
          </span>
        </div>
        <button
          class="clearBtn"
          :class="{ clearAll: todos.length <= countFun }"
          @click="clearTask()"
        >
          Clear Completed
        </button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
interface Todo {
  name: string;
  status: boolean;
  idU: number;
}
const STORAGE_KEY = "todo-list";

export default defineComponent({
  name: "ToDo",

  data() {
    return {
      newTodo: "" as string,
      visibility: "all" as string,
      count: 0 as number,
      todos: JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]") as Todo[],
    };
  },
  watch: {
    todos: {
      handler(todos) {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
      },
      deep: true,
    },
  },
  computed: {
    countFun: function () {
      let countF = this.count as number;
      const todos2 = this.todos.filter((todo) => todo.status === false);
      countF = todos2.length;
      return countF;
    },
    filtersTodos: function () {
      let todos4 = this.todos.filter((todo) => {
        switch (this.visibility) {
          case "pending":
            return todo.status === false;
          case "completed":
            return todo.status;
          default:
            return true;
        }
      }) as Todo[];
      return todos4;
    },
  },
  methods: {
    toggleAll() {
      const todosTrue: Todo[] = this.todos.filter((todo) => {
        return todo.status === true;
      });
      if (todosTrue.length) {
        this.todos.forEach((item) => {
          item.status = false;
        });
      } else {
        this.todos.forEach((item) => {
          item.status = true;
        });
      }
    },

    addTodo() {
      if (this.newTodo.length === 0) return;
      this.todos.push({
        name: this.newTodo,
        status: false,
        idU: Date.now(),
      });
      this.newTodo = "";
    },

    deleteTask(index: number) {
      this.todos.splice(index, 1);
    },

    editTask(index: number) {
      const array = this.$refs.valueInput as [];
      const input = array[index] as HTMLInputElement;
      const task = input.parentElement as HTMLDivElement;
      const task2 = task.parentElement as HTMLLIElement;
      const taskClose = task2.lastElementChild as HTMLDivElement;
      const inputCheck = task.firstElementChild as HTMLInputElement;
      taskClose.style.opacity = "0";
      inputCheck.style.opacity = "0";
      input.style.border = "1px solid #999";
      input.readOnly = false;
      input.setSelectionRange(input.value.length, input.value.length);
      if (input.classList.contains("checked")) {
        input.classList.remove("checked");
      }
    },

    doneEdit(todo: Todo, index: number) {
      if (!this.editTask) return;
      const array = this.$refs.valueInput as [];
      const input = array[index] as HTMLInputElement;
      const task = input.parentElement as HTMLDivElement;
      const task2 = task.parentElement as HTMLLIElement;
      const taskClose = task2.lastElementChild as HTMLDivElement;
      const inputCheck = task.firstElementChild as HTMLInputElement;
      taskClose.style.opacity = "1";
      inputCheck.style.opacity = "1";
      input.style.border = "none";
      input.readOnly = true;
      todo.name = input.value.trim();
      if (!todo.name) {
        this.todos.splice(this.todos.indexOf(todo), 1);
      }
      if (todo.status === true) {
        input.classList.add("checked");
      }
    },

    setFilterType(visibility: string) {
      this.visibility = visibility;
    },

    clearTask() {
      const todos3 = this.todos.filter((todo) => todo.status !== true);
      return (this.todos = todos3);
    },
  },
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap");

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background-image: linear-gradient(120deg, #8baaa6, #49a196);
  font-family: "Poppins", sans-serif;
  min-height: 150vh;
  overflow: hidden;
}

.header {
  font-size: 1.5rem;
}

.header h1 {
  min-height: 20vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

.wrapper {
  max-width: 500px;
  margin: 0 auto;
  background: #fff;
  border-radius: 7px;
  padding-top: 28px;
  padding-bottom: 22px;
}

.task-input {
  height: 52px;
  padding: 0 25px;
  position: relative;
}
.task-input i {
  position: absolute;
  top: 50%;
  transform: translate(20px, -50%);
  opacity: 0.5;
  cursor: pointer;
  font-size: 2.9rem;
  left: 4%;
}
.task-input i.tickAll {
  opacity: 1;
}

.task-input input {
  height: 100%;
  width: 100%;
  font-size: 18px;
  border-radius: 5px;
  border: 1px solid #999;
  padding: 0 20px 0 66px;
  outline: none;
}

.task-input input::placeholder {
  color: #bfbfbf;
}

.controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 18px 25px;
}
.filters span {
  cursor: pointer;
  margin: 0 8px;
  font-size: 17px;
}

.filters span:first-child {
  margin-left: 0;
}

.filters span.active {
  color: #49a196;
}

.clearAll {
  opacity: 0;
}

.controls .clearBtn {
  outline: none;
  border: none;
  color: #fff;
  border-radius: 4px;
  padding: 7px 13px;
  background: #49a196;
  cursor: pointer;
  font-size: 13px;
}

.task-box {
  margin: 20px 25px;
}

.task-box .task {
  list-style: none;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 17px;
  padding-bottom: 16px;
  margin-bottom: 18px;
  border-bottom: 1px solid #ccc;
}

.task > div {
  position: relative;
}

.task div > span input {
  width: 25rem;
  height: 2.3rem;
  position: absolute;
  top: -14px;
  font-size: 20px;
  padding-left: 10px;
  opacity: 0.5;
  left: 52%;
}

.task-box .task:last-child {
  margin-bottom: 0;
}

.task div {
  display: flex;
  align-items: center;
}

.task div input {
  margin-left: 20px;
}
.task div input[type="checkbox"] {
  height: 2rem;
  width: 2rem;
}

.task div input[type="text"] {
  height: 2.9rem;
  width: 23.5rem;
  font-size: 1.8rem;
  border: none;
  background: #fff;
  outline: none;
}

.task-close {
  cursor: pointer;
}

.task-close i {
  position: absolute;
  opacity: 0;
  padding: 5px 0;
  margin-right: 7px;
  font-size: 1.9rem;
  left: -1.4rem;
}

.task-box .task:hover i {
  opacity: 1;
}
.checked {
  text-decoration: line-through;
  opacity: 0.5;
}
</style>
