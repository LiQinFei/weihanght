<template>
  <div class="adPosition" v-loading.fullscreen.lock="fullscreenLoading">
    <el-breadcrumb separator="/" style="margin-bottom:10px">
      <el-breadcrumb-item :to="{ path: '/' }"> 物料管理</el-breadcrumb-item>
      <el-breadcrumb-item>物料分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索 -->
    <div class="inputs">
      <el-form :inline="true" :model="search" class="demo-form-inline">
        <el-form-item label="物料分类名称">
          <el-input v-model="search.categoryName" placeholder="请输入"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="searchgoods">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="warning" @click="newEdits">新增分类</el-button>
        </el-form-item>
      </el-form>
    </div>
    <!-- 列表展示 -->
    <el-table ref="multipleTable" :data="tableData.dataList" border style="width: 100%" v-loading="loading">
      <el-table-column type="index" width="50">
      </el-table-column>
      <el-table-column prop="categoryName" label="物料分类名称">
      </el-table-column>
      <el-table-column prop="code" label="编码">
      </el-table-column>
      <el-table-column prop="description" label="备注">
      </el-table-column>
      <el-table-column prop="createDate" label="创建时间">
      </el-table-column>
      <el-table-column prop="parentName" label="父分类名">
      </el-table-column>
      <el-table-column fixed="right" label="操作" width="150">
        <template scope="scope">
          <el-button @click="handleEdit(scope.row)" type="text" size="small">编辑</el-button>
          <el-button type="text" size="small" @click="del(scope.row)">删除</el-button>

        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination @current-change="handleCurrentChange" :page-size="20" layout="prev, pager, next, jumper" :page-count="tableData.totalPage">
    </el-pagination>

    <!-- 新建物料弹出层 -->
    <el-dialog class="Edit" title="新建物料分类" :visible.sync="newdialogFormVisible">
      <el-form :model="newEdit">
        <el-form-item label="物料分类名称:" :label-width="formLabelWidth">
          <el-input v-model="newEdit.categoryName" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="编码:" :label-width="formLabelWidth">
          <el-input v-model="newEdit.code" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="备注:" :label-width="formLabelWidth">
          <el-input v-model="newEdit.description" auto-complete="off"></el-input>
        </el-form-item>
        <!-- <el-form-item label="创建时间:" :label-width="formLabelWidth">
          <el-date-picker v-model="newEdit.createDate" type="date" placeholder="选择日期" :picker-options="pickerOptions0"></el-date-picker>
        </el-form-item> -->
        <el-form-item label="父分类名:" :label-width="formLabelWidth">
          <el-input v-model="newEdit.parentName" auto-complete="off"></el-input>
        </el-form-item>


      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="newdialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="sendNew">确 定</el-button>
      
      </div>
    </el-dialog>

    <!-- 编辑弹出层 -->
    <el-dialog class="Edit" title="物料分类编辑" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="物料分类名称:" :label-width="formLabelWidth">
          <el-input v-model="form.categoryName" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="编码:" :label-width="formLabelWidth">
          <el-input v-model="form.code" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="备注:" :label-width="formLabelWidth">
          <el-input v-model="form.description" auto-complete="off"></el-input>
        </el-form-item>
        <!-- <el-form-item label="创建时间:" :label-width="formLabelWidth">
          <el-date-picker v-model="form.createDate" type="date" placeholder="选择日期" :picker-options="pickerOptions0"></el-date-picker>
        </el-form-item> -->
        <el-form-item label="父分类名:" :label-width="formLabelWidth">
          <el-input v-model="form.parentName" auto-complete="off"></el-input>
        </el-form-item>


      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="clientProductModifySortInfo">确 定</el-button>
      </div>
    </el-dialog>

  </div>
</template>


<script>
export default {
  data() {
    return {
      loading: true,
      fullscreenLoading: false,
      dialogFormVisible: false,
      newdialogFormVisible: false,
      formLabelWidth: "120px",
      form: {},
      value: "",
      newEdit: {
        categoryName: "",
        code: "",
        description: "",
        createDate: "",
        parentName: ""
      },
      search: {
        pageIndex: 0,
        pageSize: 10,
        itemName: "",
        categoryName: ""
      },
      tableData: []
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      this.loading = true;
      let that = this;
      this.$http({
        method: "post",
        url: url + "/clientMaterialCategoryFindPage",
        data: that.search
      }).then(function(res) {
        that.loading = false;
        that.tableData = JSON.parse(res.data);
        console.log(that.tableData);
      });
    },
    //点击新增
    newEdits() {
      this.newdialogFormVisible = true;
      //this.productGetAllRootCategoryList()
    },

    //点击分页
    handleCurrentChange(val) {
      this.search.pageIndex = val - 1;
      this.getList();
    },
    //查询
    searchgoods() {
      this.getList();
    },
    //新增物料
    sendNew() {
      let that = this;
      this.$http({
        method: "post",
        url: url + "/clientSaveMaterialCategoryInfo",
        data: that.newEdit
      }).then(function(res) {
        let data = JSON.parse(res.data);
        if (data.apiStatus == 1) {
          that.newdialogFormVisible = false;
          that.getList();
          that.$message({
            message: data.msg,
            type: "success"
          });
        } else {
          that.$message.error(data.msg);
        }
      });
    },
    // 编辑完成确定
    clientProductModifySortInfo() {
      let that = this;
      this.$http({
        method: "post",
        url: url + "/clientSaveMaterialCategoryInfo",
        data: that.form
      }).then(function(res) {
        let data = JSON.parse(res.data);
        if (data.apiStatus == 1) {
          that.form = {};
          that.dialogFormVisible = false;
          that.getList();

          that.$message({
            message: data.msg,
            type: "success"
          });
        } else {
          that.$message.error(data.msg);
        }
      });
      console.log(that.form);
    },
    //删除
    del(row) {
      console.log(row.wicId)
      let that = this;
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          that.fullscreenLoading = true;
          that.$http({
              method: "post",
              url: url + "/clientDeleteMaterialCategoryInfo",
              data: {
                wicId: row.wicId
              }
            })
            .then(function(res) {
              let data = JSON.parse(res.data);
              if (data.apiStatus == 1) {
                that.fullscreenLoading = false;
                that.getList();
                that.$message({
                  message: data.msg,
                  type: "success"
                });
              } else {
                that.$message.error(data.msg);
              }
            });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    //点击编辑按钮获取row参数
    handleEdit(row) {
      this.dialogFormVisible = true;

      // var _item = {}
      // for (var i in row) {
      //   _item[i] = row[i]
      // }
      // this.form = _item
      // console.log(row)

      let that = this;
      this.$http({
        method: "post",
        url: url + "/clientGetMaterialCategoryInfo",
        data: {
          wicId: row.wicId
        }
      }).then(function(res) {
        let data = JSON.parse(res.data).data;
        that.form = data;
      });
    }
  }
};
</script>

<style lang="scss" scoped>

</style>
<style lang="scss">
.adPosition {
  .el-table .cell {
    text-align: center;
  }
  .inputs {
    height: 40px;
    width: 100%;
    text-align: left;
    padding-bottom: 5px;
    .el-input {
      height: 30px;
      width: 180px;
      display: inline-block;
      input {
        display: inline-block;
        width: 180px;
        height: 31px;
        vertical-align: middle;
      }
    }
  }
  .Edit {
    .el-input__inner {
      width: 200px;
    }
  }
}
</style>



