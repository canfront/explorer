<template>
  <!-- 文件管理器 auth：weilan time：2019-10-25 github：https://github.com/hql7 -->
  <div class="wl-explorer">
    <!-- 头部按钮区 -->
    <el-form class="wl-header-btn" :inline="true" :size="size" @submit.native.prevent>
      <el-breadcrumb separator="/"  class="app-breadcrumb">
        <el-breadcrumb-item >当前目录：<a @click="changePath(0)">根目录</a></el-breadcrumb-item>
        <span v-if="pathDetail.parentChains">
          <el-breadcrumb-item v-for="(pathDetail, pathKey) in pathDetail.parentChains" :key="pathKey"><a @click="changePath(pathDetail)">{{pathDetail.path}}</a></el-breadcrumb-item>
          <el-breadcrumb-item v-if="pathDetail.path">{{pathDetail.path}}</el-breadcrumb-item>
        </span>
      </el-breadcrumb>
    </el-form>
    <el-form class="wl-header-btn" :inline="true" :size="size" @submit.native.prevent>
      <el-dropdown split-button size="small" type="primary" @command="changeSystem">{{fileSystem[currentSystem]}}
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item v-for="(option, optionKey) in fileSystem" :command="optionKey" :key="optionKey">{{option}}</el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
      <el-form-item>
        <el-button type="primary" @click="handleFolder('add')">新增文件夹</el-button>
        <el-button @click="showUpload">上传文件</el-button>
        <!-- solt自定义头部按钮区 -->
        <slot name="header-btn"></slot>
      </el-form-item>
      <el-form-item v-show="uploading.ing">
        <span>正在上传：</span>
        <span class="c-blue u-uploading-name">{{uploading.name}}</span>
        <span class="c-blue">({{uploading.percentage}}%)</span>
      </el-form-item>
      <el-form-item class="u-right">
        <i
          class="iconfont icon-wl-list file-show-type"
          v-show="layout.show_list"
          @click="layout.show_list=!layout.show_list"
        ></i>
        <i
          class="iconfont icon-wl-grid file-show-type"
          v-show="!layout.show_list"
          @click="layout.show_list=!layout.show_list"
        ></i>
      </el-form-item>
      <el-form-item class="file-handle-box">
        <i
          class="iconfont icon-wl-left file-path-handle"
          :class="{'u-disabled':pathIsStart}"
          @click="pathBtn('prv')"
        ></i>
        <i
          class="iconfont icon-wl-right file-path-handle"
          :class="{'u-disabled':pathIsEnd}"
          @click="pathBtn('next')"
        ></i>
        <i
          class="iconfont icon-wl-up file-path-handle"
          :class="{'u-disabled':pathIsEnd}"
          @click="pathBtn('top')"
        ></i>
      </el-form-item>
    </el-form>
    <!--文件路径操作区-->
    <!-- 主内容区 -->
    <el-scrollbar class="wl-main-scroll">
      <!-- 文件列表区 -->
      <div class="wl-main-list">
        <!-- 表格型文件列表 -->
        <el-table
          v-show="layout.show_list"
          @selection-change="handleSelectionPath"
          highlight-current-row
          :border="showBorder"
          :data="pathDatas"
          class="wl-table"
          ref="wl-table"
        >
          <el-table-column v-if="showCheckbox" align="center" type="selection" width="55"></el-table-column>
          <el-table-column v-if="showIndex" align="center" type="index" label="序号" width="55"></el-table-column>
          <slot name="table-column-top"></slot>
          <el-table-column
            v-for="i of formatPathColumns"
            show-overflow-tooltip
            :key="i._id"
            :prop="i.prop"
            :label="i.label"
            :fixed="i.fixed"
            :align="i.align"
            :sort-by="i.sortBy"
            :sortable="i.sortable"
            :min-width="i.width"
            :formatter="i.formatter"
            :column-key="i.columnKey"
            :class-name="i.className"
            :sort-method="i.sortMethod"
            :header-align="i.headerAlign"
            :render-header="i.renderHeader"
            :label-class-name="i.labelClassName"
          >
            <template slot-scope="scope">
              <!-- 非名称列 -->
              <template v-if="i.prop !== selfProps.name">
                {{
                i.formatter
                ? i.formatter(scope.row, scope.column, scope.row[i.prop],scope.$index)
                : scope.row[i.prop]
                }}
              </template>
              <!-- 名称列 -->
              <div
                v-else
                @click="enterTheLower(scope.row, true)"
                class="wl-name-col wl-is-folder"
              >
                <!-- 不同文件类型图标区 -->
                <div class="namecol-iconbox">
                  <img :src="pathTypeIcon(scope.row)" class="name-col-icon" alt="文件类型图标" />
                </div>
                <!-- 不同文件类型 显示内容-->
                <div class="namecol-textbox">
                  {{
                  i.formatter
                  ? i.formatter(scope.row, scope.column, scope.row[i.prop],scope.$index)
                  : scope.row[i.prop]
                  }}
                </div>
              </div>
            </template>
          </el-table-column>
          <slot name="table-column-bottom"></slot>
        </el-table>
        <div style="margin-top: 20px" v-if="pathDatas.length">
          <el-button @click="deleteSelection('path')">删除选中的目录(只有空目录才能被删除)</el-button>
        </div>
        <!-- 列表型文件列表 -->
        <ul class="wl-list" v-show="!layout.show_list">
          <li class="wl-list-item wl-is-folder" v-for="(i, idx) in pathDatas" :key="i.Id">
            <el-checkbox class="wl-checkbox" @change="listItemCheck($event,i)" v-model="i._checked"></el-checkbox>
            <div @click="enterTheLower(i, true)">
              <img :src="pathTypeIcon(i)" class="name-col-icon" alt="文件类型图标" />
              <p class="list-item-name" :title="i[selfProps.name]">
                {{
                i.formatter
                ? i.formatter(i, null, i[selfProps.name], idx)
                : i[selfProps.name]
                }}
              </p>
            </div>
          </li>
        </ul>
        <!-- 横排型文件列表 -->
        <slot name="main"></slot>
      </div>
      <!-- 文件列表区 -->
      <div class="wl-main-list">
        <!-- 表格型文件列表 -->
        <el-table
          v-show="layout.show_list"
          @selection-change="handleSelectionFile"
          highlight-current-row
          :border="showBorder"
          :data="fileDatas"
          class="wl-table"
          ref="wl-table-file"
        >
          <el-table-column v-if="showCheckbox" align="center" type="selection" width="55"></el-table-column>
          <el-table-column v-if="showIndex" align="center" type="index" label="序号" width="55"></el-table-column>
          <slot name="table-column-top"></slot>
          <el-table-column
            v-for="i of formatFileColumns"
            show-overflow-tooltip
            :key="i._id"
            :prop="i.prop"
            :label="i.label"
            :fixed="i.fixed"
            :align="i.align"
            :sort-by="i.sortBy"
            :sortable="i.sortable"
            :min-width="i.width"
            :formatter="i.formatter"
            :column-key="i.columnKey"
            :class-name="i.className"
            :sort-method="i.sortMethod"
            :header-align="i.headerAlign"
            :render-header="i.renderHeader"
            :label-class-name="i.labelClassName"
          >
            <template slot-scope="scope">
              <!-- 非名称列 -->
              <template v-if="i.prop !== fileSelfProps.name">
                {{
                i.formatter
                ? i.formatter(scope.row, scope.column, scope.row[i.prop],scope.$index)
                : scope.row[i.prop]
                }}
              </template>
              <!-- 名称列 -->
              <div
                v-else
                @click="enterTheLower(scope.row, false)"
                class="wl-name-col wl-is-folder"
              >
                <!-- 不同文件类型图标区 -->
                <div class="namecol-iconbox">
                  <img :src="fileTypeItem(scope.row, 'path')" class="name-col-icon" alt="文件类型图标" />
                </div>
                <!-- 不同文件类型 显示内容-->
                <div class="namecol-textbox">
                  {{
                  i.formatter
                  ? i.formatter(scope.row, scope.column, scope.row[i.prop],scope.$index)
                  : scope.row[i.prop]
                  }}
                </div>
              </div>
            </template>
          </el-table-column>
          <slot name="table-column-bottom"></slot>
        </el-table>
        <div style="margin-top: 20px" v-if="fileDatas.length">
          <el-button @click="deleteSelection('file')">删除选中的文件(软删除)</el-button>
        </div>

        <!-- 列表型文件列表 -->
        <ul class="wl-list" v-show="!layout.show_list">
          <li class="wl-list-item wl-is-folder" v-for="(i, idx) in fileDatas" :key="i.Id">
            <el-checkbox class="wl-checkbox" @change="listItemCheck($event,i)" v-model="i._checked"></el-checkbox>
            <div @click="enterTheLower(i, false)">
              <img :src="fileTypeItem(i, 'path')" class="name-col-icon" alt="文件类型图标" />
              <p class="list-item-name" :title="i[fileSelfProps.name]">
                {{
                i.formatter
                ? i.formatter(i, null, i[fileSelfProps.name], idx)
                : i[fileSelfProps.name]
                }}
              </p>
            </div>
          </li>
        </ul>
        <!-- 横排型文件列表 -->
        <slot name="main"></slot>
      </div>
    </el-scrollbar>
    <!-- slot 自定义dom区 -->
    <slot></slot>
  </div>
