<script setup>
import { ref, onMounted, computed, watch } from "vue";
import profile from "./assets/gato.jpg";
const todos = ref([]);
const name = ref("");

const input_content = ref("");
const input_category = ref(null);

const todos_ascending = computed(() =>
  todos.value.sort((a, b) => {
    return b.createdAt - a.createdAt;
  })
);

const addTodo = () => {
  if (input_content.value.trim() === "" || input_category.value === null) {
    return;
  }

  todos.value.push({
    content: input_content.value,
    category: input_category.value,
    done: false,
    createdAt: new Date().getTime(),
    editing: false,
  });

  input_content.value = "";
  input_category.value = null;
};

watch(
  todos,
  (newVal) => {
    localStorage.setItem("todos", JSON.stringify(newVal));
  },
  {
    deep: true,
  }
);

const toggleEditing = (todo) => {
  todo.editing = true;
};

const saveEdit = (todo, event) => {
  todo.content = event.target.value.trim();
  todo.editing = false;
};

const deleteTodo = (index) => {
  todos.value.splice(index, 1);
};

watch(name, (newVal) => {
  localStorage.setItem("name", newVal);
});

onMounted(() => {
  name.value = localStorage.getItem("name") || "";
  const storedTodos = localStorage.getItem("todos");
  todos.value = storedTodos ? JSON.parse(storedTodos) : [];
});
</script>

<template>
  <main class="container mx-auto h-auto bg-slate-900 min-h-screen max-w-lg">
    <section class="p-5">
      <div class="flex justify-between items-center">
        <div class="flex items-center">
          <h1 class="text-3xl text-white font-semibold">Wassup,</h1>
          <input
            type="text"
            class="focus:outline-none border-transparent cursor-pointer focus:border-t-white focus:ring-0 w-1/2 placeholder:text-2xl border-none bg-transparent font-medium text-white max-w-xs md:max-w-full text-3xl transition"
            placeholder="Name here..."
            v-model="name"
          />
        </div>
        <img
          :src="profile"
          alt="gato"
          class="w-[35px] rounded-full active:scale-[.85] cursor-pointer transition duration-200"
        />
      </div>

      <h2 class="uppercase text-lg font-semibold text-slate-500">
        Create activities today..
      </h2>
      <form @submit.prevent="addTodo">
        <h4 class="text-base text-slate-500 mb-2 font-semibold">
          Got something in mind?
        </h4>
        <input
          type="text"
          class="mb-3 w-full focus:outline-none text-white border-none shadow bg-slate-800 rounded-md"
          placeholder="Today daily.."
          v-model="input_content"
        />
        <h4 class="text-base text-slate-500 mb-2 font-semibold">
          Select Category
        </h4>
        <div>
          <div class="grid grid-cols-2 gap-5">
            <label
              class="px-4 py-4 rounded-md active:scale-[.85] cursor-pointer transition duration-200 max-w-xs bg-slate-800"
            >
              <input
                id="green-radio"
                type="radio"
                value="Personal"
                name="colored-radio"
                v-model="input_category"
                class="w-4 h-4 text-green-600 cursor-pointer flex mx-auto bg-gray-100 border-gray-300 focus:ring-green-500 dark:focus:ring-green-600 dark:ring-offset-gray-800 focus:ring-2 mb-3 dark:bg-gray-700 dark:border-gray-600"
              />
              <div
                class="text-lg font-medium text-center"
                :class="
                  input_category === 'Personal'
                    ? 'text-green-600'
                    : 'text-slate-500'
                "
              >
                Personal
              </div>
            </label>

            <label
              class="px-4 py-4 rounded-md active:scale-[.85] cursor-pointer transition duration-200 max-w-xs bg-slate-800"
            >
              <input
                id="purple-radio"
                type="radio"
                value="Business"
                name="colored-radio"
                v-model="input_category"
                class="w-4 h-4 text-purple-600 cursor-pointer flex mx-auto bg-gray-100 border-gray-300 focus:ring-purple-500 dark:focus:ring-purple-600 dark:ring-offset-gray-800 focus:ring-2 mb-3 dark:bg-gray-700 dark:border-gray-600"
              />
              <div
                class="text-lg font-medium text-center"
                :class="
                  input_category === 'Business'
                    ? 'text-purple-600'
                    : 'text-slate-500'
                "
              >
                Business
              </div>
            </label>
          </div>
          <input
            type="submit"
            class="bg-blue-500 w-full mt-5 p-2 rounded text-white hover:bg-blue-800 active:scale-[.85] cursor-pointer transition duration-200"
            value="Add Task"
          />
        </div>
      </form>

      <section class="mt-8">
        <h3 class="text-base text-slate-500 uppercase mb-3">todolist</h3>
        <div class="grid grid-cols-1 gap-3">
          <!-- list todos -->
          <div
            v-for="(todo, index) in todos_ascending"
            :key="todo.createdAt"
            class="px-5 py-3 rounded bg-slate-800 cursor-pointer shadow flex items-center justify-between"
          >
            <!-- checkbox -->
            <div class="flex items-center">
              <input
                type="checkbox"
                v-model="todo.done"
                :class="
                  todo.category === 'Business'
                    ? 'text-purple-600 focus focus:ring-purple-600 dark:focus:ring-purple-600'
                    : 'text-green-600 focus:ring-green-500 dark:focus:ring-green-600'
                "
                class="w-4 h-4 cursor-pointer bg-gray-100 border-gray-300 rounded dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600"
              />

              <!-- edit mode -->
              <div class="ml-3 text-lg font-medium">
                <input
                  v-if="todo.editing"
                  type="text"
                  :value="todo.content"
                  @blur="saveEdit(todo, $event)"
                  @keydown.enter="saveEdit(todo, $event)"
                  class="w-full focus:outline-none border-b rounded border-gray-300"
                />

                <!-- content -->
                <span
                  v-else
                  :class="
                    todo.done
                      ? 'line-through text-gray-400'
                      : 'text-white cursor-pointer'
                  "
                  @click="toggleEditing(todo)"
                >
                  {{ todo.content }}
                </span>
              </div>

              <!-- Delete button -->
            </div>
            <button
              @click="deleteTodo(index)"
              class="bg-red-500 text-white px-3 py-1 rounded active:scale-[.85] cursor-pointer hover:bg-red-700 transition duration-150"
            >
              Delete
            </button>
          </div>
        </div>
      </section>
    </section>
  </main>
</template>
