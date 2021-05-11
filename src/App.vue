<template>
  <div class="container">
    <h1 class="text-light mb-4">待辦事項列表</h1>
    <div class="input-group mb-3">
      <div class="input-group-prepend">
        <span class="input-group-text" id="basic-addon1">事項</span>
      </div>
      <input
        type="text"
        class="form-control"
        placeholder="輸入代辦事項"
        v-model="newTodo"
        @keyup.enter="addTodo"
      />
      <div class="input-group-append">
        <button class="btn btn-primary" type="button" @click="addTodo">
          新增事項
        </button>
      </div>
    </div>
    <div class="card text-center">
      <div class="card-header">
        <ul class="nav nav-tabs card-header-tabs">
          <li class="nav-item">
            <a
              class="nav-link"
              href="#"
              :class="{ active: visibility == 'all' }"
              @click="visibility = 'all'"
              >全部</a
            >
          </li>
          <li class="nav-item">
            <a
              class="nav-link"
              href="#"
              :class="{ active: visibility == 'active' }"
              @click="visibility = 'active'"
              >進行中</a
            >
          </li>
          <li class="nav-item">
            <a
              class="nav-link"
              href="#"
              :class="{ active: visibility == 'completed' }"
              @click="visibility = 'completed'"
              >已完成</a
            >
          </li>
        </ul>
      </div>
      <ul class="list-group list-group-flush text-left">
        <li
          class="list-group-item"
          v-for="(item, key) in filteredTodos"
          :key="key"
          @dblclick="editTodo(item)"
        >
          <div class="d-flex" v-if="item.id !== cacheTodo.id">
            <div class="d-flex align-items-center">
              <button
                class="btn btn-primary btn-sm me-3"
                @click="setCompleted(item)"
                :id="item.id"
                v-if="!item.completed"
              >
                確認完成
              </button>
              <button
                class="btn btn-danger btn-sm me-3"
                @click="setCompleted(item)"
                :id="item.id"
                v-if="item.completed"
              >
                取消完成
              </button>
              <div :for="item.id" :class="{ completed: item.completed }">
                {{ item.title }}
              </div>
            </div>
            <button
              type="button"
              class="btn-close ms-auto"
              aria-label="Close"
              @click="removeTodo(item)"
            ></button>
          </div>
          <input
            type="text"
            class="form-control"
            v-if="item.id === cacheTodo.id"
            v-model="cacheTitle"
            @keyup.esc="cancelEdit()"
            @keyup.enter="doneEdit(item)"
          />
        </li>
      </ul>
      <div class="card-footer d-flex justify-content-between">
        <span>還有 {{ undone }} 筆事項尚未完成</span>
        <a href="#" @click.prevent="clearAll">清除所有事項</a>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      newTodo: "",
      todos: [],
      cacheTodo: {},
      cacheTitle: "",
      visibility: "all",
      data: JSON.parse(localStorage.getItem("listData")) || [],
    };
  },
  methods: {
    addTodo() {
      let vm = this;
      let value = vm.newTodo.trim();
      let timestamp = Math.floor(Date.now());
      if (!value) {
        return;
      }
      let todo = {
        id: timestamp,
        title: value,
        completed: false,
      };
      vm.data.push(todo);
      localStorage.setItem("listData", JSON.stringify(vm.data));
      vm.newTodo = "";
    },
    removeTodo(todo) {
      let newTodos = "";
      let vm = this;
      vm.data.forEach(function (item, key) {
        if (todo.id === item.id) {
          newTodos = key;
        }
      });
      vm.data.splice(newTodos, 1);
      localStorage.setItem("listData", JSON.stringify(vm.data));
    },
    setCompleted(item) {
      let vm = this;
      vm.data.forEach(function (items) {
        if (item.id === items.id) {
          item.completed = !item.completed;
        }
      });
      localStorage.setItem("listData", JSON.stringify(vm.data));
    },
    editTodo(item) {
      let vm = this;
      vm.cacheTodo = item;
      vm.cacheTitle = item.title;
    },
    cancelEdit() {
      let vm = this;
      vm.cacheTodo = {};
    },
    doneEdit(item) {
      let vm = this;
      item.title = this.cacheTitle;
      localStorage.setItem("listData", JSON.stringify(vm.data));
      vm.cacheTitle = "";
      vm.cacheTodo = {};
    },
    clearAll() {
      let vm = this;
      vm.data = [];
      localStorage.setItem("listData", JSON.stringify(vm.data));
    },
  },
  computed: {
    filteredTodos() {
      if (this.visibility == "all") {
        return this.data;
      } else if (this.visibility === "active") {
        let newTodos = [];
        this.data.forEach(function (item) {
          if (!item.completed) {
            newTodos.push(item);
          }
        });
        return newTodos;
      } else if (this.visibility === "completed") {
        let newTodos = [];
        this.data.forEach(function (item) {
          if (item.completed) {
            newTodos.push(item);
          }
        });
        return newTodos;
      }
    },
    undone: function () {
      let allDoneTodos = [];
      this.data.forEach(function (item) {
        if (item.completed === false) {
          allDoneTodos.push(item);
        }
      });
      return allDoneTodos.length;
    },
  },
};
</script>

<style lang="scss">
@import "bootstrap";
.container {
  margin-top: 40px;
}
.completed {
  text-decoration: line-through;
  opacity: 0.5;
}
</style>