</template>

<script>
import submitBtn from "../../components/submit-btn.vue"; // 导入防抖组件
const guid = "00000000-0000-0000-0000-000000000000";
export default {
  name: "wlExplorer",
  components: {submitBtn},
  data() {
    return {
      uploading: {name: "JS从脱贫到脱发你好长啊", percentage: 0, ing: false}, // 当前上传文件状态
      layout: {
        show_list: true, // 文件展示形式
        //edit_path: false, // 是否编辑路径
        //view: false, // 预览视图
        //move: false, // 移动视图
        upload: false // 上传视图
      }, // 视图管理
      multipleSelections: {},
    };
  },
  props: {
    pathDetail: {type: Object},
    showCheckbox: {type: Boolean, default: true}, // 是否显示复选框
    showIndex: {type: Boolean, default: true}, // 是否显示顺序号
    showBorder: {type: Boolean, default: true}, // 表格是否显示边框
    pathDatas: Array,
    fileDatas: Array,
    // 文件表头数据【[参数：所有el-Table-column Attributes] (https://element.eleme.cn/#/zh-CN/component/table)】
    pathColumns: Array,
    fileColumns: Array,
    fileSystem: Object,
    currentSystem: String,
    props: Object,
    fileProps: Object,
    // 是否锁定文件、文件夹函数,true则不可进行操作
    isLockFn: Function,
    size: {type: String, default: "medium"}
  },
  computed: {
    formatFileColumns() {
      let _data = this.fileColumns || [];
      _data.forEach((i, idx) => {
        i._id = `_col_${idx}`;
      });
      return _data;
    },
    // 自身表头数据
    formatPathColumns() {
      let _data = this.pathColumns || [];
      _data.forEach((i, idx) => {
        i._id = `_col_${idx}`;
      });
      return _data;
    },
    fileSelfProps() {
      return {
        isLock: "isLock", // Boolean 用于有布尔值字段表示数据是否锁定文件类型的情况,当使用isLockFn函数时，此参数被忽略
        name: "name", // String 用于显示名称列的字段
        suffix: "suffix", // String 用于判断后缀或显示文件类型列的字段
        match: "name", // String 用于设定输入框自动补全的匹配字段
        ...this.fileProps
      };
    },
    // 自身配置项
    selfProps() {
      return {
        isLock: "isLock", // Boolean 用于有布尔值字段表示数据是否锁定文件类型的情况,当使用isLockFn函数时，此参数被忽略
        name: "name", // String 用于显示名称列的字段
        suffix: "suffix", // String 用于判断后缀或显示文件类型列的字段
        match: "name", // String 用于设定输入框自动补全的匹配字段
        ...this.props
      };
    },
    // 自身移动 下拉框树 配置项
    selfMoveProps() {
      return {
        label: this.selfProps.pathName,
        children: this.selfProps.pathChildren,
        disabled: this.selfProps.pathDisabled
      };
    },
    // 将是否锁定文件、文件夹的两种判断方式合并返回
    selfIsLock() {
      return this.isLockFn ? "isLock" : this.selfProps.isLock;
    },
    // 当前是否最后一步
    pathIsEnd() {
      return false;/*(
        this.path.history[this.path.history.length - 1].id === this.file.id
      );*/
    },
    // 当前是否最后一步
    pathIsStart() {
      return true;//this.path.history[0].id === this.file.id;
    },
  },
  methods: {
    changePath(pathData) {
      if (pathData === 0) {
        pathData = {id: 0};
      }
      this.$emit("search", pathData);
    },
    changeSystem(system) {
      //alert(system);
    },
    /**
     * 文件夹时进入下一级-文件时预览文件
     * row: Object 行数据
     * isFolder: Boolean 行是否文件夹
     */
    enterTheLower(row, isFolder) {
      if (!isFolder) {
        this.previewFile(row);
        return;
      }

      this.$emit("search", row, true);
    },
    pathTypeIcon(row) {
      let _path = "";
      _path = row[this.selfIsLock]
        ? require("./images/file_automatic@3x.png")
        : require("./images/folder@3x.png");
      return _path;
    },
    // 根据文件类型显示图标
    fileTypeItem(row, type) {
      let _suffix = row[this.fileSelfProps.suffix];
      let fileTypes = {
        image: {path: 'img', elems: ["jpg", "jpeg", "png", "gif", "bmp"]},
        zip: {path: 'zip', elems: ["zip", "rar", "7z"]},
        video: {path: 'video', elems: ["avi", "mp4", "rmvb", "flv", "mov", "m2v", "mkv"]},
        audio: {path: 'mp3', elems: ["mp3", "wav", "wmv", "wma"]},
        xlsx: {path: 'excel', elems: ["xls", "xlsx"]},
        docx: {path: 'docx', elems: ["doc", "docx"]},
        pdf: {path: 'pdf', elems: ["pdf"]},
        ppt: {path: 'ppt', elems: ['ppt']},
        txt: {path: 'txt', elems: ['txt']},
        none: {path: 'none', elems: []},
      };

      let currentTypeKey = 'none';
      for (let key in fileTypes) {
        let elems = fileTypes[key].elems;
        if (fileTypes[key].elems.includes(_suffix)) {
          currentTypeKey = key;
          break;
        }
      }
      if (type != 'path') {
        return currentTypeKey;
      }
      let currentType = fileTypes[currentTypeKey];
      return require('./images/file_' + currentType.path + '@3x.png');
    },
    // 列表模式记录多选数据
    listItemCheck(check, val) {
      this.$refs["wl-table"].toggleRowSelection(val);
    },

    /**
     * 文件夹编辑操作
     * type: string 添加add 编辑edit
     * auth: boolean 是否只修改权限
     */
    handleFolder(type) {
      this.$emit("handleFolder");
      this.closeUpload();
    },
    // 显示上传界面
    showUpload() {
      this.$emit("showUpload");
    },
    closeUpload() {
      this.$emit("closeUpload");
    },
    previewFile(row) {
      let previewType = this.fileTypeItem(row, 'type');
      this.$emit("previewFile", row, previewType);
    },
    handleSelectionPath(val) {
        console.log(val, 'ttttt');
      this.multipleSelectionDeal(val, 'path');
    },
    handleSelectionFile(val) {
      this.multipleSelectionDeal(val, 'file');
    },
    multipleSelectionDeal(rows, type) {
        console.log(rows);
      let selections = [];
      if (rows) {
        rows.forEach(row => {
          selections.push(row.id);
        });
        this.multipleSelections[type] = selections;
      }
        console.log(this.multipleSelections, 'hhhhhhhf');
    },
    deleteSelection(type) {
      let elems = this.multipleSelections[type] ? this.multipleSelections[type] : [];
      if (elems.length == 0) {
        this.$message({
          showClose: true,
          message: "请选择要删除的文件或文件夹",
          type: "error"
        });
        return;
      }

      // 删除确认
      this.$confirm("是否确认删除选中数据？", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        this.$emit("deleteSelection", type, elems);
      }).catch(() => {
        this.$message({
          type: "info",
          message: "已取消删除"
        });
      });
    },
  }
};
</script>

<style lang="scss">
@import "./css/index.css";
@import "./css/clear.css";
@import "./icons/iconfont.css";
</style>
