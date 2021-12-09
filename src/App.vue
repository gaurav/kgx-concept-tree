<template>
  <div id="app">
    <h1>KGX Concept Tree</h1>

    <div class="col-10 m-auto">
      <b-card header="Inputs" class="mt-3">
        <b-input-group prepend="KGX nodes file">
          <b-form-file v-model="nodes_file"></b-form-file>
        </b-input-group>
        <b-input-group prepend="KGX edges file">
          <b-form-file v-model="edges_file"></b-form-file>
        </b-input-group>
      </b-card>

      <b-card header="File stats" class="mt-3">
        The KGX files contain {{nodes.length}} nodes (including {{concepts.length}} concepts, {{concept_ids.length}} unique) and {{edges.length}} edges.
      </b-card>

      <b-card header="Concepts" class="mt-3">
        <ul style="text-align: left">
          <li v-for="category in concept_categories" :key="category.name" :id="category.name">
            <a :href="'#' + category.name"
              @click="selected_category=(selected_category == category.name ? '' : category.name)">{{category.name}}</a> ({{category.count}})
 
            <ul v-if="selected_category == category.name">
              <li v-for="concept in category.concepts" :key="concept['id']">
                {{concept.name}} ({{concept.id}})
                
                <ul v-if="get_cdes_for_concept(concept)">
                  <li v-for="cde in get_cdes_for_concept(concept)" :key="cde['id']">
                    <tt>{{cde.terms}}</tt> in {{cde.id}}
                  </li>
                </ul>
              </li>
            </ul>
           
          </li>
        </ul>

      </b-card>

      <p></p>
    </div>
  </div>
</template>

<script>
import { groupBy, toPairs } from 'lodash'

export default {
  name: 'App',
  data() { return {
    nodes_file: null,
    nodes_text: "",
    edges_file: null,
    edges_text: "",
    selected_category: "",
  }},
  watch: {
    nodes_file() {
      this.nodes_file.text().then(content => { this.nodes_text = content })
    },
    edges_file() {
      this.edges_file.text().then(content => { this.edges_text = content })
    },
  },
  computed: {
    nodes() {
      return (this.nodes_text).split(/\r\n|\r|\n/).filter(str => str != '').map(JSON.parse);
    },
    concepts() {
      return this.nodes.filter(node => {
        let provided_by = node['provided_by']
        if(provided_by && (provided_by[0] == 'Monarch NER service + Translator normalization API'))
          return true;
        return false;
      });
    },
    concept_ids() {
      return [...new Set(this.concepts.map(concept => concept['id']))];
    },
    concept_categories() {
      // TODO: Replace with lodash.includes()
      let categories = [...new Set(this.concepts.map(concept => concept['category']).flat())];
      return categories.map(category => {
        let concepts = this.concepts.filter(concept => new Set(concept.category).has(category))
        return {
          name: category,
          count: concepts.length,
          concepts: concepts
        }
      }).sort((a, b) => a.count > b.count)
    },
    edges() {
      return (this.edges_text).split(/\r\n|\r|\n/).filter(str => str != '').map(JSON.parse);
    },
  },
  methods: {
    get_cdes_for_concept(concept) {
      let edges = this.edges.filter(edge => edge['object'] == concept.id)
      let cdes = groupBy(edges, edge => edge['subject'])
      return toPairs(cdes).map(entry => {
        let [key, value] = entry;
        return {
          id: decodeURI(key),
          terms: [...new Set(value.map(edge => edge['name'].toLowerCase()))],
        }
      })
    },
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
