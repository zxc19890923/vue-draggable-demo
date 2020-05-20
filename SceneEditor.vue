<template>
  <div class="content">
    <div class="sceneEditor">
      <div class="pullList">
        <div class="pullBaseTag">
          基础节点
        </div>
        <!-- 公共节点 -->
        <draggable
          :group="{name: 'scene', pull: 'clone'}"
          :sort="false"
          v-model="sceneWidgets"
          dragClass="pullDragClass"
          chosenClass="pullChosenClass"
          ghostClass="pullGhostClass"
        >
          <div v-for="(item, index) in sceneWidgets" :key="index" class="pullRow">
            <div class="pullItem" :style="{'margin-left': index % 2 === 1 ? '5px' : 0}">
              <div class="pullItemIcon">
                <img :src="require('@/assets/images/help-icon.png')" />
              </div>
              <div class="pullItemText">{{item.label}}</div>
            </div>
          </div>
        </draggable>
      </div>
      <!-- 编辑场景 -->
      <draggable
        :group="{name: 'scene', put: true}"
        :put='true'
        handle='.moveHandle'
        v-model="sceneEditor"
        @change="changeItem"
        @choose="chooseItem"
        class="putList"
        @add="addItem"
      >
        <div v-for="(item, index) in sceneEditor" :key="(index + 1) * 1000" class="putItem" :class="{'active': index === putAcitveIndex}" v-if="sceneEditor.length > 0" @click="putItemClick (item, index)">
          <div class="editSceneLable">
            <span class="moveHandle"></span>{{index}} <span>{{item.label}}</span>:
          </div>
          <div class="editSceneInput">
            <!-- 单行文本 -->
            <Input type="text" :name="item.type + index" :value="item.text" v-if="item.type === 'text'" @on-change="inputChange($event, item, index)" />
            <!-- 多行文本 -->
            <Input type="textarea" :name="item.type + index" :value="item.text" v-if="item.type === 'textarea'" @on-change="inputChange($event, item, index)" />
            <!-- 单选 -->
            <RadioGroup :value="item.text" :vertical="item.style" v-if="item.type === 'radio'" @on-change="inputChange($event, item, index)">
              <Radio v-for="(val, i) in item.radioList" :label="val.value" :key="i">{{val.label}}</Radio>
            </RadioGroup>
            <!-- 多选 -->
            <CheckboxGroup :value="item.text" v-if="item.type === 'checkbox'" @on-change="inputChange($event, item, index)">
              <Checkbox  v-for="(val, i) in item.checkboxList" :label="val.value" :key="i">{{val.label}}</Checkbox>
            </CheckboxGroup>
            <!-- 下拉选项 -->
            <Select :name="item.type + index" v-if="item.type === 'select'" @on-change="inputChange($event, item, index)">
              <Option v-for="(val, i) in item.selectList" :value="val.value" :key="i">{{item.label}}</Option>
            </Select>
            <!-- 日期组件 -->
            <DatePicker :type="item.type" :name="item.type + index" :placeholder="item.placeholder" :value="item.text" v-if="item.type === 'date'" @on-change="inputChange($event, item, index)" ></DatePicker>
            <!-- 链接地址 -->
            <div style="position: relative">
              <Input type="text" prefix="ios-link" :name="item.type + index" :placeholder="item.placeholder" :value="item.text" v-if="item.type === 'link'" @on-change="inputChange($event, item, index)" />
              <div>{{item.title}}</div>
            </div>
            <linkage @getLawyerListInfo="inputChange($event, item, index)" v-if="item.type === 'place'"></linkage>
          </div>
        </div>
        <!-- <div class="noTag" v-if="sceneEditor.length === 0">
          <div>从左侧拖拽节点</div>
        </div> -->
      </draggable>
      <div class="editItemInfo">
        <div v-if="inputItemObj.type === 'text'">
          <div class="editTag">
            <span class="editTagLabel">节点名称</span>
            <span class="editTagNumber">4/10</span>
          </div>
          <div class="editTag">
            <Input type="text" v-model="inputItemObj.label" :maxlength="10" />
          </div>
          <div class="editTag">
            <span class="editTagLabel">格式</span>
          </div>
          <div class="editTag">
            <Select v-model="inputItemObj.classify">
              <Option value='text'>文本</Option>
              <Option value='number'>数字</Option>
              <Option value='phone'>电话号码</Option>
              <Option value='email'>邮箱</Option>
            </Select>
          </div>
          <div class="editTag">
            <span class="editTagLabel">校验</span>
          </div>
          <div class="editTag">
            <Checkbox v-model="inputItemObj.required" @on-change="requireStateChange">设为必填项</Checkbox>
          </div>
        </div>
      </div>
    </div>
    <div style="height: 30vh; overflow: scroll; margin: 20px; padding-bottom: 100px;">
      <div v-for="(item, index) in sceneWidgets" :key="(index + 1000) * 1000">
        <b style="color: blue">{{item}}</b>
      </div>
      <div v-for="(item, index) in sceneEditor" :key="(index + 10000) * 10000">
        <b style="color: red">{{item}}</b>
      </div>
    </div>
  </div>
