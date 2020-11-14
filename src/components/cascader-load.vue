<template>
  <span>
  <el-cascader
    :props="props"
    :value="currentValue"
    clearable></el-cascader>
  </span>
</template>

<script>

let parent_id = 0;

export default {
  data() {
    let _self = this;
    return {
      props: {
        lazy: true,
        lazyLoad (node, resolve) {
          const { level } = node;
          console.log(level, 'aaaa', node, 'nnnnnnnnmmm', resolve);

          setTimeout(() => {
            const nodes = Array.from({ length: level + 1 })
              .map(item => ({
                value: ++parent_id,
                label: `选项${parent_id}`,
                leaf: level >= 2
              }));
              console.log(nodes, 'naaffffff');
            // 通过调用resolve将子节点数据返回，通知组件数据加载完成
            //resolve(nodes);
            _self.attachmentPathModel.$fetch({query: {parent_id: node.value, action: 'list', 'point_scene': 'keyvalue'}}).then(response => {
              let keyField = response.key;
              let nameField = response.name;
              let elems = response.data;
              let nodes = [];
              elems.forEach(info => {
                  console.log(info, '===index');

                nodes.push({
                  value: info[keyField],
                  label: info[nameField],
                  leaf: false,
                });
              });
              resolve(nodes);
console.log('hhafffffffff', response, nodes);
            })
          }, 1000);
        }
      },
      currentValue: 1,
    }
  },
  computed: {
    test() {
    }
  },
  created() {
    //if (this.attachmentPathModel.entity) {
      //this.getPathDatas()
    //}
  },
  props: {
    attachmentPathModel: {type: Function},
  },
  methods: {
    getPathDatas() {
      this.attachmentPathModel.$fetch({params: {action: 'my-routes'}}).then(response => {
        console.log('ppppppppp', response);
      })
    },
  }
};
</script>
