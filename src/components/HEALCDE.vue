<template>
  <div class="text-left">
    <p v-for="designation in cde.designations" :key="designation.designation">
      <strong>Designation: </strong> <span v-html="highlight(designation.designation)" />
    </p>
    <ul>
      <li v-for="element in cde.formElements" :key="element.label">
        <strong>Question: </strong> <span v-html="highlight(element.label)" />
      </li>
    </ul>
    <button class="col-12" @click="toggle_textarea = !toggle_textarea">Toggle NER text</button>
    <textarea
        v-if="toggle_textarea"
        disabled
        class="cde_text col-12"
        rows="20"
        v-model="cde.text"
    ></textarea>
  </div>
</template>

<script>
export default {
  name: 'HEALCDE',
  props: {
    cde: Object,
    highlight_text: Array[String],
  },
  data() { return {
      toggle_textarea: false,
    };
  },
  methods: {
    highlight(text) {
      if (!this.highlight_text) return text;
      let highlighted_text = text;
      this.highlight_text.forEach(text_to_highlight => {
        const query_text = text_to_highlight.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
        console.log('text_to_highlight=', text_to_highlight, 'query_text=', query_text, 'highlighted_text=', highlighted_text);
        highlighted_text = highlighted_text.replace(new RegExp(query_text, 'gi'), match => {
          return `<span class="highlight">${match}</span>`;
        })
      });
      return highlighted_text;
    },
  },
}
</script>

<style>
span.highlight {
  font-size: 150%;
  font-weight: bolder;
  font-style: italic;
}
</style>
