<template>
  <div id="app">
    <KnowledgeGraph />
  </div>
</template>

<script>
import KnowledgeGraph from "./components/KnowledgeGraph.vue";

export default {
  components: {
    KnowledgeGraph,
  },
};
</script>

<style>
#app {
  text-align: center;
  padding: 20px;
}
</style>
