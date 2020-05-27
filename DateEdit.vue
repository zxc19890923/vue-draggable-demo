<template>
  <div>
    <div class="editTagText">
      <span class="editTagLabel">节点名称</span>
      <span class="editTagNumber">4/10</span>
    </div>
    <div class="editTag">
      <Input type="text" v-model="childItemInfo.label" :maxlength="10" @on-change="requireStateChange"/>
    </div>
    <div class="editTagText">
      <span class="editTagLabel">时间日期起类型</span>
    </div>
    <div class="editTag">
      <RadioGroup v-model="childItemInfo.limit"  @on-change="requireStateChange">
        <Radio :value=1 :label=1>时间点</Radio>
        <Radio :value=2 :label=2>起止时间</Radio>
      </RadioGroup>
    </div>
    <div class="editTag">
      <Select v-model="childItemInfo.dateType"  @on-change="requireStateChange">
        <Option :value=1 label='日期'>日期</Option>
        <Option :value=2 label='日期+时间'>日期+时间</Option>
      </Select>
    </div>
    <div class="editTagText">
      <span class="editTagLabel">校验</span>
    </div>
    <div class="editTag">
      <el-checkbox v-model="childItemInfo.required" @change="requireStateChange">
      <span style="color: #333;">设为必填项</span>
      </el-checkbox>
    </div>
    <div class="deleteScene">
      <!-- <button @click="deleteTagFun">删除节点</button> -->
    </div>
  </div>
</template>
<script>
export default {
  data () {
    return {}
  },
  props: ['itemInfo'],
  computed: {
    childItemInfo () {
      return this.itemInfo
    }
  },
  methods: {
    requireStateChange () {
      console.log(this.childItemInfo)
      this.$emit('changeItemInfo', this.childItemInfo)
    },
    deleteTagFun () {
      this.$emit('deleteItem', this.childItemInfo)
    }
  }
}
</script>
<style lang="scss" scoped>
  .editTagText {
    margin-bottom: 10px;
    .editTagLabel {
      font-weight: 600;
      margin-right: 5px;
    }
    .editTagNumber {
      color: #999999;
    }
  }
  .editTag {
    margin-bottom: 20px;
  }
  .deleteScene button {
    margin-top: 10px;
    background: #fff;
    border: 1px solid #F05A50;
    color: #F05A50;
    text-align: center;
    outline: none;
    height: 30px;
    line-height: 30px;
    width: 100%;
    cursor: pointer;
  }
</style>
