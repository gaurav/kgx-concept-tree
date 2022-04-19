<template>
  <div class="text-left">
    <p v-for="designation in cde.designations" :key="designation.designation">
      <strong>Designation: </strong> {{designation}}
    </p>
    <ul>
      <li v-for="element in cde.formElements" :key="element.label">
        <strong>Question: </strong> {{element.label}}
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
    cde: Object
  },
  data() { return {
      toggle_textarea: false,
    };
  },
}
</script>

<style>
</style>
