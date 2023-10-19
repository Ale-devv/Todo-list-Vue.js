<template>
    <div class="h-full pb-20 bg-slate-800 text-slate-200">
      <div class="flex flex-col space-y-5">
        <img class="img-responsive" src="../assets/lista.png" alt="TodoTag">
        <div class="flex flex-col items-center pt-2 mx-auto space-y-2 w-96">
            <label v-if="inputValue" class="text-2xl" for="input">New task: {{ inputValue }}</label>
            <span v-if="disabled" class="text-red-800">Input field cannot be empty!</span>
            <input @keyup.enter="addNewTask()" v-model.trim="inputValue" type="text" id="input" class="form-input" :class="{ shake: disabled }" placeholder="Adicionar tarefa...">
            <button @click="addNewTask()" class="p-2 text-2xl transition bg-orange-800 rounded-sm hover:bg-orange-900 active:bg-orange-950">Nova tarefa</button>
        </div>

        <div v-for="item in tasksArray" :key="item.id" class="flex flex-col items-start pt-2 mx-auto w-72">

            <div class="flex items-center space-x-5" v-if="!item.isEditing">
                <input @click="item.isDone = !item.isDone"  :checked="item.isDone" type="checkbox" :id="item.id" class="w-7 h-7 checked:ring-black">
                <label :for="item.id" class="text-2xl"  :class="item.isDone ? 'line-through italic text-gray-700' : ''">{{ item.title }}</label>
            </div>


            <input v-else v-model.trim="newTitle" type="text" class="p-4 text-2xl text-black rounded-sm focus:outline-none" @keyup.enter="changeTitle(item.id,item.isEditing)" :class="{ shake: disabled }">
            <div class="flex pt-5">
                <button v-if="!item.isEditing" @click="item.isEditing = !item.isEditing" class="p-1 mr-5 text-xl transition bg-green-800 rounded-sm hover:bg-green-900 active:bg-green-950">Edit</button>
                <button v-else @click="changeTitle(item.id,item.isEditing)" class="p-1 mr-5 text-xl transition bg-green-800 rounded-sm hover:bg-green-900 active:bg-green-950">Confirm</button>
                <button v-if="!item.isEditing" @click="deleteTask(item.id)" class="p-1 text-xl transition bg-red-800 rounded-sm hover:bg-red-900 active:bg-red-950"> Delete</button>
                <button v-else @click="item.isEditing = !item.isEditing"  class="p-1 text-xl transition bg-red-800 rounded-sm hover:bg-red-900 active:bg-red-950"> Cancel</button>
            </div>
        </div>

        <div v-if="tasksArray.length == 0" class="flex flex-col items-center justify-center mx-auto">
            <p class="text-2xl"> Sem novas tarefas!  😭</p>
            <img class="max-w-[40rem]" src="../assets/sadcat.webp" alt="sad kitten">
        </div>


        <div v-else-if="tasksArray.length !== 0 && allTasksDone" class="flex flex-col items-center justify-center mx-auto">
            <p class="text-2xl">Todas as tarefas estão prontas! 🥳</p>
            <img class="max-w-[20rem]" src="../assets/coolcat.webp" alt="sad kitten">
        </div>

        <div v-else-if="tasksArray.length !== 0 && Toomuchtasks" class="flex flex-col items-center justify-center mx-auto">
            <p class="text-2xl">Muitas tarefas! 🥳</p>
            <img class="max-w-[20rem]" src="../assets/coolcat.webp" alt="sad kitten">
        </div>

        <div v-else class="flex flex-col items-center justify-center mx-auto">
            <p class="text-2xl text-gray-700">Nem todas as tareas estão prontas</p>
        </div>

       



      </div>
    </div>
  </template>

  
  
  <script setup>
  import {ref, onMounted, computed} from "vue"

  //get items from LocalStorage
  onMounted(() => {
     const items = localStorage.getItem('tasks')
     if(items) {
        const parsedItems = JSON.parse(items);
        tasksArray.value.push(...parsedItems)
        console.log(tasksArray.value)
     }
  })   
  
  const inputValue = ref('')
  const tasksArray = ref([]);
  
  function addNewTask() {
    if(inputValue.value === '') {
        warnDisabled()
    } else {
        const task = {
            id: crypto.randomUUID(),
            isDone: false,
            isEditing: false,
            title: inputValue.value,
        }
        tasksArray.value.push(task)
        inputValue.value = '';
    }

  }
  
  function deleteTask(id) {
    tasksArray.value.forEach((item, index) => {
      if(item.id === id) {
        tasksArray.value.splice(index, 1)
      }
    })
  }
  
  const newTitle = ref('')
  function changeTitle(id, isEditing) {
    if(newTitle.value === '') {
        warnDisabled()
    } else {
      tasksArray.value.forEach((item) => {
        if(item.id === id) {
            item.title = newTitle.value;
            newTitle.value = '';
            item.isEditing = !isEditing;
        }
    })
    }
  }

  //set items to LocalStorage right before the page refresh
  window.addEventListener("beforeunload", () => {
    localStorage.setItem("tasks", JSON.stringify(tasksArray.value));
  });
  
  //animation if input is empty
  const disabled = ref(false)

  function warnDisabled() {
    disabled.value = true
    setTimeout(() => {
        disabled.value = false
    }, 1500)
  }

  //is all tasks done?
  const allTasksDone = computed(() => {
    return tasksArray.value.every((item) => {
        return item.isDone === true;
    })
  })
  </script>

  <style scoped>
  .shake {
    animation: shake 0.82s cubic-bezier(0.36, 0.07, 0.19, 0.97) both;
    transform: translate3d(0, 0, 0);
    }

    @keyframes shake {
    10%,
    90% {
        transform: translate3d(-1px, 0, 0);
    }

    20%,
    80% {
        transform: translate3d(2px, 0, 0);
    }

    30%,
    50%,
    70% {
        transform: translate3d(-4px, 0, 0);
    }

    40%,
    60% {
        transform: translate3d(4px, 0, 0);
    }
    }

img {
  max-width: 150px;
  margin: auto;
  background: button;
}
  </style>