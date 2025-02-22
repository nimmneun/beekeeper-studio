<template>
  <form
    class="save-connection expand"
    @submit.prevent="save"
  >
    <h3 class="dialog-c-title">
      Add database
    </h3>

    <div
      v-if="error"
      class="alert alert-danger"
    >
      {{ error }}
    </div>

    <div class="form-group">
      <input
        class="form-control"
        v-model="databaseName"
        @keydown.enter.prevent.stop="save"
        type="text"
        placeholder="Database Name"
      >
    </div>

    <div
      class="form-group"
      v-if="charsets.length > 0"
    >
      <select v-model="selectedCharset">
        <option
          v-for="charset in charsets"
          :key="charset"
          :selected="charset === selectedCharset"
          :value="charset"
        >
          {{ charset }}
        </option>
      </select>
    </div>

    <div
      class="form-group"
      v-if="collations.length > 0"
    >
      <select v-model="selectedCollation">
        <option
          v-for="collation in collations"
          :key="collation"
          :value="collation"
        >
          {{ collation }}
        </option>
      </select>
    </div>

    <div class="save-actions">
      <button
        class="btn btn-flat"
        @click.prevent="$emit('cancel')"
      >
        Cancel
      </button>
      <button
        class="btn btn-primary save"
        type="submit"
      >
        Add
      </button>
    </div>
  </form>
</template>

<script>
  export default {
    props: ['connection'],
    data() {
      return {
        charsets: [],
        collations: [],
        databaseName: null,
        selectedCharset: null,
        selectedCollation: null,
        error: null
      }
    },
    async mounted(){
      this.charsets = await this.connection.listCharsets()
      this.selectedCharset = await this.connection.getDefaultCharset()
      await this.updateCollations()
    },
    methods: {
      async updateCollations() {
        this.collations = await this.connection.listCollations(this.selectedCharset)
        this.selectedCollation = this.collations[0]
      },
      async save() {
        try {
          await this.connection.createDatabase(this.databaseName, this.selectedCharset, this.selectedCollation)
          this.$noty.success('The database was created')
          this.$emit('databaseCreated', this.databaseName)
        } catch (err) {
          this.error = `The database could not be created: ${err.message}"`
        }
      }
    },
    watch: {
      selectedCharset() {
        this.updateCollations()
      }
    }
  }
</script>