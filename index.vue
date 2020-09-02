<!-- v1.0.2
  2020/6/30：优化判断逻辑
  2020/7/1:优化懒加载
 -->
<template>
  <el-table class="treeTable" :ref="mqy" :default-expand-all='defaultExpandAll' :data="treeData" :indent="0"
    :tree-props="treeProps" :row-key="rowKey" :border="border" highlight-current-row :height="height" @row-click="handleRowClick"
    @expand-change="expandChange" @current-change="handleCurrentChange"  @selection-change="handleSelectionChange" :reserve-selection='true'>
    <el-table-column v-if="showCheckbox" label="#" width="45" fixed="left">
      <template slot="header" slot-scope="scope">
        <el-checkbox :indeterminate="isIndeterminate" v-model="checkAll" @change="handleCheckAllChange"></el-checkbox>
      </template>
      <template slot-scope="scope">
        <el-checkbox :indeterminate="scope.row.indeterminate" v-model="scope.row.checked" @click.stop.native="handleCheckChange($event,scope.row)"></el-checkbox>
      </template>
    </el-table-column>
    <el-table-column :label="idxReplace.header" :width="idxReplace.width.length>0?idxReplace.width:(100+(treeLevel-1)*30)"
      header-align="center" fixed="left" :show-overflow-tooltip="idxReplace.width.length>0?true:false">
      <template slot-scope="scope">
        <span style="position: relative;">

          <span v-for="(item,index) in (scope.row.idx.split('-').length)" :key="index">
            <span v-if="showLine">
              <span v-if="index==(scope.row.idx.split('-').length-1)">
                <span @click.stop="openTree($event,scope.row)" style="padding:0px 4px;cursor: pointer;position: relative;top: 3px;"
                  v-if="scope.row[treeProps.children]&&scope.row[treeProps.children].length>0||scope.row[treeProps.hasChildren]">
                  <img v-if="!scope.row.expand" src="./static/icon/加号-带线.png" />
                  <img v-else src="./static/icon/减号-带线.png" style="padding-top: 0px;" />
                </span>
                <span v-else style="padding-left:8px;position: relative; top: 3px;">
                  <!--当前children没有或者为0，且不是当前父子级中最后一个。则显示t图标-->
                  <img v-if="scope.row.num!=scope.row.parentNum" src="./static/icon/T形线.png" />
                  <!--当前children没有或者为0，且是当前父子级中最后一个。则显示l图标-->
                  <img v-else src="./static/icon/直角.png" style="position: relative;top:-7px" />
                </span>
              </span>
              <img v-else src="./static/icon/直线.png" style="padding:0px 8px;position: relative; top: 3px;" />
            </span>
            <span v-else style="padding:0px 8px;"></span>
          </span>

          <span v-if="(scope.row[treeProps.children]&&scope.row[treeProps.children].length>0)||(scope.row[treeProps.hasChildren])"
            @click.stop="openTree($event,scope.row)" style="margin-left:-6px;">
            <span style="position: relative;top: 3px;cursor: pointer;">
              <img v-if="scope.row.expand" width="15px" height="15px" src="./static/icon/文件夹-打开.png" />
              <img v-if="!scope.row.expand&&!scope.row.loading" width="15px" height="15px" src="./static/icon/文件夹-收起.png" />
            </span>
            <i v-if="!scope.row.expand&&scope.row.loading" class="el-icon-loading" style="color: #949494;"></i>
          </span>
          <span v-else>
            <img style="margin-left: -2px; position: relative;top: 2px;" width="13px" height="15px" src="./static/icon/文件.png" />
          </span>
          <span>{{scope.row[idxReplace.value]}}</span>
        </span>
      </template>
    </el-table-column>
    <slot name="dataColumn">
    </slot>
  </el-table>
</template>

