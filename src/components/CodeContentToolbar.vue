<template>
  <div class="top-toolbar">
    <el-button
      type="primary"
      @click="play"
      size="mini"
      v-if="!active_tab.is_running"
    >
      Run
    </el-button>
    <el-button v-else type="info" size="mini" disabled>
      Running
    </el-button>
    <label class="label-language">Language</label>
    <el-select v-model="language" placeholder="Choose Language" size="mini">
      <el-option
        v-for="item in languages"
        :key="item.value"
        :label="item.text"
        :value="item.value"
      >
      </el-option>
    </el-select>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import languages from '@/lib/languages'
import shell from 'shelljs'

export default {
  name: 'CodeContentToolbar',
  computed: {
    ...mapState('code', {
      active_tab: state => state.active_tab
    }),
    language: {
      get() {
        return this.active_tab.language
      },
      set(value) {
        this.active_tab.language = value
        this.active_tab.code = this.languages.find(
          it => it.value === value
        ).code
      }
    }
  },
  data() {
    return {
      languages
    }
  },
  methods: {
    play() {
      this.active_tab.is_running = true
      const workDir = `/tmp/codepad/${this.active_tab.language}`
      shell.mkdir('-p', workDir)
      this.languages
        .find(it => it.value === this.active_tab.language)
        .run(workDir, this.active_tab.code, (result, error) => {
          this.active_tab.output = error || result.output
          this.active_tab.command = result.command || ''
          this.active_tab.is_running = false
        })
    }
  }
}
</script>

<style lang="scss" scoped>
.top-toolbar {
  flex-wrap: nowrap;
  padding: 0 0.75rem 0.75rem 0.75rem;
  border-bottom: 1px solid rgba(0, 0, 0, 0.125);
}
.label-language {
  vertical-align: middle;
  margin: 0 1rem 0 1rem;
}
</style>
