<template>
    <div class="max-w-screen-md w-full mx-auto">
        <div class="form-group mb-4">
            <Link :href="route('posts.index')" class="inline-block bg-sky-600 px-3 py-2 text-white">Назад</Link>
        </div>
        <h1 style="word-break: break-word;" class="pb-4 text-xl">{{post.title}}</h1>
        <div class="pb-4"><img class="w-50 h-80 object-cover" :src="post.image" :alt="post.id"></div>
        <p style="word-break: break-word;" class="pb-4">{{post.content}}</p>
        <div class="flex justify-between items-center mt-2">
            <div class="flex">
                <svg @click.prevent="toggleLike(post)" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"
                     stroke-width="1.5"
                     stroke="currentColor"
                     :class="['mr-2 stroke-sky-500 cursor-pointer hover:fill-sky-500 w-6 h-6', post.is_liked ? 'fill-sky-500' : 'fill-white']">
                    <path stroke-linecap="round" stroke-linejoin="round"
                          d="M21 8.25c0-2.485-2.099-4.5-4.688-4.5-1.935 0-3.597 1.126-4.312 2.733-.715-1.607-2.377-2.733-4.313-2.733C5.1 3.75 3 5.765 3 8.25c0 7.22 9 12 9 12s9-4.78 9-12z"/>
                </svg>
                <p>{{post.likes_count}}</p>
            </div>
            <p class="text-right text-sm text-slate-500">{{post.date}}</p>
        </div>
        <div v-if="post.comments_count > 0" class="mt-4">
            <p class="pb-4 text-xl link-text" v-if="!isShowed" @click="getComments(post)">Показать
                {{ post.comments_count }} комментарий</p>
            <p class="pb-4 text-xl link-text" v-if="isShowed" @click="isShowed = false">Закрыть</p>
            <div v-if="comments && isShowed">
                <div v-for="comment in comments" class="mt-4 pt-4 border-t border-gray-300">
                    <p class="text-sm">{{ comment.user.name }}</p>
                    <p style="word-break: break-word;">{{ comment.body }}</p>
                    <p class="text-right text-sm text-slate-500">{{ comment.date }}</p>
                </div>
            </div>
        </div>
        <div class="mt-4">
            <div class=" mb-3">
                <input v-model="body" class="w-96 border p-2 border-slate-300" type="text"
                       placeholder="Добавить комментарий">
            </div>
            <div class="mb-3" v-if="errors.body">
                <p v-for="error in errors.body" class="text-sm mt-2 text-red-500">{{ error }}</p>
            </div>
            <div class="form-group mb-4">
                <a @click.prevent="storeComment(post)" href="#"
                   class="inline-block bg-sky-600 px-3 py-2 text-white">Комментировать</a>
            </div>
        </div>
        <div v-if="isAdmin" class="form-group my-4 flex items-center">
            <Link :href="route('posts.edit', post.id)" class="inline-block bg-green-600 px-3 py-2 text-white">Редактировать</Link>
            <Link as="button" method="delete" :href="route('posts.destroy', post.id)" class="inline-block bg-rose-600 px-3 py-2 text-white ml-2">Удалить</Link>
        </div>
        </div>
</template>

<script>
import MainLayout from "@/Layouts/MainLayout.vue";
import {Link} from "@inertiajs/vue3";
import axios from "axios";

export default {
    name: "Show",

    props:['post', "isAdmin"],
    data() {
        return {
            body: '',
            comments: [],
            errors: [],
            isShowed: false,
        }
    },

    components: {Link},

    methods: {

        toggleLike(post) {
            axios.post(`/posts/${post.id}/toggle_like`)
                .then(res => {
                    post.is_liked = res.data.is_liked;
                    post.likes_count = res.data.likes_count;
                })
                .catch(error => {
                    console.error("Ошибка при обновлении лайка:", error);
                });
        },
        storeComment(post) {
            axios.post(`/posts/${post.id}/comment`, {body: this.body})
                .then(res => {
                    this.body = ''
                    this.comments.unshift(res.data.data)
                    post.comments_count++
                    this.isShowed = true
                })
                .catch(e => {
                    this.errors = e.response.data.errors
                })
        },

        getComments(post) {
            axios.get(`/posts/${post.id}/comment`)
                .then(res => {
                    this.comments = res.data.data
                    this.isShowed = true
                })
        },
    },

    layout: MainLayout
}
</script>

<style scoped>
.link-text {
    font-size: medium;
    transition: color 0.3s; /* добавлен переход для плавного изменения цвета */
    cursor: pointer;
}

.link-text:hover {
    color: blue; /* цвет при наведении */
}
</style>
