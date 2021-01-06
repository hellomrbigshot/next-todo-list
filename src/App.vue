<template>
  <div id="app" class="todoapp">
    <h1 class="header">todos</h1>
    <input
      v-model="newTodo"
      class="new-todo"
      autofocus
      autocomplete="off"
      placeholder="What needs to be done?"
      @keyup.enter="addTodo"
    />
    <!-- <TodoList/> -->
    <section v-show="todos.length" class="main">
      <input
        v-model="allDone"
        id="toggle-all"
        class="toggle-all"
        type="checkbox"
      />
      <label for="toggle-all"></label>
      <ul class="todo-list">
        <li
          v-for="todo in filteredTodos"
          :key="todo.id"
          class="todo"
          :class="{ completed: todo.completed, editing: todo == editedTodo }"
        >
          <div class="view">
            <input v-model="todo.completed" class="toggle" type="checkbox"/>
            <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
            <button class="destroy" @click="removeTodo(todo)"></button>
          </div>
          <input
            v-model="todo.title"
            v-todo-focus="todo === editedTodo"
            class="edit"
            type="text"
            @blur="doneEdit(todo)"
            @keyup.enter="doneEdit(todo)"
            @keyup.esc="cancelEdit(todo)"
          />
        </li>
      </ul>
    </section>
    <footer v-show="todos.length" class="footer">
      <span class="todo-count">
        <strong>{{ remaining }}</strong> {{ remaining > 1 ? 'items' : 'item' }} left
      </span>
      <ul class="filters">
        <li>
          <a href="#/all" :class="{ selected: visibility == 'all' }">All</a>
        </li>
        <li>
          <a href="#/active" :class="{ selected: visibility == 'active' }">Active</a>
        </li>
        <li>
          <a
            href="#/completed"
            :class="{ selected: visibility == 'completed' }"
          >Completed</a>
        </li>
      </ul>
      <button
        class="clear-completed"
        v-show="todos.length > remaining"
        @click="removeCompleted"
      >
        Clear completed
      </button>
    </footer>
  </div>
</template>
<script lang="ts">
import { defineComponent, ref, computed, watch, onMounted, onUnmounted, Ref } from 'vue'

interface ITodo {
  completed: boolean,
  id: number,
  title: string
}
interface ITodoStorage {
  fetch: () => ITodo[],
  save: (todos: ITodo[]) => boolean | undefined,
  uid: number
}
interface IFilters {
  [x: string]: any,
  all: (todos: ITodo[]) => ITodo[],
  active: (todos: ITodo[]) => ITodo[],
  completed: (todos: ITodo[]) => ITodo[]
}

const STORAGE_KEY: string = "todos-vuejs-3.0"
const todoStorage: ITodoStorage = {
  fetch () {
    const todos: ITodo[] = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
    todos.forEach((todo, index) => {
      todo.id = index
    })
    todoStorage.uid = todos.length
    return todos as ITodo[]
  },
  save (todos) {
    if (!Array.isArray(todos)) return false
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
  },
  uid: 0
}

const filters: IFilters = {
  all (todos) {
    return todos
  },
  active (todos) {
    return todos.filter(todo => {
      return !todo.completed
    });
  },
  completed (todos) {
    return todos.filter(todo => {
      return todo.completed
    })
  }
}

const Component = defineComponent({
  name: 'App',
  setup () {
    const todos = ref(todoStorage.fetch())
    const visibility = ref('all')
    const newTodo = ref('')
    const editedTodo: Ref<null | ITodo> = ref(null)
    const beforeEditCache = ref('')
    const filteredTodos = computed(() => filters[visibility.value](todos.value))
    const remaining = computed(() => filters.active(todos.value).length)
    const allDone = computed({
      get: () => remaining.value === 0,
      set: value => {
        todos.value.forEach(todo => {
          todo.completed = value
        })
      }
    })
    watch(todos, (todos) => {
      todoStorage.save(todos)
    }, { deep: true })
    onMounted(() => {
      window.addEventListener('hashchange', onHashChange)
    })
    onUnmounted(() => {
      window.removeEventListener('hashchange', onHashChange)
    })
    const addTodo = () => {
      const value = newTodo.value && newTodo.value.trim()
      if (!value) {
        return
      }
      todos.value.push({
        id: todoStorage.uid++,
        title: value,
        completed: false
      })
      newTodo.value= ''
    }
    const removeTodo = (todo: ITodo) => {
      todos.value.splice(todos.value.indexOf(todo), 1)
    }
    const editTodo = (todo: ITodo) => {
      beforeEditCache.value = todo.title
      editedTodo.value = todo
    }
    const doneEdit = (todo: ITodo) => {
      if (!editedTodo.value) {
        return
      }
      editedTodo.value = null
      todo.title = todo.title.trim()
      if (!todo.title) {
        removeTodo(todo)
      }
    }
    const cancelEdit = (todo: ITodo) => {
      editedTodo.value = null
      todo.title = beforeEditCache.value
    }
    const removeCompleted = () => {
      todos.value = filters.active(todos.value)
    }
    const onHashChange = () => {
        const visible: string = window.location.hash.replace(/#\/?/, '')
        if (filters[visible]) {
          visibility.value = visible
        } else {
          window.location.hash = ''
          visibility.value = ''
        }
      }
    return {
      todos,
      newTodo,
      editedTodo,
      visibility,
      remaining,
      allDone,
      filteredTodos,
      addTodo,
      removeTodo,
      editTodo,
      doneEdit,
      cancelEdit,
      removeCompleted
    }
  },
  directives: {
    'todo-focus' (el, binding) {
      if (binding.value) {
        el.focus()
      }
    }
  }
})

export default Component
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  /* margin-top: 60px; */
}
</style>
