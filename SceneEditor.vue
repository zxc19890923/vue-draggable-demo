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
          <div v-for="(item, index) in sceneWidgets" :key="index" class="pullRow" :class="item.type">
            <div class="pullItem">
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
        handle='.moveHandle'
        v-model="sceneEditor"
        @change="changeItem"
        @choose="chooseItem"
        class="putList"
        @add="addItem"
      >
        <div v-for="(item, index) in sceneEditor" :key="(index + 1) * 1000" :class="{'active': index === putAcitveIndex}" v-if="sceneEditor.length > 0" @click="putItemClick (item, index)">
          <div class="putItem" v-if="item.showState !== false">
            <div class="editSceneLable">
              <span class="moveHandle"></span><span>{{item.label}}</span>
            </div>
            <div class="editSceneInput">
              <!-- 单行文本 -->
              <Input type="text" :name="item.type + index" :placeholder="item.placeholder" :value="item.text" v-if="item.type === 'text'" @on-change="inputChange($event, item, index)" />
              <!-- 多行文本 -->
              <Input type="textarea" :name="item.type + index" :placeholder="item.placeholder" :value="item.text" v-if="item.type === 'textarea'" @on-change="inputChange($event, item, index)" />
              <!-- 单选 -->
              <RadioGroup v-model="item.text" :vertical="item.style" v-if="item.type === 'radio'" @on-change="inputChange($event, item, index)">
                <Radio v-for="(val, i) in item.radioList" :label="val.value" :key="i"><span style="color: #666">{{val.label}}</span></Radio>
              </RadioGroup>
              <!-- 多选 -->
              <el-checkbox-group v-model="item.text" v-if="item.type === 'checkbox'" @change="inputChange($event, item, index)">
                <el-checkbox v-for="(val, i) in item.checkboxList" :label="val.value" :key="i"><span style="color: #666">{{val.label}}</span></el-checkbox>
              </el-checkbox-group>
              <!-- 下拉选项 -->
              <Select :name="item.type + index" :placeholder="item.placeholder" v-if="item.type === 'select'" @on-change="inputChange($event, item, index)">
                <Option v-for="(val, i) in item.selectList" :value="val.value" :key="i">{{item.label}}</Option>
              </Select>
              <!-- 日期组件 -->
              <div v-if="item.type === 'date'">
                <el-date-picker :type="'date'" :name="item.type + index" :placeholder="item.placeholder" v-model="item.text" v-if="item.dateType === 1 && item.limit === 1" @change="inputChange($event, item, index)" ></el-date-picker>
                <el-date-picker :type="'daterange'" range-separator="至" start-placeholder="开始时间" end-placeholder="结束时间" :name="item.type + index" :placeholder="item.placeholder" v-model="item.text" v-if="item.dateType === 1 && item.limit === 2" @change="inputChange($event, item, index)" ></el-date-picker>

                <el-date-picker :type="'datetime'" :name="item.type + index" :placeholder="item.placeholder" v-model="item.text" v-if="item.dateType === 2 && item.limit === 1" @change="inputChange($event, item, index)" ></el-date-picker>
                <el-date-picker :type="'datetimerange'" range-separator="至" start-placeholder="开始时间" end-placeholder="结束时间" :name="item.type + index" :placeholder="item.placeholder" v-model="item.text" v-if="item.dateType === 2 && item.limit === 2" @change="inputChange($event, item, index)" ></el-date-picker>
              </div>
              <!-- 上传文件 -->
              <upload-file :accept="''" :showFileList='false' :buttonText="'点击或者拖拽到此上传'" :multiple="true" :fileList="[]" v-if="item.type === 'upload'"></upload-file>
              <!-- 链接地址 -->
              <div style="position: relative">
                <Input type="text" prefix="ios-link" :name="item.type + index" :placeholder="item.placeholder" :value="item.text" v-if="item.type === 'link'" @on-change="inputChange($event, item, index)" />
                <div style="color: #666;">{{item.title}}</div>
              </div>
              <!-- 省市区县 -->
              <province-city-county-linkage @getLinageValue="inputChange($event, item, index)" v-if="item.type === 'place'"></province-city-county-linkage>
            </div>
          </div>
        </div>
        <!-- <div class="noTag" v-if="sceneEditor.length === 0">
          <div>从左侧拖拽节点</div>
        </div> -->
      </draggable>
      <div class="editItemInfo">
        <div class="tab">
          <!-- 因为默认下边框是1px，选中的时候是2px，为了解决选中tab页面抖动，所以添加一下样式（padding-bottom: 18/17 px） -->
          <div class="tagTab" :style="{'border-bottom': activeTab === 'tagTab' ? '2px solid #1C447A': '', 'padding-bottom': activeTab === 'tagTab' ? '17px' : '18px', 'color': activeTab === 'tagTab' ? '#333' : '#999'}" @click="tabItemClick(1)">节点设置</div>
          <div class="sceneTab" :style="{'border-bottom': activeTab === 'sceneTab' ? '2px solid #1C447A': '', 'padding-bottom': activeTab === 'sceneTab' ? '17px' : '18px', 'color': activeTab === 'sceneTab' ? '#333' : '#999'}" @click="tabItemClick(2)">场景设置</div>
        </div>
        <div class="editItemInfoContent">
          <!-- 右侧节点编辑栏 -->
          <div v-if="activeTab === 'tagTab'" class="tabContent">
            <!--
              参数说明：
              itemInfo: 每一条节点的json数据
              changeItemInfo() 编辑每一条节点的属性时候触发
              deleteItem() 删除一条节点数据触发
            -->
            <!-- text -->
            <text-edit v-if="inputItemObj.type === 'text'" :itemInfo="inputItemObj" @changeItemInfo="requireStateChange" @deleteItem="deleteTagFun"></text-edit>
            <!-- textarea -->
            <textarea-edit v-if="inputItemObj.type === 'textarea'" :itemInfo="inputItemObj" @changeItemInfo="requireStateChange" @deleteItem="deleteTagFun"></textarea-edit>
            <!-- radio -->
            <radio-edit v-if="inputItemObj.type === 'radio'" :itemInfo="inputItemObj" :sceneList="sceneEditor" @changeItemInfo="requireStateChange" @deleteItem="deleteTagFun" @updateLinkList="updateEditorList"></radio-edit>
            <!-- linkage -->
            <linkage-edit v-if="inputItemObj.type === 'place'" :itemInfo="inputItemObj" @changeItemInfo="requireStateChange" @deleteItem="deleteTagFun"></linkage-edit>
            <!-- uploadEdit -->
            <upload-edit v-if="inputItemObj.type === 'upload'" :itemInfo="inputItemObj" @changeItemInfo="requireStateChange" @deleteItem="deleteTagFun"></upload-edit>
            <!-- link链接文字 -->
            <link-edit v-if="inputItemObj.type === 'link'" :itemInfo="inputItemObj" @changeItemInfo="requireStateChange" @deleteItem="deleteTagFun"></link-edit>
            <!-- dateEdit -->
            <date-edit v-if="inputItemObj.type === 'date'" :itemInfo="inputItemObj" @changeItemInfo="requireStateChange" @deleteItem="deleteTagFun"></date-edit>
          </div>
          <!-- 右侧场景编辑栏 -->
          <div v-if="activeTab === 'sceneTab'" class="tabContent">
            <div class="editTagText">
              <span class="editTagLabel">场景名称</span>
              <span class="editTagNumber">4/10</span>
            </div>
            <div class="editTag">
              <Input type="text" v-model="sceneSet.name" :maxlength="10" />
            </div>
            <div class="editTagText">
              <span class="editTagLabel">场景负责人</span>
            </div>
            <div class="editTag">
              <div class="editTag">
                <Select v-model="sceneSet.principal">
                  <Option value='number'>张</Option>
                  <Option value='phone'>旭</Option>
                  <Option value='email'>超</Option>
                </Select>
              </div>
            </div>
            <div class="editTagText">
              <span class="editTagLabel">场景可见团队</span>
              <span class="editTagNumber">*可多选</span>
            </div>
            <div class="editTag">
              <el-checkbox-group v-model="sceneSet.team">
                <el-checkbox label="1">
                  <span style="color: #333">所有案件参与团队</span>
                </el-checkbox>
                <div style="height: 10px"></div>
                <el-checkbox label="2">
                  <span style="color: #333">指定团队</span>
                </el-checkbox>
              </el-checkbox-group>
            </div>
          </div>
        </div>
        <div class="deleteScene" v-if="activeTab === 'sceneTab'">
          <button @click="deleteSceneFun">删除场景</button>
        </div>
        <div class="deleteScene" v-if="activeTab === 'tagTab' && this.sceneEditor.length > 0">
          <button @click="deleteTagFun">删除节点</button>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import draggable from 'vuedraggable'
