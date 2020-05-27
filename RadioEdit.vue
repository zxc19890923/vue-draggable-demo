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
      <span class="editTagLabel">设置选项 </span>
    </div>
    <div class="editTag">
      <RadioGroup v-model="childItemInfo.text" :vertical="true" style="width: 100%;" @on-change="requireStateChange">
        <draggable
          :group="{name: 'radioList', put: false, pull: false}"
          v-model="childItemInfo.radioList"
        >
          <div v-for="(val, i) in childItemInfo.radioList" :key="i" class="editSceneLable">
            <div class="moveHandle"></div>
            <div class="rowTwo">
              <Radio :label="val.value">
                <span></span>
              </Radio>
            </div>
            <div class="rowThree">
              <Input type="text" v-model="val.label" style="width: 100%;" />
            </div>
            <div class="rowFour">
              <i class="el-icon-delete" @click.stop="deleteOneItem(val, i)"></i>
            </div>
          </div>
        </draggable>
      </RadioGroup>
      <div class="optionEdit">
        <span @click="addOption">添加选项</span>
        <span @click="editAllOption">批量编辑</span>
        <span @click="linkOption">选项关联</span>
      </div>
      <div class="alreadySet" v-if="optionLinkState">
        <div class="alreadySetLabel">已设置关联选项</div>
        <div class="alreadySetIcon">
          <i class="el-icon-edit"></i>
        </div>
      </div>
    </div>
    <div class="editTagText">
      <span class="editTagLabel">排列方式</span>
    </div>
    <div class="editTag">
      <RadioGroup v-model="direction" @on-change="requireStateChange">
        <Radio :label='1'><span style="color: #666">横向</span></Radio>
        <Radio :label='2'><span style="color: #666">纵向</span></Radio>
      </RadioGroup>
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
    <Modal
      :title="'批量编辑'"
      v-model="editAllState"
      :width="555"
      @on-ok="submitEditChanne"
      @on-cancel="cancelEditChannel"
      :loading="loading"
      :mask-closable="false"
      :closable="false"
    >
    <div class="editModal">
      <div class="modalDec">一样对应一个选项</div>
      <div class="modalTextarea">
        <Input type="textarea" :rows="10" v-model="textAreaList" />
      </div>
    </div>
    </Modal>
    <Modal
      :title="'选项关联'"
      v-model="linkState"
      :width="555"
      @on-ok="submitEditLink"
      @on-cancel="cancelEditLink"
      :loading="loading"
      :mask-closable="false"
      :closable="false"
    >
    <div class="editLinkModal">
      <div class="editLinkLeft">
        <div class="editLabel">选项</div>
        <div class="editContent">
          <div v-for="(item, index) in itemInfo.radioList" :key="index" :class="{'editItem': true, 'linkActive': leftActiveIndex === index ? true : false}" @click="selectLeftItem(item, index)">
            {{item.label}}
          </div>
        </div>
      </div>
      <div class="editLinkRight">
        <div class="editLabel">关联本场景节点</div>
        <div class="editContent">
          <div>
            <el-checkbox-group v-model="childItemInfo.radioList[leftActiveIndex].linkArr">
              <el-checkbox v-for="(item, index) in childSceneList" :key="index" :label="item.number" style="display: block" class="editItem editItemRight">
                <div style="color: #333">{{item.label}}</div>
              </el-checkbox>
            </el-checkbox-group>
          </div>
        </div>
      </div>
    </div>
    </Modal>
  </div>
