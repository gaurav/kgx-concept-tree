<template>
  <div>
    {{cde.id}}
    <p v-for="designation in cde.designations" :key="designation.designation">
      <strong>Designation: </strong> {{designation}}
    </p>
    <ul>
      <li v-for="element in cde.formElements" :key="element.label">
        <strong>Question: </strong> {{element.label}}
      </li>
    </ul>
    <textarea
        disabled
        class="cde_text"
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
  }
}
</script>

<style>
</style>
