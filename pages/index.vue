<template>
  <div class="flex flex-col items-center p-8">
    <h1 class="text-4xl mb-4">Todo</h1>
    <form @submit.prevent="onSubmit" class="flex flex-col items-center">
      <UInput v-model="todo" />
      <UButton
        type="submit"
        loading-icon="i-heroicons-arrow-up-circle-solid"
        class="mt-4"
        >Add Todo</UButton
      >
    </form>
  </div>
  <div class="flex flex-col gap-4">
    <div v-for="todo in todos" :key="todo.id" class="w-1/2 m-auto">
      <UCard>
        <template #header>
          <div class="flex justify-between items-center">
            <div class="text-blue-600">{{ todo.content }}</div>
            <UButton
              icon="i-heroicons-trash"
              square
              color="white"
              size="2xs"
              @click="onClickDelete(todo.id)"
            />
          </div>
        </template>

        <div class="text-xs text-gray-400">
          {{ todo.id }}
        </div>
      </UCard>
    </div>
  </div>
</template>

<script setup lang="ts">
const todo = ref("");
const { $Amplify } = useNuxtApp();
import * as query from "@/src/graphql/queries";
import * as mutations from "@/src/graphql/mutations";

const { data: todos, refresh } = useAsyncData(async () => {
  const result = await $Amplify.GraphQL.client.graphql({
    query: query.listTodos,
  });
  console.log("result", result?.data);
  return result.data.listTodos.items;
});

const onSubmit = async () => {
  if (!todo.value) return;
  const { data, errors } = await $Amplify.GraphQL.client.graphql({
    query: mutations.createTodo,
    variables: {
      input: { content: todo.value },
    },
  });
  console.log("result", data);
  todo.value = "";
  refresh();
};

const onClickDelete = async (id: string) => {
  const { data, errors } = await $Amplify.GraphQL.client.graphql({
    query: mutations.deleteTodo,
    variables: {
      input: { id },
    },
  });
  refresh();
};
</script>

<style scoped></style>
