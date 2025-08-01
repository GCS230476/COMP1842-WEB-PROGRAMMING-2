<template>
  <div>
    <h1>Show Word</h1>

    <div class="ui labeled input fluid">
      <div class="ui label">
        <i class="germany flag"></i> German
      </div>
      <input type="text" readonly :value="word.german" />
    </div>

    <div class="ui labeled input fluid">
      <div class="ui label">
        <i class="united kingdom flag"></i> English
      </div>
      <input type="text" readonly :value="word.english" />
    </div>

    <div class="ui labeled input fluid">
      <div class="ui label">
        <i class="france flag"></i> French
      </div>
      <input type="text" readonly :value="word.french" />
    </div>

    <div class="actions">
      <router-link
        :to="{ name: 'edit', params: { id: $route.params.id } }"
        class="ui blue button"
      >
        Edit Word
      </router-link>

      <router-link
        to="/words/new"
        class="ui green button"
        style="margin-left: 10px"
      >
        Add Another Word
      </router-link>

      <router-link
        to="/words"
        class="ui grey button"
        style="margin-left: 10px"
      >
        Back to Home
      </router-link>
      </div>
  </div>
</template>

<script>
import { api } from '../helpers/helpers';

export default {
  name: 'show',
  data() {
    return {
      word: ''
    };
  },
  async mounted() {
    this.word = await api.getWord(this.$route.params.id);
  },
  methods: {
    clearFlash() {
      if (this.$root.$children[0]?.$refs?.flashMessage) {
        this.$root.$children[0].$refs.flashMessage.clear();
      }
    }
  }
};
</script>

<style scoped>
.actions a {
  display: inline-block;
  margin-top: 20px;
  margin-right: 10px;
}
</style>
