<template>
    <div>
        <h1>Words</h1>
        <div class="ui icon input" style="margin-bottom: 20px;">
            <input
                type="text"
                v-model="searchQuery"
                placeholder="Search words..."
            />
            <i class="search icon"></i>
        </div>

        <table id="words" class="ui celled compact table">
            <thead>
                <tr>
                    <th>English</th>
                    <th>German</th>
                    <th>French</th>
                    <th colspan="3"></th>
                </tr>
            </thead>
            <tr v-for="(word, i) in filteredWords" :key="i">
                <td>{{ word.english }}</td>
                <td>{{ word.german }}</td>
                <td>{{ word.french }}</td>
                <td width="75" class="center aligned">
                <router-link :to="{ name: 'show', params: { id: word._id }}">Show</router-link></td>
                <td width="75" class="center aligned">
                <router-link :to="{ name: 'edit', params: { id: word._id }}">Edit</router-link></td>
                <td width="75" class="center aligned" @click.prevent="onDestroy(word._id)">
                <a :href="`/words/${word._id}`">Destroy</a></td> 
            </tr>
        </table>
    </div>
</template>

    <script>
    import { api } from '../helpers/helpers';

    export default {
        name:'words',
        data() {
            return {
                words: [],
                searchQuery: ""
            };
            },
            computed: {
            filteredWords() {
                if (!this.searchQuery) return this.words;
                const q = this.searchQuery.toLowerCase();
                return this.words.filter(word =>
                word.english.toLowerCase().includes(q) ||
                word.german.toLowerCase().includes(q) ||
                word.french.toLowerCase().includes(q)
                );
            }
            },
        methods: {
            async onDestroy(id) {
                const sure = window.confirm('Are you sure?');
                if (!sure) return;
                await api.deleteWord(id);
                this.flash('Word deleted sucessfully', 'success');
                const newWords = this.words.filter(word => word._id !== id);
                this.words = newWords
            }
        },
        async mounted() {
            this.words = await api.getWords();
        }
    };
    </script>