</template>
<script>
import draggable from 'vuedraggable'
export default {
  data () {
    return {
      // 控制radio方向字段，因为父组件传递过来的style是boolean，但是radio只能绑定string/number所以需要转换
      direction: 1,
      editAllState: false,
      textAreaList: '',
      loading: false,
      linkState: false,
      leftActiveIndex: 0,
      rightActiveIndex: null
    }
  },
  mounted () {
    if (this.childItemInfo.style === false) {
      this.direction = 1
    } else {
      this.direction = 2
    }
  },
  props: ['itemInfo', 'sceneList'],
  computed: {
    childItemInfo () {
      console.log(this.itemInfo, 'radio子组件收到的数据')
      return this.itemInfo
    },
    childSceneList () {
      let arr = []
      this.sceneList.forEach((value, index) => {
        if (value.number !== this.itemInfo.number) {
          arr.push(value)
        }
      })
      return arr
    },
    optionLinkState: {
      get () {
        let state = false
        if (this.itemInfo && this.itemInfo.radioList) {
          for (let i = 0; i < this.itemInfo.radioList.length; i++) {
            let value = this.itemInfo.radioList[i]
            if (value.linkArr.length > 0) {
              state = true
              break
            }
          }
        }
        return state
      },
      // 只有设置了set才可以修改computed的值，val就是新设置的值。
      set (val) {
        console.log(val, 'set值')
      }
    }
  },
  components: {
    draggable
  },
  methods: {
    requireStateChange () {
      console.log(this.childItemInfo)
      if (this.direction === 1) {
        this.childItemInfo.style = false
      } else {
        this.childItemInfo.style = true
      }
      this.$emit('changeItemInfo', this.childItemInfo)
    },
    deleteTagFun () {
      this.$emit('deleteItem', this.childItemInfo)
    },
    deleteOneItem (val, i) {
      this.childItemInfo.radioList.splice(i, 1)
    },
    // 添加选项
    addOption () {
      this.childItemInfo.radioList.push(
        {
          value: this.childItemInfo.radioList.length,
          label: '选项' + (this.childItemInfo.radioList.length + 1),
          linkArr: []
        }
      )
      this.$emit('changeItemInfo', this.childItemInfo)
    },
    // 批量编辑
    editAllOption () {
      this.editAllState = true
      this.textAreaList = ''
      // 组织回显数据
      if (this.childItemInfo.radioList) {
        this.childItemInfo.radioList.forEach((value, index) => {
          this.textAreaList += value.label + '\n'
        })
      }
    },
    // 批量编辑确定按钮
    submitEditChanne () {
      let arr = this.textAreaList.split('\n')
      arr.forEach((value, index) => {
        if (this.childItemInfo.radioList && this.childItemInfo.radioList[index] && this.childItemInfo.radioList[index].label) {
          this.childItemInfo.radioList[index].label = value
        }
      })
      this.$emit('changeItemInfo', this.childItemInfo)
      this.editAllState = false
    },
    // 批量编辑取消按钮
    cancelEditChannel () {
      this.editAllState = false
    },
    // 选项关联
    linkOption () {
      this.linkState = true
    },
    // 选项关联提交
    submitEditLink () {
      let state = false
      // 判断是否显示已关联
      console.log(this.childItemInfo, '信息')
      for (let i = 0; i < this.childItemInfo.radioList.length; i++) {
        let value = this.childItemInfo.radioList[i]
        if (value.linkArr.length > 0) {
          state = true
          break
        }
      }
      if (state) {
        this.optionLinkState = true
      } else {
        this.optionLinkState = false
      }
      // 选择列表，更新列表条数
      this.$emit('changeItemInfo', this.childItemInfo)
    },
    // 选项关联取消
    cancelEditLink () {},
    // 单机关联选项的左边条目
    selectLeftItem (item, index) {
      this.leftActiveIndex = index
      this.itemLinkArr = this.childItemInfo.radioList[index].linkArr
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
  .moveHandle {
    width: 6px;
    height: 14px;
    border-left: 1px dashed #D8D8D8;
    border-right: 1px dashed #D8D8D8;
    margin-right: 10px;
    cursor: pointer;
  }
  .editSceneLable {
    display: flex;
    align-items: center;
    margin-bottom: 5px;
    .rowOne {}
    .rowTwo {}
    .rowThree {
      flex: 1;
      color: #666;
      margin-left: -10px;
      >>> .ivu-input, .ivu-select, .ivu-radio-wrapper {
        color: #999;
      }
    }
    .rowFour {
      color: #ccc;
      font-size: 16px;
      margin-left: 15px;
      i {
        cursor: pointer;
      }
    }
  }
  .optionEdit {
    margin-left: 35px;
    color: #1c447a;
    margin-top: 5px;
    span {
      margin-right: 10px;
      cursor: pointer;
    }
  }
  .modalDec {
    color: #666;
    margin-bottom: 10px;
  }
  .editLinkModal {
    display: flex;
    align-items: flex-start;
    justify-content: flex-start;
    .editLinkLeft {
      flex: 1;
      margin-right: 10px;
    }
    .editLinkRight {
      flex: 2
    }
    .editContent {
      padding: 10px 0;
      border-radius:2px;
      border:1px solid rgba(227,227,227,1);
      margin-top: 15px;
      min-height: 100px;
      .editItem {
        padding: 9px 20px;
        cursor: pointer;
      }
      .editItem:hover {
        background: rgba(232,241,255,1);
      }
      .editItemRight checkbox:hover {
        background: rgba(232,241,255,0.5);
        padding: 8px 20px;
        border-top: 1px solid rgba(61,128,206,0.2);
        border-bottom: 1px solid rgba(61,128,206,0.2);
      }
    }
  }
  .linkActive {
    background:rgba(232,241,255,1);
  }
  .rightLinkActive {
    background: rgba(232,241,255,1);
    background: rgba(232,241,255,0.5);
    padding: 8px 20px;
    border-top: 1px solid rgba(61,128,206,0.2);
    border-bottom: 1px solid rgba(61,128,206,0.2);
  }
  .alreadySet {
    margin: 10px 0 0 0;
    padding: 0 10px;
    display: flex;
    border-radius:2px;
    height: 32px;
    line-height: 32px;
    border:1px solid rgba(227,227,227,1);
    .alreadySetLabel {
      flex: 1
    }
    .alreadySetIcon {
      width: 20px;
      flex-shrink: 1;
      font-size: 14px;
      color: #999;
    }
  }
</style>