</template>
<script>
import draggable from 'vuedraggable'
import Linkage from '@/components/common/Linkage'
export default {
  name: 'sceneEditor',
  data () {
    return {
      putAcitveIndex: 0,
      putActiveItem: {},
      showHtml: '',
      sceneWidgets: [
        {
          type: 'text',
          classify: 'text', // 文字、数字、email、phone等
          icon: '',
          label: '单行节点',
          placeholder: '请输入文本',
          text: '',
          required: false
        },
        {
          type: 'textarea',
          icon: '',
          label: '多行节点',
          placeholder: '请输入文本',
          text: ''
        },
        {
          type: 'radio',
          icon: '',
          label: '单选',
          placeholder: '',
          text: '',
          style: false,
          radioList: [
            {
              value: 1,
              label: 'Mac'
            },
            {
              value: 2,
              label: 'Ipad'
            }
          ]
        },
        {
          type: 'checkbox',
          icon: '',
          label: '多选',
          placeholder: '',
          text: [],
          checkboxList: [
            {
              value: 1,
              label: 'Mac'
            },
            {
              value: 2,
              label: 'Ipad'
            }
          ]
        },
        {
          type: 'select',
          icon: '',
          label: '下拉选项',
          placeholder: '',
          text: '',
          selectList: [
            {
              value: 1,
              label: '测试'
            }
          ]
        },
        {
          type: 'date',
          icon: '',
          label: '日期时间',
          placeholder: '',
          text: ''
        },
        {
          type: 'link',
          icon: 'ios-link',
          label: '链接地址',
          placeholder: '',
          text: 'http://www.baidu.com',
          title: '百度一下'
        },
        {
          type: 'place',
          icon: '',
          label: '省市区县',
          placeholder: '',
          text: ''
        }
      ],
      sceneEditor: [
      ],
      // 单行文本右边编辑数据
      inputItemObj: {
        label: '',
        type: '',
        required: false
      }
    }
  },
  components: {
    draggable,
    Linkage
  },
  methods: {
    addItem (evt) {
      console.log(evt, 'add')
    },
    chooseItem (evt) {
      console.log(evt, 'choose')
    },
    changeItem (evt) {
      console.log(evt, 'change')
      // 拖动添加的时候，初始化选中index,和选中的item值
      if (evt && evt.added) {
        this.putAcitveIndex = evt.added.newIndex
        // 深拷贝一行数据，如果不这样的话，会影响节点列表中的json数据格式
        this.inputItemObj = JSON.parse(JSON.stringify(evt.added.element))
      }
      // 拖动排序的时候，修改index，和选中的item值
      if (evt && evt.moved) {
        this.putAcitveIndex = evt.moved.newIndex
        this.inputItemObj = JSON.parse(JSON.stringify(evt.moved.element))
      }
    },
    // 组合判断数据
    searchDataType (item) {},
    // 每行被单击
    putItemClick (item, index) {
      console.log(item, index)
      // 选中哪一行
      this.putAcitveIndex = index
      // 深拷贝一行数据，如果不这样的话，会影响节点列表中的json数据格式
      this.inputItemObj = JSON.parse(JSON.stringify(this.sceneEditor[this.putAcitveIndex]))
    },
    inputChange (event, item, index) {
      console.log(event, item, index, 'event')
      // 获取输入的内容
      // let val = event.target.value
      let val = ''
      if (item.type === 'select' || item.type === 'checkbox' || item.type === 'radio' || item.type === 'date') {
        val = event
      } else if (item.type === 'text' || item.type === 'textarea' || item.type === 'link') {
        val = event.target.value
      } else if (item.type === 'place') {
        console.log(event)
      }
      // 深拷贝对象，如果指直接操作 this.sceneEditor相同的输入框数据一起变化
      let arr = JSON.parse(JSON.stringify(this.sceneEditor))
      // 修改text字段的值
      arr[index].text = val
      // 赋值修改编辑列表
      this.sceneEditor = arr
      console.log(arr)
    },
    // 必填项状态修改
    requireStateChange () {
      console.log(this.putAcitveIndex, this.inputItemObj)
      this.sceneEditor[this.putAcitveIndex] = this.inputItemObj
    }
  }
}
</script>
<style lang="scss" scoped>
  .sceneEditor {
    background: #fff;
    display: flex;
    .pullList, .putList {
      border-right: 1px solid #d8d8d8;
      padding: 20px;
      height: 100vh;
      .putItem {
        margin-bottom: 20px;
        display: flex;
        align-items: center;
        .editSceneLable {
          width: 100px;
          overflow: hidden;
        }
        .editSceneInput {
          flex: 1;
        }
      }
      .putItem:hover {
        background: #E8F1FF
      }
      .active {
        background: #E8F1FF
      }
    }
    .pullList {
      flex-shrink: 1;
      width: 290px;
      .pullBaseTag {
        font-size: 14px;
        color: #333;
        margin-bottom: 10px;
        font-weight: 600;
      }
      .pullRow {
        float: left;
        .pullItem {
          cursor: pointer;
          display: flex;
          width: 120px;
          align-items: center;
          padding: 10px;
          margin-bottom: 5px;
          background: #f4f6fc;
          .pullItemIcon {
            img {
              width: 12px;
              height: 12px;
              margin-right: 5px;
            }
          }
          .pullItemText {
            font-size: 12px;
            color: #333;
          }
        }
      }
    }
    .editItemInfo {
      width: 300px;
      flex-shrink: 1;
      padding: 20px;
      .editTag {
        margin-bottom: 10px;
        .editTagLabel {
          font-weight: 600;
          margin-right: 5px;
        }
        .editTagNumber {
          color: #999999;
        }
      }
    }
    .putList {
      flex: 1;
      clear: both;
      // 拖到编辑场景中的样式
      .pullChosenClass {
        .pullItem {
          cursor: pointer;
          display: flex;
          width: 100%;
          align-items: center;
          padding: 10px;
          margin-bottom: 20px;
          background: #f4f6fc;
          .pullItemIcon {
            img {
              width: 12px;
              height: 12px;
              margin-right: 5px;
            }
          }
          .pullItemText {
            font-size: 12px;
            color: #333;
          }
        }
      }
    }
    .moveHandle {
      display: inline-block;
      width: 6px;
      height: 14px;
      border-left: 1px dashed #D8D8D8;
      border-right: 1px dashed #D8D8D8;
      margin-right: 10px;
      cursor: pointer;
    }
  }
  .noTag {
    flex: 1;
    display: flex;
    margin: 20px;
    height: 90vh;
    align-items: center;
    justify-content: center;
    color: #999;
    border: 1px dashed rgba(28,68,122,0.2);;
  }
</style>
