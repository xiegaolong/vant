<template>
  <picker
    class="van-area"
    ref="picker"
    show-toolbar
    value-key="name"
    :title="title"
    :columns="columns"
    @change="onChange"
    @confirm="$emit('confirm', $event)"
    @cancel="$emit('cancel', $event)"
  />
</template>

<script>
import { create } from '../utils';
import Picker from '../picker';

export default create({
  name: 'van-area',

  components: {
    Picker
  },

  props: {
    value: {},
    title: String,
    areaList: Object,
    // 省市县显示列数，3-省市县，2-省市，1-省
    columnsNum: {
      type: [String, Number],
      default: 3
    }
  },

  computed: {
    listValid() {
      return this.areaList && typeof this.areaList.province_list === 'object';
    },

    columns() {
      const columns = [];

      if (!this.listValid) {
        return columns;
      }

      const code = this.value || '';
      const columnsNum = +this.columnsNum;

      columns.push({
        values: this.getList('province')
      });

      if (columnsNum > 1) {
        columns.push({
          values: this.getList('city', code.slice(0, 2))
        });
      }

      if (columnsNum > 2) {
        columns.push({
          values: this.getList('county', code.slice(0, 4))
        });
      }

      return columns;
    }
  },

  mounted() {
    this.setIndex();
  },

  watch: {
    value() {
      this.setIndex();
    }
  },

  methods: {
    setIndex() {
      this.$nextTick(() => {
        const code = this.value || '';
        const { picker } = this.$refs;
        picker && picker.setIndexes([
          this.getIndex('province', code),
          this.getIndex('city', code),
          this.getIndex('county', code)
        ]);
      });
    },

    // 根据省市县类型和对应的`code`获取对应列表
    getList(type, code) {
      if (!this.listValid) {
        return [];
      }

      const { areaList } = this;
      const list =
        type === 'province'
          ? areaList.province_list
          : type === 'city' ? areaList.city_list : areaList.county_list;

      let result = Object.keys(list).map(code => ({
        code,
        name: list[code]
      }));

      if (type !== 'province' && code) {
        result = result.filter(item => item.code.indexOf(code) === 0);
      }

      result.unshift({
        code: '-1',
        name: this.$t(type)
      });

      return result;
    },

    // 获取对应省市县在列表中的索引
    getIndex(type, code) {
      const compareNum = type === 'province' ? 2 : type === 'city' ? 4 : 6;
      const areaList = this.getList(type, code.slice(0, compareNum - 2));
      code = code.slice(0, compareNum);

      for (let i = 0; i < areaList.length; i++) {
        if (areaList[i].code.slice(0, compareNum) === code) {
          return i;
        }
      }

      return 0;
    },

    onChange(picker, values, index) {
      const code = values[index].code;
      // 处理省变化
      if (index === 0) {
        picker.setColumnValues(1, this.getList('city', code.slice(0, 2)));
        picker.setColumnValues(2, this.getList('county', code.slice(0, 4)));
      } else if (index === 1) {
        picker.setColumnValues(2, this.getList('county', code.slice(0, 4)));
      }
    },

    getValues() {
      return this.$refs.picker ? this.$refs.picker.getValues() : [];
    }
  }
});
</script>