import Linkage from '@/components/common/Linkage'
import UploadFile from '@/components/common/UploadFile'
import ProvinceCityCountyLinkage from '@/components/common/element-ui/ProvinceCityCountyLinkage'
import TextEdit from '@/components/scene/TextEdit'
import TextareaEdit from '@/components/scene/TextareaEdit'
import RadioEdit from '@/components/scene/RadioEdit'
import LinkageEdit from '@/components/scene/LinkageEdit'
import UploadEdit from '@/components/scene/UploadEdit'
import LinkEdit from '@/components/scene/LinkEdit'
import DateEdit from '@/components/scene/DateEdit'
export default {
  name: 'sceneEditor',
  data () {
    return {
      putAcitveIndex: 0,
      putActiveItem: {},
      showHtml: '',
      radioOrCheckboxSelectItem: {},
      sceneWidgets: [
        {
          type: 'text',
          classify: 'text', // 文字、数字、email、phone等
          icon: '',
          label: '单行节点',
          placeholder: '请输入文本',
          text: '',
          required: false,
          number: null
        },
        {
          type: 'textarea',
          icon: '',
          label: '多行节点',
          placeholder: '请输入文本',
          text: '',
          number: null,
          required: false
        },
        {
          type: 'radio',
          icon: '',
          label: '单选',
          placeholder: '',
          text: '',
          number: null,
          style: false,
          required: false,
          radioList: [
            {
              value: 0,
              label: '选项1',
              linkArr: []
            },
            {
              value: 1,
              label: '选项2',
              linkArr: []
            },
            {
              value: 2,
              label: '选项3',
              linkArr: []
            }
          ]
        },
        {
          type: 'checkbox',
          icon: '',
          label: '多选',
          placeholder: '',
          text: [],
          number: null,
          required: false,
          checkboxList: [
            {
              value: 0,
              label: '选项1'
            },
            {
              value: 1,
              label: '选项2'
            },
            {
              value: 2,
              label: '选项3'
            }
          ]
        },
        {
          type: 'select',
          icon: '',
          label: '下拉选项',
          placeholder: '请选择',
          text: '',
          number: null,
          required: false,
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
          placeholder: '请选择',
          text: '',
          number: null,
          required: false,
          limit: 1, // 1：时间点，2：时间段
          dateType: 2 // 1：日期，2：日期+时间
        },
        {
          type: 'upload',
          icon: '',
          label: '上传文件',
          placeholder: '文件名称',
          text: '',
          required: false,
          number: null
        },
        {
          type: 'link',
          icon: 'ios-link',
          label: '链接地址',
          placeholder: '请输入文本',
          text: '',
          required: false,
          number: null,
          title: '链接说明文字'
        },
        {
          type: 'place',
          icon: '',
          label: '省市区县',
          placeholder: '',
          text: {province: '', city: '', county: '', address: ''},
          required: false,
          number: null
        }
      ],
      sceneEditor: [
      ],
      // 单行文本右边编辑数据
      inputItemObj: {
        label: '',
        type: '',
        text: null,
        required: false,
        number: null
      },
      // 激活节点设置、场景设置tab页
      activeTab: 'sceneTab',
      sceneSet: {
        name: '',
        principal: '',
        team: []
      },
      initIndex: 0
    }
  },
  components: {
    draggable,
    Linkage,
    ProvinceCityCountyLinkage,
    UploadFile,
    TextEdit,
    TextareaEdit,
    RadioEdit,
    LinkageEdit,
    UploadEdit,
    LinkEdit,
    DateEdit
  },
  methods: {
    addItem (evt) {
      console.log(evt, 'add')
      this.initIndex++
      let obj = JSON.parse(JSON.stringify(this.sceneWidgets[evt.oldIndex]))
      // 往编辑器中拖动数据的的时候，给每条数据添加编号，这样方便当选复选框关联显示其他输入框，index会受到排序和删除的影响。
      // obj.number = this.sceneEditor.length
      obj.number = this.initIndex
      this.sceneEditor.splice(evt.newIndex, 1, obj)
    },
    chooseItem (evt) {
      console.log(evt, 'choose')
    },
    changeItem (evt) {
      console.log(evt, 'change')
      // 拖动添加的时候，初始化选中index,和选中的item值
      if (evt && evt.added) {
        this.activeTab = 'tagTab'
        // 设置编辑框里元素的编号，每次拖动一个编号加一
        console.log(this.sceneEditor.length, 'length')
        this.putAcitveIndex = evt.added.newIndex
        // 深拷贝一行数据，如果不这样的话，会影响节点列表中的json数据格式
        this.inputItemObj = JSON.parse(JSON.stringify(evt.added.element))
      }
      // 拖动排序的时候，修改index，和选中的item值
      if (evt && evt.moved) {
        this.activeTab = 'tagTab'
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
      this.activeTab = 'tagTab'
      // 深拷贝一行数据，如果不这样的话，会影响节点列表中的json数据格式
      this.inputItemObj = JSON.parse(JSON.stringify(this.sceneEditor[this.putAcitveIndex]))
    },
    inputChange (event, item, index) {
      console.log(event, item, index, 'event')
      // 获取输入的内容
      // let val = event.target.value
      let val = ''
      if (item.type === 'select' || item.type === 'checkbox' || item.type === 'radio' || item.type === 'date' || item.type === 'place') {
        val = event
      } else if (item.type === 'text' || item.type === 'textarea' || item.type === 'link') {
        val = event.target.value
      }
      // 深拷贝对象，如果指直接操作 this.sceneEditor相同的输入框数据一起变化
      let arr = JSON.parse(JSON.stringify(this.sceneEditor))
      // 修改text字段的值
      arr[index].text = val
      // 子组件监听的是inputItemObj,如果想联动必须改变，在单选框值改变的时候渲染到编辑框右边的设置选择中
      this.inputItemObj = JSON.parse(JSON.stringify(arr[index]))
      // 赋值修改编辑列表
      this.sceneEditor = arr
      console.log(arr)

      // 如果是radio或者checkbox控制关联选项的隐藏显示，特殊处理。
      if (item.type === 'radio' || item.type === 'checkbox') {
        // 查看关联的项目
        console.log(item, '每一行的数据')
        this.radioOrCheckboxSelectItem = {}
        for (let i = 0; i < item.radioList.length; i++) {
          let value = item.radioList[i]
          if (value.value === item.text) {
            this.radioOrCheckboxSelectItem = value
            break
          }
        }
        console.log(this.radioOrCheckboxSelectItem)
        this.updateEditorList()
      }
    },
    updateEditorList () {
      // linkArr：就是关了的节点的number值，在sceneEditor中查找linkArr中是否有匹配字段
      this.sceneEditor.forEach((value, index) => {
        if (value.number !== this.inputItemObj.number) {
          this.$set(value, 'showState', false)
        }
      })
      this.radioOrCheckboxSelectItem.linkArr.forEach((value, index) => {
        this.sceneEditor.forEach((val, i) => {
          if (val.number === value) {
            val.showState = true
          }
        })
      })
    },
    // 编辑标签内容
    requireStateChange (item) {
      console.log(this.putAcitveIndex, item, '子组件传递过来的值')
      this.sceneEditor.splice(this.putAcitveIndex, 1, item)
    },
    // 删除节点
    deleteTagFun () {
      this.sceneEditor.splice(this.putAcitveIndex, 1)
      if (this.sceneEditor.length === 0) {
        this.activeTab = 'sceneTab'
      }
      // 删除一条以后默认选中第一条编辑框
      this.putAcitveIndex = 0
    },
    // tab页被单击
    tabItemClick (num) {
      console.log(num)
      if (num === 1) {
        this.activeTab = 'tagTab'
      } else if (num === 2) {
        this.activeTab = 'sceneTab'
      }
    },
    // 删除场景
    deleteSceneFun (item) { console.log(console.log(item)) }
  }
}
</script>
<style lang="scss" scoped>
  .sceneEditor {
    background: #fff;
    display: flex;
    .pullList, .putList {
      border-right: 1px solid #d8d8d8;
      height: 100vh;
      overflow-y: auto;
    }
    .pullList {
      flex-shrink: 1;
      padding: 20px;
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
      .pullRow:nth-child(even) {
        margin-left: 5px;
      }
    }
    .editItemInfo {
      width: 300px;
      height: 100vh;
      position: relative;
      flex-shrink: 1;
      .tab {
        display: flex;
        align-items: center;
        justify-content: center;
        .tagTab, .sceneTab {
          flex: 1;
          border-bottom: 1px solid #D8D8D8;
          text-align: center;
          font-size: 14px;
          padding: 18px 0;
          cursor: pointer;
        }
      }
      .tabContent {
        padding: 20px 20px 0 20px;
      }
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
      .editItemInfoContent {
        max-height: calc(100vh - 130px);
        overflow-y: auto;
      }
      .deleteScene {
        margin: 0 20px 20px 20px;
        button {
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
      }
    }
    .putList {
      flex: 1;
      clear: both;
      padding: 15px 0;
      .putItem {
        margin-bottom: 20px;
        padding: 5px 20px;
        display: flex;
        align-items: flex-start;
        .editSceneLable {
          width: 100px;
          overflow: hidden;
          line-height: 30px;
        }
        .editSceneInput {
          flex: 1;
          line-height: 30px;
        }
      }
      .putItem:hover {
        background: #E8F1FF
      }
      .active {
        background: #E8F1FF
      }
      // 拖动的时候，拖入编辑框根据不同的组件设置不同高度占位
      .textarea {
        .pullItem {
          height: 60px;
        }
      }
      // 拖到编辑场景中的样式
      .pullChosenClass {
        .pullItem {
          margin-left: 0;
          cursor: pointer;
          display: flex;
          width: 100%;
          align-items: center;
          padding: 10px;
          margin-bottom: 20px;
          background:rgba(248,206,85,0.05);
          border: 1px dashed rgba(248,206,85,1);
          .pullItemIcon {
            img {
              width: 12px;
              height: 12px;
              margin-right: 5px;
            }
            // display: none;
          }
          .pullItemText {
            font-size: 12px;
            color: #333;
            // display: none;
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
