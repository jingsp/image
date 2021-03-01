<template>
  <div>
    <el-select
      class="select-scroll"
      v-scroll="handleScroll"
      v-model="currentValue"
      clearable
      filterable
      :popper-append-to-body="false"
      v-if="list.length > 0"
    >
      <el-option
        :value="item.value"
        :label="item.label"
        v-for="(item, index) in list"
        :key="index"
      >{{ item.label }}</el-option>
    </el-select>
    <el-select
      class="select-scroll"
      v-scroll="handleScroll"
      :popper-append-to-body="false"
      v-if="list.length === 0"
      v-model="currentValue"
    >
      <el-option
        :value="item.value"
        :label="item.label"
        v-for="(item, index) in list"
        :key="index"
      >{{ item.label }}</el-option>
    </el-select>
  </div>
</template>

<script>
import scroll from './directive';

export default {
  props: {
    list: {
      type: Array,
      default: () => [],
    },
    value: {
      type: String,
      default: null,
    },
  },

  // data() {
  //   return {
  //     currentValue: this.value,
  //   };
  // },
  computed: {
    currentValue: {
      get() { return this.value; },
      set(val) {
        this.$emit('update:value', val);
      },
    },
  },
  methods: {
    handleScroll(v) {
      this.$emit('handleScroll', v);
    },
  },
  watch: {
    // currentValue(val) {
    //   // this.$emit('handleChnage', v);
    //   // this.value = val;
    //   this.$emit('update:value', val);
    // },
  },
  directives: {
    scroll,
  },
};
</script>

<style lang="scss" scoped>
.select-scroll {

  .el-select-dropdown__item {
    width: 200px;
  }
}
</style>
