<template>
  <el-table :data="data" border style="width: 100%" :row-style="showTr">
    <el-table-column v-for="(column, index) in columns" :key="column.dataIndex" :label="column.text">
      <template scope="scope">
        <span v-if="!scope.row._edit">
          <span v-if="spaceIconShow(index)" v-for="(space, levelIndex) in scope.row._level" class="ms-tree-space"></span>
          <el-button style="padding:2px 3px;width:22px" type="text" class="button is-outlined is-primary is-small" v-if="toggleIconShow(index,scope.row)" @click="toggle(scope.$index)">
            <!-- 按钮的每种状态根据行展开数据来决定  -->
            <i v-if="scope.row._expanded" class="el-icon-caret-right" aria-hidden="true"></i>
            <i v-if="!scope.row._expanded" class="el-icon-caret-bottom" aria-hidden="true"></i>
          </el-button>
          <span v-else-if="index===0" class="ms-tree-space"></span>
          {{scope.row[column.dataIndex]}}
        </span>
        <span v-else>
            <input type="text" :value="scope.row[column.dataIndex]" @change="inputChange($event,column.dataIndex,scope.row)"></input>
        </span>
      </template>
    </el-table-column>
    <el-table-column label="操作" v-if="treeType === 'normal'" width="260">
      <template scope="scope">
        <!-- 点击按钮，将按钮的当前行数据和当前行的索引，以及数组和全局对象传过去  -->
        <span v-if="!scope.row._edit">
            <span v-if="item.show?item.show(scope.row):true" v-for="(item,i) in buttons" :key="i" size="small" type="text" class="control-btn" :class="item.type" @click="btnClick(item.text,scope.row,scope.$index,scope.store.states.data)">
            {{item.text}}
          </span>
        </span>
        <span v-else>
          <el-button  type="text" style="color:red;" @click = "save(scope.row,scope.$index,scope.store.states.data,scope)">保存</el-button>
        </span>
      </template>
    </el-table-column>
  </el-table>
</template>  
<script>  
import Utils from './dataTranslate.js'

let keys = [];

//递归获取所有子孙元素的总长度
var getLen = function(data) {
  let len = 0;
  (function childLen(data) {
    if (data.children) {
      len += data.children.length;
      for (let item of data.children) {
        childLen(item);
      }
    }
  })(data)
  return len
}

