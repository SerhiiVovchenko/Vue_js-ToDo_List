<script>
import todos from "./data/todos";
import StatusFilter from "./components/StatusFilter.vue";
import TodoItem from "./components/TodoItem.vue";

export default {
  components: {
    StatusFilter,
    TodoItem,
  },
  data() {
    let todos = [];
    const jsonData = localStorage.getItem("todos") || "[]";

    try {
      todos = JSON.parse(jsonData);
    } catch (error) {}

    return {
      todos,
      title: "",
      status: "all",
    };
  },
  computed: {
    activeTodos() {
      return this.todos.filter((todo) => !todo.completed);
    },
    completedTodos() {
      return this.todos.filter((todo) => todo.completed);
    },
    visibleTodos() {
      switch (this.status) {
        case "active":
          return this.activeTodos;

        case "completed":
          return this.completedTodos;

        default:
          return this.todos;
      }
    },
  },
  watch: {
    todos: {
      deep: true,
      handler() {
        localStorage.setItem("todos", JSON.stringify(this.todos));
      },
    },
  },
  methods: {
    handleSubmit() {
      this.todos.push({
        id: Date.now(),
        title: this.title,
        completed: false,
      });

      this.title = "";
    },
    togglerCompAllTodos() {
      this.todos.map((todo) => (todo.completed = false));
    },
    hendleDeleteAll() {
      this.todos = this.todos.filter((todo) => todo.completed !== true);
    },
  },
};
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos</h1>

    <div class="todoapp__content">
      <header class="todoapp__header">
        <button
          class="todoapp__toggle-all"
          v-bind="{ disabled: activeTodos.length !== 0 }"
          v-bind:class="{ active: activeTodos.length === 0 }"
          @click="togglerCompAllTodos"
        ></button>

        <form @submit.prevent="handleSubmit">
          <input
            type="text"
            class="todoapp__new-todo"
            placeholder="What needs to be done?"
            v-model="title"
          />
        </form>
      </header>

      <TransitionGroup name="listAn" tag="section" class="todoapp__main">
        <TodoItem
          v-for="(todo, index) of visibleTodos"
          v-bind:key="todo.id"
          v-bind:todo="todo"
          @update="Object.assign(todo, $event)"
          @delete="todos.splice(todos.indexOf(todo), 1)"
        />
      </TransitionGroup>

      <footer class="todoapp__footer" v-if="todos.length > 0">
        <span class="todo-count"> {{ activeTodos.length }} items left </span>

        <StatusFilter v-model="status" />

        <button
          class="todoapp__clear-completed"
          v-if="completedTodos.length > 0"
          @click="hendleDeleteAll"
        >
          Clear completed
        </button>
      </footer>
    </div>
  </div>
</template>

<style>
.listAn-enter-active,
.listAn-leave-active {
  max-height: 60px;
  transition: all 0.5s ease;
}

.listAn-enter-from,
.listAn-leave-to {
  opacity: 0;
  max-height: 0;
  transform: scaleY(0);
}
</style>