<script>
  const {
    log
  } = console
  export default {
    name: 'Index',
    props: {
      //表格的唯一标志
      mqy: {
        type: String,
        default: new Date().getTime().toString()+Math.random().toString()
      },
      //表格的高度
      height: {
        type: String,
        default: '500px'
      },
      //是否展示表格边框
      border: {
        type: Boolean,
        default: true
      },
      //idx的代替值
      idxReplace: {
        type: Object,
        default: () => ({
          header: '序号',
          value: 'idx',
          width: ''
        })
      },
      //是否全部打开,和懒加载相对应，如果没使用懒加载，则默认全部打开，如果使用了懒加载，则默认全部关闭
      defaultExpandAll: {
        type: Boolean,
        default: !this.lazy
      },
      //是否展示分线栏
      showLine: {
        type: Boolean,
        default: true
      },
      //是否展示复选框
      showCheckbox: {
        type: Boolean,
        default: true
      },
      //是否严格的遵守父子节点互相关联,默认不关联
      checkStrictly: {
        type: Boolean,
        default: false
      },
      //是否开启懒加载
      lazy: {
        type: Boolean,
        default: false
      },
      //懒加载函数
      load: {
        type: Function,
        default: () => (row, resolve) => {}
      },
      //展示的树参数  （树形）（必传）
      treeData: {
        type: Array,
        required: true,
        default: () => []
      },
      //树形参数的对象（element表格组件自带的）
      treeProps: {
        type: Object,
        default: () => ({
          children: 'children',
          hasChildren: 'hasChildren'
        })
      },
      // 根据什么参数判断是否是树（element表格组件自带的）
      rowKey: {
        type: String,
        required: true,
        default: ''
      },
      // 父id的名称（key）（必传）
      parentKey: {
        type: String,
        required: true,
        default: ''
      },
      // 父id的名称（val）（必传）
      topParentVal: {
        type: String | null,
        required: true,
        default: () => []
      },
      //默认勾选的数据，数组类型为字符串，传递id即可
      checkList: {
        type: Array,
        default: () => []
      },
      //默认高亮
      defaultHeightRowId: {
        type: String,
        default: ''
      },
    },
    data() {
      return {
        //当前选择的所有行
        selectRows: [],
        //当前选择行
        selectRow: {},
        treeLevel: 0, //数的实际层数
        initTreeLevel: 0, //数的层数
        idxObj: {},
        treeTotal: 0, //树的总数

        checkAll: false, //是否全部选中
        isIndeterminate: false

      }
    },
    computed: {

    },
    created() {

    },
    watch: {
      //监听到treeData的变化，就进行初始化
      treeData() {
        if (this.treeData.length > 0) {
          this.init();
        }
      },
      //监听已经选择的
      selectRows() {
        if (this.selectRows.length == 0) {
          this.checkAll = false
          this.isIndeterminate = false
        } else if (this.selectRows.length < this.treeTotal) {
          this.checkAll = false
          this.isIndeterminate = true
        } else if (this.selectRows.length == this.treeTotal) {
          this.checkAll = true
          this.isIndeterminate = false
        }
      },
    },
    mounted() {

    },
    methods: {
      init() {
        this.treeTotal = 0
        //获得每一行的索引
        this.treeData.forEach((e, i) => {
          this.setVisible(e, i)
        })

        //计算层级
        this.treeData.forEach((e, i) => {
          this.initTreeLevel = 0
          this.getTreeLevel(e)
          if (this.initTreeLevel > this.treeLevel) {
            this.treeLevel = this.initTreeLevel
          }
        })

        //默认选中的数据
        this.setSelectList()

        //默认高亮
        this.setDefaultHeightRow();
      },
      setSelectList() {
        if (this.checkList.length > 0) {
          this.checkList.forEach(e => {
            var row = this.getParentdata(e, this.treeData)
            this.handleCheckChange(row.checked, row)
          })
        }
      },
      setDefaultHeightRow() {
        if (this.defaultHeightRowId) {
          var row = this.getParentdata(this.defaultHeightRowId, this.treeData)
          this.$nextTick(() => {
            this.$refs[this.mqy].setCurrentRow(row);
          });
        }
      },
      //得到树的高度
      getTreeLevel(obj) {
        this.initTreeLevel++
        if (obj[this.treeProps.children]) {
          if (obj[this.treeProps.children].length > 0) {
            this.getTreeLevel(obj[this.treeProps.children][0])
          }
        }
      },
      // 递归赋值
      setVisible(obj, i1) {
        var pidx = ''
        //如果不是根目录
        if (obj[this.parentKey] !== this.topParentVal) {
          pidx = (this.getParentdata(obj[this.parentKey], this.treeData)).idx + '-'
          obj.parentNum = (this.getParentdata(obj[this.parentKey], this.treeData))[this.treeProps.children].length
        } else {
          obj.parentNum = this.treeData.length
        }
        obj.idx = pidx + (i1 + 1).toString()
        obj.num = i1 + 1 //自己当前在所在父级下面的位置
        this.treeTotal++;
        this.$set(obj, 'checked', false) //默认没选中
        this.$set(obj, 'indeterminate', false)
        this.$set(obj, 'expand', false) //全部默认为false，则不打开
        this.$set(obj, 'loading', false) //全部默认为false，则不loading
        //如果有children
        if (obj[this.treeProps.children]) {
          if (obj[this.treeProps.children].length > 0) {
            this.$set(obj, 'expand', true) //默认全部打开，但是只给有children的赋值
            obj[this.treeProps.children].forEach((e, i2) => {
              this.setVisible(e, i2)
            })
          }
        }
      },
      // 根据parentid/id找到数据
      getParentdata(parentid, tableData) {
        for (var e of tableData) {
          if (e[this.rowKey] === parentid) {
            this.idxObj = e
            break;
          } else if (e[this.treeProps.children]) {
            if (e[this.treeProps.children].length > 0) {
              this.getParentdata(parentid, e[this.treeProps.children])
            }
          }
        }
        return this.idxObj
      },
      // 某一行展开或者关闭的时候会触发该事件,expanded---boolean：true/false：当前是否展开
      expandChange(row, expanded) {
        //把当前行展开状态放入row
        this.$set(row, 'expand', expanded)
      },
      //点击文件夹打开树形
      openTree(event, row) {
        //组织事件冒泡，防止触发行事件,在上面使用了stop修饰符，也能达到这个效果
        // event.stopPropagation()
        //开启了懒加载且hasChildren为true，且children没有
        if (this.lazy && row[this.treeProps.hasChildren] && !row[this.treeProps.children]) {
          //异步懒加载
          //开启loading
          this.$set(row, 'loading', true);
          this.load(row, (childrenData) => {
            const promise = new Promise((resolve, reject) => {
              if (typeof childrenData == 'object') {
                return resolve(childrenData)
              } else {
                return reject(new Error('必须是一个数组！'))
              }
            }).then(res => {
              this.$set(row, 'children', []);
              this.$set(row, 'children', childrenData);
              delete row[this.treeProps.hasChildren]
              this.init()
            }).catch(err => {
              console.log(err)
            }).finally(() => {
              //关闭loading
              this.$set(row, 'loading', false);
              this.$refs[this.mqy].toggleRowExpansion(row)
            })
          });
        }
        //展开
        this.$refs[this.mqy].toggleRowExpansion(row)
      },
      //全选
      handleCheckAllChange() {
        this.isIndeterminate = false
        //改变所有子级
        this.changeAllVal(this.treeData, this.checkAll, this.checkAll, true)
        this.getSelectedDatas()
      },
      //单选
      handleCheckChange(event, row) {
        //防止冒泡，只允许一次
        if (event.target && event.target.tagName) {
          if (event.target.tagName !== 'INPUT') {
            return
          }
        }

        //当前行选中
        this.$refs[this.mqy].setCurrentRow(row);

        //当前行
        this.selectRow = row
        //只要点了自己，就代表全选,所以，无论自己的indeterminate是什么，直接为false
        this.$set(row, 'indeterminate', false)
        //首选改变自身状态
        this.$set(row, 'checked', !row.checked)

        //1.数据源中找到父级-----(设置所有父亲的indeterminate状态,这是在先点自己，父级checked为false的情况下，我们不改变父亲的checked状态)
        this.changeAllValByPid(row[this.parentKey], row)

        //2.如果有子级，遍历找到所有的子级------（设置所有子级的状态）
        if (row[this.treeProps.children] && row[this.treeProps.children].length > 0) {
          //------------------1.强关联：所有子级的状态随着父级的状态改变--------------------
          //------------------2.弱关联：如果子级没有选完，不处理；如果子级没选中/子级都选中，，所有子级的状态随着父级的状态改变；--------------------
          var isChecked = []
          var parentChildren = row[this.treeProps.children];
          parentChildren.forEach(e => {
            if (e.checked) {
              isChecked.push(e)
            }
          })
          //arg1：所有的子级，arg2：自己的选中状态,arg3:当前子级的选中状况（一个没选中为true，否则为false）
          this.changeAllVal(row[this.treeProps.children], row.checked, (isChecked.length < parentChildren.length &&
              isChecked.length > 0) ?
            false : true, false)
        }

        //传递给父组件的数据
        if (this.showCheckbox) {
          this.getSelectedDatas()
        } else {
          //替换
          this.selectRows.splice(0, 1, row)
          this.$emit("getSelectedDatas", this.selectRows,this.$refs[this.mqy]);
          this.$emit("getSelectedRow", this.selectRow,this.$refs[this.mqy]);
        }
      },
      //点击行触发，行事件只触发当前行选中
      handleRowClick(row, column, event) {
        this.handleCheckChange(row.checked, row)
      },
      handleCurrentChange(currentRow,oldCurrentRow){
          //额外返回表格的实例
         this.$emit("current-change", currentRow,oldCurrentRow,this.$refs[this.mqy]);
      },
      handleSelectionChange(selection){
         this.$emit("selection-change",selection,this.$refs[this.mqy]);
      },
      //获得所有已经选中的数据
      getSelectedDatas() {
        this.selectRows = []
        this.getAllCheckedData(this.treeData)
        this.$emit("getSelectedDatas", this.selectRows,this.$refs[this.mqy]);
        this.$emit("getSelectedRow", this.selectRow,this.$refs[this.mqy]);
      },
      //递归改变所有子级的值，所有的子级跟着父级的状态走
      changeAllVal(data, status, checkStrictlyed, checkedAll) {
        data.forEach(e => {
          //强关联//弱关联
          if (this.checkStrictly || checkStrictlyed || checkedAll) {
            this.$set(e, 'indeterminate', false)
            this.$set(e, 'checked', status)
          }

          if (e[this.treeProps.children] && e[this.treeProps.children].length > 0) {
            this.changeAllVal(e[this.treeProps.children], status, checkStrictlyed, checkedAll)
          }
        })
      },
      //递归改变所有父级的值
      changeAllValByPid(fid, row) {
        if (fid !== this.topParentVal) {
          var parentRow = this.getParentdata(fid, this.treeData)
          //查看该父级下面的子级的（选中&伪选中）状况，三种情况
          //-----------------------checkStrictly------------------------
          //1.全部为选中：indeterminate=false，checked=true，
          //2.全部为伪选中：indeterminate=true，checked=false，
          //3.一部分选中，一部分伪选中：indeterminate=true，checked=false，
          //4.没有伪选中，也没有选中，indeterminate=false，checked=false，
          //---------------------!checkStrictly------------------------
          //----------------------1.此时父级已经选中
          //1.全部为选中：.indeterminate=false，checked=true，
          //2.全部为伪选中：.indeterminate=false，checked=true，
          //3：一部分选中，一部分伪选中：.indeterminate=false，checked=true，
          //4.没有伪选中，也没有选中，indeterminate=false，checked=true，
          //----------------------1.此时父级未选中
          //1.2.3.indeterminate=true，checked=false，
          //4.没有伪选中，也没有选中，indeterminate=false，checked=false，
          var isChecked1 = [] //选中的
          var isChecked2 = [] //伪选中的
          var parentChildrenLength = parentRow[this.treeProps.children].length //children的长度
          parentRow[this.treeProps.children].forEach(e => {
            if (e.checked) {
              isChecked1.push(e)
            }
            if (e.indeterminate) {
              isChecked2.push(e)
            }
          })

          if (this.checkStrictly) {
            if (isChecked1.length == parentChildrenLength) {
              this.$set(parentRow, 'indeterminate', false)
              this.$set(parentRow, 'checked', true)
            } else if (isChecked1.length == 0 && isChecked2.length == 0) {
              this.$set(parentRow, 'indeterminate', false)
              this.$set(parentRow, 'checked', false)
            } else {
              this.$set(parentRow, 'indeterminate', true)
              this.$set(parentRow, 'checked', false)
            }
          } else {
            //如果已经选中
            if (parentRow.checked) {
              this.$set(parentRow, 'indeterminate', false)
              this.$set(parentRow, 'checked', true)
            } else {
              if (isChecked1.length == parentChildrenLength) {
                this.$set(parentRow, 'indeterminate', true)
                this.$set(parentRow, 'checked', false)
              } else if (isChecked1.length == 0 && isChecked2.length == 0) {
                this.$set(parentRow, 'indeterminate', false)
                this.$set(parentRow, 'checked', false)
              } else {
                this.$set(parentRow, 'indeterminate', true)
                this.$set(parentRow, 'checked', false)
              }
            }
          }

          //继续找这条数据的父亲
          this.changeAllValByPid(parentRow[this.parentKey], parentRow)
        }
      },
      //获得所有选中的数据
      getAllCheckedData(data) {
        data.forEach(e => {
          if (e.checked) {
            this.selectRows.push(e)
          }
          if (e[this.treeProps.children] && e[this.treeProps.children].length > 0) {
            this.getAllCheckedData(e[this.treeProps.children])
          }
        })
      }
    }
  }
</script>

<style scoped>
  /deep/ .treeTable .el-table__expand-icon--expanded {
    transform: rotate(0deg);
  }

  /deep/.el-table [class*="el-table__row--level"] .el-table__expand-icon {
    display: none !important;
  }

  /deep/ .el-icon-arrow-right {
    display: none;
  }

  /deep/ .el-table__placeholder {
    width: 0px;
  }

  /deep/ .treeTable .el-table-column--selection {
    /* background-color:#F4F6FA */
  }

  /deep/ .el-badge__content.is-fixed.is-dot {
    right: 4px;
    top: 5px;
    background: #74db41;
  }

  /deep/ .el-badge__content.is-dot {
    height: 10px;
    width: 10px;
  }
</style>
