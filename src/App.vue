<template>
  <div id="app">
    <h1>KGX Concept Tree</h1>

    <div class="col-10 m-auto">
      <b-card header="Inputs" class="mt-3">
        <b-alert variant="info" show v-if="input_in_progress">
          Please wait, loading input files ...
        </b-alert>
        <b-alert variant="danger" class="text-left" show v-if="input_errors.length > 0">
          <p>Errors occurred while reading input files:</p>
          <ul>
            <li v-for="err in input_errors" :key="err.row">
              Row {{err.rowIndex}}: {{err.text}}
            </li>
          </ul>
        </b-alert>
        <b-input-group prepend="KGX nodes file">
          <b-form-file v-model="nodes_file"></b-form-file>
        </b-input-group>
        <b-input-group prepend="KGX edges file">
          <b-form-file v-model="edges_file"></b-form-file>
        </b-input-group>
        <b-input-group prepend="Comprehensive JSONL file">
          <b-form-file v-model="comprehensive_file"></b-form-file>
        </b-input-group>
      </b-card>

      <b-card header="File stats" class="mt-3">
        The KGX files contain {{nodes.length}} nodes (including {{concepts.length}} concepts, {{concept_ids.length}} unique) and {{edges.length}} edges.
        <br>
        The comprehensive JSONL file contains {{comprehensive_keys.length}} CDEs.
      </b-card>

      <div class="container col-12">
        <div class="row">
          <div :class="selected_category_cde ? 'col-7' : 'col-12'">
            <b-card header="Concepts by categories" class="mt-3">
              <ul style="text-align: left">
                <li v-for="category in concept_categories" :key="category.name" :id="category.name">
                  <a :href="'#' + category.name"
                    @click="selected_category=(selected_category == category.name ? '' : category.name)">{{category.name}}</a> ({{category.count}})

                  <ul v-if="selected_category == category.name">
                    <li v-for="concept in category.concepts" :key="concept['id']">
                      <a :href="'#' + concept.name"
                         @click="selected_concept=(selected_concept === concept.name ? '' : concept.name)"
                      >
                      {{concept.name}}: {{get_cdes_for_concept(concept).length}} CRFs
                      </a>
                      (<a target="code" :href="get_uri_for_curie(concept.id)">{{concept.id}}</a>)

                      <ul v-if="selected_concept == concept.name && get_cdes_for_concept(concept)">
                        <li v-for="cde in get_cdes_for_concept(concept)" :key="cde['id']">
                          <tt>{{cde.terms}}</tt> in <a :href="'#' + category.name" @click="selected_category_cde = comprehensive[cde.id.substring(8)]; selected_category_text = cde.terms;">{{cde.id}}</a>
                        </li>
                      </ul>
                    </li>
                  </ul>

                </li>
              </ul>
            </b-card>
          </div>

          <div v-if="selected_category_cde" class="col-5">
            <b-card class="mt-3" v-for="cde in selected_category_cde" :key="cde.id" :header="cde.id">
              <HEALCDE :cde="cde" :highlight_text="selected_category_text" />
            </b-card>
          </div>
        </div>

        <div class="row">
          <div :class="selected_category_cde ? 'col-7' : 'col-12'">
            <b-card header="Concepts by count" class="mt-3">
              <ul style="text-align: left">
                <li v-for="concept in sorted_concepts" :key="concept.name">

                  <a :href="'#count_for_' + concept.name"
                     @click="selected_concept=(selected_concept === concept.name ? '' : concept.name)"
                  >{{concept.name}}</a>
                  (<a target="code" :href="get_uri_for_curie(concept.id)">{{concept.id}}</a>): {{concept.count}} CRFs
                  <ul v-if="selected_concept === concept.name">
                    <li v-for="cde in concept.cdes" :key="cde['id']">
                      <tt>{{cde.terms}}</tt> in <a :href="'#' + concept.name"  @click="selected_concept_cde = comprehensive[cde.id.substring(8)];  selected_concept_text = cde.terms;">{{cde.id}}</a>
                    </li>
                  </ul>
                </li>
              </ul>
            </b-card>
          </div>

          <div v-if="selected_concept_cde" class="col-5">
            <b-card class="mt-3" v-for="cde in selected_concept_cde" :key="cde.id" :header="cde.id">
              <HEALCDE :cde="cde" :highlight_text="selected_concept_text" />
            </b-card>
          </div>
        </div>
      </div>

      <!--
      <b-card header="CDEs" class="mt-3">
        <ul style="text-align: left">
          <li v-for="filename in comprehensive_keys" :key="filename">
            <a :id="'HEALCDE:' + filename">{{filename}}</a>
            <ul>
              <li v-for="cde in comprehensive[filename]" :key="cde.id">
                <HEALCDE :cde="cde" />
              </li>
            </ul>
          </li>
        </ul>
      </b-card> -->

      <p></p>
    </div>
  </div>
