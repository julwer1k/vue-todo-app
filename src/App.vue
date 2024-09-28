<script>
import { createTodo, deleteTodo, getTodos, updateTodo } from '@/api/todos.js';
import Message from '@/components/Message.vue';
import TodoItem from '@/components/TodoItem.vue';
import StatusFilter from './components/StatusFilter.vue';
import todos from './data/todos';

export default {
	components: {
		Message,
		TodoItem,
		Filter: StatusFilter,
	},
	data() {
		return {
			todos: [],
			title: '',
			status: 'all',
			errorMessage: '',
		};
	},
	computed: {
		activeTodos() {
			return this.todos.filter(todo => !todo.completed);
		},
		completedTodos() {
			return this.todos.filter(todo => todo.completed);
		},
		
		visibleTodos() {
			switch (this.status) {
				case 'active':
					return this.activeTodos;
				case 'completed':
					return this.completedTodos;
				default:
					return this.todos;
			}
		},
	},
	/*watch: {
	 todos: {
	 deep: true,
	 handler() {
	 localStorage.setItem('todos', JSON.stringify(this.todos));
	 },
	 },
	 },*/
	
	mounted() {
		getTodos()
			.then(({ data }) => this.todos = data)
			.catch(() => {
				this.errorMessage = 'Unable to load todos';
			});
	},
	methods: {
		handleSubmit() {
			createTodo(this.title).then(({ data }) => {
				this.todos = [...this.todos, data];
				this.title = '';
			});
		},
		updateTodo({ id, title, completed }) {
			updateTodo({ id, title, completed }).then(({ data }) => {
				this.todos = this.todos.map(
					todo => todo.id !== id ? todo : data,
				);
			});
		},
		deleteTodo(todoId) {
			deleteTodo(todoId).then(() => {
				this.todos = this.todos.filter(todo => todo.id !== todoId);
			});
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
					type="button"
					class="todoapp__toggle-all"
					:class="{active: activeTodos.length === 0}"
					data-cy="ToggleAllButton"
				/>
				
				<form @submit.prevent="handleSubmit">
					<input
						data-cy="NewTodoField"
						type="text"
						class="todoapp__new-todo"
						placeholder="What needs to be done?"
						v-model="title"
					/>
				</form>
			</header>
			
			<TransitionGroup
				name="list"
				tag="section"
				class="todoapp__main"
				data-cy="TodoList"
			>
				<TodoItem
					v-for="(todo, index) of visibleTodos"
					:key="todo.id"
					:todo="todo"
					@update="updateTodo"
					@delete="deleteTodo(todo.id)"
				/>
			</TransitionGroup>
			
			<footer
				class="todoapp__footer"
				data-cy="Footer"
			>
          <span
	          class="todo-count"
	          data-cy="TodosCounter"
          >
            {{ activeTodos.length }} items left
          </span>
				
				<Filter
					v-model="status"
				/>
				
				<button
					v-if="activeTodos.length > 0"
					type="button"
					class="todoapp__clear-completed"
					data-cy="ClearCompletedButton"
				>
					Clear completed
				</button>
			</footer>
		</div>
		
		<Message
			class="is-warning"
			:active="errorMessage !== ''"
			@hide="errorMessage = ''"
			ref="errorMessage"
		>
			<template #default="{x}">
				<p>{{ errorMessage }} {{ x }}</p>
			</template>
			
			<template #header>
				<p>Server Error</p>
			</template>
		</Message>
	</div>
</template>

<style>
.list-enter-active,
.list-leave-active {
	max-height: 60px;
	transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
	opacity: 0;
	max-height: 0;
	transform: scaleY(0);
}
</style>