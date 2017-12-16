<template>
    <div>
        <div>
            <div v-if="scope">
                <directive :directive="scope"
                                :name="scope.name"
                                :value="getDirectivePropertiesString(scope)"
                                :type="DIRECTIVE_TYPES.FAKE_SCOPE_PARAMETER"
                                :postfix="''"
                                @on-click="selectView"
                ></directive>
            </div>
            <div v-for="({directive, name, value, type, postfix}, index) in formattedDirectives" :key="index">
                <directive :directive="directive"
                                :name="name"
                                :value="value"
                                :type="type"
                                :postfix="postfix"
                                @on-click="selectView"
                ></directive>
            </div>
        </div>
    </div>
</template>

<script>
    import Directive from '@/components/HttpdConfigViewer/Directive'
    import {DIRECTIVE_TYPES} from '@/utils/types'

    export default {
      components: {Directive},
      props: {
        config: {
          type: Object,
          default () {
            return {}
          }
        }
      },
      data () {
        return {DIRECTIVE_TYPES}
      },
      computed: {
        scope () {
          return this.config.type && this.config.type !== DIRECTIVE_TYPES.ROOT ? this.config : null
        },
        filteredDirectives () {
          if (!this.config.body) return []
          return this.config.body.filter(item => {
            return [
              DIRECTIVE_TYPES.SCOPED,
              DIRECTIVE_TYPES.PLAIN
            ].indexOf(item.type) !== -1
          })
        },
        formattedDirectives () {
          return this.filteredDirectives.map(directive => {
            return this.formatDirective(directive)
          })
        }
      },
      methods: {
        formatDirective (directive) {
          const format = {
            directive,
            name: directive.name,
            value: this.getDirectivePropertiesString(directive),
            type: directive.type
          }

          if (directive.type === DIRECTIVE_TYPES.SCOPED) {
            if (directive.name.toLowerCase() === 'virtualhost') {
              Object.assign(format, {
                value: this.getChildDirectives(directive)['ServerName'],
                postfix: directive.parameters[0].value
              })
            }
          }

          return format
        },
        selectView (view) {
          if (view.type !== DIRECTIVE_TYPES.SCOPED || view === this.config) return
          this.$store.commit('Files/PUSH_VIEW', { view })
        },
        getChildDirectives (directive) {
          const directives = {}
          directive.body.forEach(item => {
            if (item.type !== DIRECTIVE_TYPES.PLAIN || item.parameters.length === 0) return true
            directives[item.name] = item.parameters[0].value
          })
          return directives
        },
        getDirectivePropertiesString (directive) {
          return directive.parameters.map(param => {
            return param.value
          }).join(', ')
        }
      }
    }
</script>