</template>

<script>
import { groupBy, toPairs, cloneDeep, has, keys } from 'lodash'
import Vue from 'vue'

import HEALCDE from './components/HEALCDE'

export default {
  name: 'App',
  components: { HEALCDE },
  data() { return {
    input_in_progress: false,
    input_errors: [],
    nodes_file: null,
    nodes_text: "",
    edges_file: null,
    edges_text: "",
    comprehensive_file: null,
    comprehensive: {},
    selected_category: "",
    selected_concept: "",
    selected_category_cde: null,
    selected_category_text: null,
    selected_concept_cde: null,
    selected_concept_text: null,
    PREFIXES: {
      'HP': 'http://purl.obolibrary.org/obo/HP_',
      'GO': 'http://purl.obolibrary.org/obo/GO_',
      'PUBCHEM.COMPOUND': 'https://pubchem.ncbi.nlm.nih.gov/compound/',
    },
  }},
  watch: {
    nodes_file() {
      this.input_in_progress = true;
      this.nodes_file.text().then(content => {
        this.nodes_text = content;
        this.input_in_progress = false;
      });
    },
    edges_file() {
      this.input_in_progress = true;
      this.edges_file.text().then(content => {
        this.edges_text = content;
        this.input_in_progress = false;
      });
    },
    comprehensive_file() {
      this.input_in_progress = true;
      this.comprehensive_file.text().then(content => {
        this.input_errors = [];
        content.split(/[\n\r]+/).forEach((row, rowIndex) => {
          if (row.trim() == '') return;
          let entry;
          try {
            entry = JSON.parse(row);
            console.log(keys(entry));
          } catch (err) {
            this.input_errors.push({
              rowIndex,
              text: `Could not parse comprehensive JSONL line ${rowIndex}: ${err}`
            })
          }
          let filenames = entry.designations.flatMap(d => {
            if (d.designation.startsWith('Filename: ')) {
              return [d.designation.substr(10)];
            } else return [];
          });
          filenames.forEach(filename => {
            if (!(filename in this.comprehensive)) {
              Vue.set(this.comprehensive, filename, []);
            }
            this.comprehensive[filename].push({
              id: `HEALCDE:${filename}`,
              designations: entry['designations'],
              formElements: entry['formElements'],
              text: entry['_ner']['oger']['crf_text'],
            });
            // console.log("Found entry for", filename, "with keys", keys(entry['_ner']));
          });
          this.input_in_progress = false;
        });
      });
    }
  },
  computed: {
    comprehensive_keys() {
      return keys(this.comprehensive);
    },
    nodes() {
      return (this.nodes_text).split(/\r\n|\r|\n/).filter(str => str != '').map(JSON.parse);
    },
    concepts() {
      return this.nodes.filter(node => {
        let provided_by = node['provided_by']
        if(provided_by && (provided_by[0] != 'Graph'))
          return true;
        return false;
      });
    },
    sorted_concepts() {
      // Returns a list of concepts sorted by count, including the count in the 'count' field.
      return this.concepts.map(concept => {
        let c = cloneDeep(concept)
        c['cdes'] = this.get_cdes_for_concept(c)
        c['count'] = c['cdes'].length
        return c
      }).sort((a, b) => a['count'] < b['count'])
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
          concepts: concepts.map(concept => {
            let c = cloneDeep(concept)
            c['cdes'] = this.get_cdes_for_concept(c)
            c['count'] = c['cdes'].length
            return c
          }).sort((a, b) => a['count'] < b['count'])
        }
      }).sort((a, b) => a.count > b.count)
    },
    edges() {
      return (this.edges_text).split(/\r\n|\r|\n/).filter(str => str != '').map(JSON.parse);
    },
  },
  methods: {
    get_uri_for_curie(curie) {
      const [prefix, code] = curie.split(':');
      if (has(this.PREFIXES, prefix)) {
        return `${this.PREFIXES[prefix]}${code}`;
      }
      return `https://www.ebi.ac.uk/ols/search?q=${encodeURIComponent(curie)}`;
    },
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

textarea.cde_text {
  width: 100%;
}
</style>