// import Vue from 'vue'  
export default {
  mounted() {
    let newArr = restoreData(this.datainit);
    this.$emit("syncTreeGridData", newArr);
  },
  props: {
    // 该属性是确认父组件传过来的数据是否已经是树形结构了，如果是，则不需要进行树形格式化  
    treeStructure: {
      type: Boolean,
      default: function() {
        return false
      }
    },
    // 这是相应的字段展示  
    columns: {
      type: Array,
      default: function() {
        return []
      }
    },
    // 这是数据源  
    datainit: {
      type: Array,
      default: function() {
        return []
      }
    },
    // 这个作用是根据自己需求来的，比如在操作中涉及相关按钮编辑，删除等，需要向服务端发送请求，则可以把url传过来  
    requestUrl: {
      type: String,
      default: function() {
        return ''
      }
    },
    // 这个是是否展示操作列  
    treeType: {
      type: String,
      default: function() {
        return 'normal'
      }
    },
    // 是否默认展开所有树  
    defaultExpandAll: {
      type: Boolean,
      default: function() {
        return false
      }
    },
    buttons: {
      type: Array,
      default: function() {
        return []
      }
    },
  },
  data(){
    return {
      dataSource:(()=>{
        return tanslate(this.datainit);
      })()
    }
  },
  computed: {
    // 格式化数据源  
    data: function() {
      let me = this
      if (me.treeStructure) {
        let data = Utils.treeToArray(me.dataSource, null, null, me.defaultExpandAll);
        return data
      }
      return me.dataSource
    },
  },

  methods: {
    // 显示行  
    showTr: function(row, index) {
      let show = (row._parent ? (row._parent._expanded && row._parent._show) : true)
      row._show = show
      return show ? '' : 'display:none;'
    },
    // 展开下级  
    toggle: function(trIndex) {
      let me = this
      let record = me.data[trIndex]
      record._expanded = !record._expanded
    },
    // 显示层级关系的空格和图标  
    spaceIconShow(index) {
      let me = this
      if (me.treeStructure && index === 0) {
        return true
      }
      return false
    },
    // 点击展开和关闭的时候，图标的切换  
    toggleIconShow(index, record) {
      let me = this
      if (me.treeStructure && index === 0 && record.children && record.children.length > 0) {
        return true
      }
      return false
    },
    btnClick(type, rowArr, index, arr) {
      if (type == "删除") {
        //在数据数组中删除
        let len = getLen(rowArr)
        arr.splice(index, 1 + len);
        //如果父亲存在，在父亲的子中也删除它
        if(rowArr._parent){
           let i = rowArr._parent.children.indexOf(rowArr)
           rowArr._parent.children.splice(i,1);
        }
        
        isleaf(arr); //将数组中的每个元素进行叶子判断
        let newArr = restoreData.call(this, arr);
        this.$emit("syncTreeGridData", newArr);
      }
      if (type == "编辑") {
        rowArr._edit = true;
      }
      if(type == "添加子菜单"){
        let level = rowArr["level"]; //获取当前level
        let obj = [{level:level+1,leaf:"YES"}]; //初步定义要增加的子元素
        
        // obj.level = level + 1;// obj.level = level + 1;
        let newdata= Utils.treeToArray(obj, rowArr, level,true);  //将子元素进行数据加工
        if(!rowArr.children || rowArr.children.length == 0){    //如果当前行没有子的话，给它添加children字段并且让它不是叶子节点
          rowArr.children = [];
          rowArr.leaf = "NO";
        } 
                 
        rowArr.children.push(newdata[0]);  //添加到当前行的孩子中
        arr.splice(index+1,0,newdata[0]); //添加到数据数组中
        newdata[0]._edit = true;    //可编辑状态
        rowArr._expanded = true;  //默认展开
      }
    },
    // 编辑保存
    save(rowArr, index, arr){
      rowArr._edit = false;
      let newArr = restoreData.call(this, arr);
      this.$emit("syncTreeGridData", newArr);
    },
    inputChange(e,colFiled,rowArr){
      rowArr[colFiled] = e.target.value;
    }
  }
}



function tanslate(data) {
  // 先把原来的keys存起来
  keys = Object.keys(data[0]);
  let arr = [];
  //记录之前层级的元素的索引
  let level = {};
  for (let i = 0; i < data.length; i++) {
    let item = data[i];
    level[item.level] = i;
    if (item.level === 1) {
      if (item.leaf == "NO" && !item.children) item.children = [];
      arr.push(item);
    }
    if (item.level !== 1) {
      if (item.leaf == "NO" && !item.children) item.children = [];
      data[level[item.level - 1]].children.push(item);
    }

  }
  return arr
}

//还原成最初的数据结构
function restoreData(dataArr) {
  let arr = [];
  dataArr.forEach(item => {
    let obj = {};
    for (let key of keys) {
      obj[key] = item[key]
    }
    arr.push(obj)
  });
  return arr
}

//将数组中的每个数组进行叶子判断
function isleaf(arr){
  arr.forEach((item)=>{
    if(item.children && item.children>0){
      item.leaf = "NO"
    }else{
      item.leaf = "YES"
    } 
  })
  
}
</script>  
<style scoped>
  /* 操作按钮的样式 */

  .control-btn {
    margin: 0 2px;
    cursor: pointer;
  }

  .control-btn:hover {
    opacity: 0.7;
  }

  span.success {
    color: #13ce66;
  }

  span.warning {
    color: #f7ba2a;
  }

  span.danger {
    color: #ff4949;
  }

  span.primary {
    color: #50bfff;
  }

  .el-button--text {
    background: transparent !important;
  }

  .ms-tree-space {
    position: relative;
    top: 1px;
    display: inline-block;
    font-family: 'Glyphicons Halflings';
    font-style: normal;
    font-weight: 400;
    line-height: 1;
    width: 25px;
    height: 14px;
  }

  .ms-tree-space::before {
    content: ""
  }

  table td {
    line-height: 26px;
  }
</style>  