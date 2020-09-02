<script>
// Reference: https://vuejs.org/v2/guide/render-function.html#v-if-and-v-for
// props: ['items'],
// render: function (createElement) {
//   if (this.items.length) {
//     return createElement('ul', this.items.map(function (item) {
//       return createElement('li', item.name)
//     }))
//   } else {
//     return createElement('p', 'No items found.')
//   }
// }

// Reference: https://vuejs.org/v2/guide/render-function.html#Basics
// Vue.component('anchored-heading', {
//   render: function (createElement) {
//     return createElement(
//       'h' + this.level,   // tag name
//       this.$slots.default // array of children
//     )
//   },
//   props: {
//     level: {
//       type: Number,
//       required: true
//     }
//   }
// })

// Reference: https://frontendsociety.com/why-you-shouldnt-use-vue-component-ff019fbcac2e
// on the topic of correctly export the component
  export default {
    render: function (createElement) {
      return createElement('li', this.str2dom(this.$slots.default[0].text, createElement))
    },
    // props: {
    //   key: {
    //     type: String,
    //     required: true
    //   }
    // },
    methods: {
      str2dom (str, h) {
        var re = /(%(username|warning)=.+?%)/g
        return str.split(re).map((el) => {
          switch (el) {
          case 'username':
            return ''
          case 'warning':
            return ''
          default:
            break
          }
          var m = el.match(/%(.+)=(.+)%/)
          if (m) {
            return h('span', {
              attrs: { class: m[1] }
            }, m[2])
          } else {
            return el
          }
        })
      }
    }
  }

</script>>
