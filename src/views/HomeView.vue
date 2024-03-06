<script setup lang="ts">
import axios from 'axios'
import { ref, computed } from 'vue'
import type { Post, User } from '@/types'

const allPosts = ref<Post[] | null>(null)

axios.get('http://jsonplaceholder.typicode.com/posts').then((res) => {
  allPosts.value = res.data
})

const users = ref<User[] | null>(null)

axios.get('http://jsonplaceholder.typicode.com/users').then((res) => {
  users.value = res.data
})

const searchText = ref('')

function searchTextIncludes(text: string, query: string) {
  return text.toLowerCase().includes(query.toLowerCase())
}

const filteredPosts = computed<Post[] | null | undefined>(() => {
  if (!searchText.value) {
    return allPosts.value
  }
  // вычислим id авторов, а затем отберём посты по id авторов
  const filteredUsersId = users.value
    ?.filter((user: User) => searchTextIncludes(user.name, searchText.value))
    .map((user) => user.id)

  return allPosts.value?.filter((post: Post) => filteredUsersId?.includes(post.userId))
})
</script>

<template>
  <div class="container-fluid background">
    <div class="container pt-5">
      <div
        class="input-group mb-4 mx-auto"
        style="max-width: 25vw; max-width: 360px; min-width: 260px"
      >
        <div class="input-group-prepend">
          <span class="input-group-text">@</span>
        </div>
        <input
          type="text"
          class="form-control"
          placeholder="Filter by author"
          aria-label="Filter by author"
          v-model.trim="searchText"
        />
      </div>
      <div class="card-columns">
        <div class="card" v-for="post in filteredPosts" :key="post.id">
          <div class="card-body">
            <h4 class="card-title text-primary">{{ post.title }}</h4>
            {{ post.body }}
            <p class="mt-3 mb-0">
              <small class="text-muted">{{
                users?.find((user: User) => user.id == post.userId)?.name
              }}</small>
            </p>
          </div>
        </div>
      </div>
      <div v-if="!filteredPosts?.length" class="card mx-auto w-50">
        <div class="card-body">Посты не найдены</div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.background {
  background-color: #eaeff3;
}

@media screen and (max-width: 992px) {
  .card-columns {
    column-count: 2;
  }
}

@media screen and (max-width: 576px) {
  .card-columns {
    column-count: 1;
  }
}
</style>
