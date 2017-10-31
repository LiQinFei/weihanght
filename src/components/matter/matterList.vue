<template>
  <div class="adPosition" v-loading.fullscreen.lock="fullscreenLoading">
    <el-breadcrumb separator="/" style="margin-bottom:10px">
      <el-breadcrumb-item :to="{ path: '/' }"> 物料管理</el-breadcrumb-item>
      <el-breadcrumb-item>物料列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索 -->
    <div class="inputs">
      <el-form :inline="true" :model="search" class="demo-form-inline">
        <el-form-item label="物料名称">
          <el-input v-model="search.itemName" placeholder="请输入"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="searchgoods">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="warning" @click="newEdits">新增物料</el-button>
        </el-form-item>
      </el-form>
    </div>
    <!-- 列表展示 -->
    <el-table ref="multipleTable" :data="tableData.dataList" border style="width: 100%" v-loading="loading">
      <el-table-column type="index" width="50">
      </el-table-column>
      <el-table-column prop="itemName" label="物料名称">
      </el-table-column>
      <el-table-column prop="code" label="物料编码">
      </el-table-column>
      <el-table-column prop="categoryName" label="物料分类名称">
      </el-table-column>
      <el-table-column prop="price" label="成本价">
      </el-table-column>
      <el-table-column prop="price1" label="普卡价">
      </el-table-column>
      <el-table-column prop="price2" label="金卡价">
      </el-table-column>
      <el-table-column prop="price3" label="钻卡价">
      </el-table-column>
      <el-table-column prop="createDate" label="创建时间" min-width="130" >
      </el-table-column>
      <el-table-column prop="description" min-width="160" label="备注" show-overflow-tooltip>
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
    <el-dialog class="Edit" title="新建物料" :visible.sync="newdialogFormVisible">
      <el-form :model="newEdit">
        <el-form-item label="物料名称:" :label-width="formLabelWidth">
          <el-input v-model="newEdit.itemName" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="物料编码:" :label-width="formLabelWidth">
          <el-input v-model="newEdit.code" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="物料分类名称:" :label-width="formLabelWidth">
          <el-select v-model="newEdit.categoryId" placeholder="请选择">
            <el-option v-for="item in newList" :key="item.wicId" :label="item.categoryName" :value="item.wicId">
            </el-option>
          </el-select>

        </el-form-item>
        <el-form-item label="成本价:" :label-width="formLabelWidth">
          <el-input v-model="newEdit.price" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="普卡价:" :label-width="formLabelWidth">
          <el-input v-model="newEdit.price1" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="金卡价:" :label-width="formLabelWidth">
          <el-input v-model="newEdit.price2" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="钻卡价:" :label-width="formLabelWidth">
          <el-input v-model="newEdit.price3" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="备注:" :label-width="formLabelWidth">
          <el-input v-model="newEdit.description" auto-complete="off"></el-input>
        </el-form-item>


      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="newdialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="sendNew">确 定</el-button>

      </div>
    </el-dialog>

    <!-- 编辑弹出层 -->
    <el-dialog class="Edit" title="物料编辑" :visible.sync="dialogFormVisible">
      <el-form :model="form">

        <el-form-item label="物料名称:" :label-width="formLabelWidth">
          <el-input v-model="form.itemName" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="物料编码:" :label-width="formLabelWidth">
          <el-input v-model="form.code" auto-complete="off"></el-input>
        </el-form-item>
        <!-- <el-form-item label="物料分类名称:" :label-width="formLabelWidth">
          <el-input v-model="form.categoryId" auto-complete="off"></el-input>
        </el-form-item> -->
        <el-form-item label="物料分类名称:" :label-width="formLabelWidth">
          <el-select v-model="form.categoryId" placeholder="请选择">
            <el-option v-for="item in newList" :key="item.wicId" :label="item.categoryName" :value="item.wicId">
            </el-option>
          </el-select>

        </el-form-item>
        <el-form-item label="成本价:" :label-width="formLabelWidth">
          <el-input v-model="form.price" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="普卡价:" :label-width="formLabelWidth">
          <el-input v-model="form.price1" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="金卡价:" :label-width="formLabelWidth">
          <el-input v-model="form.price2" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="钻卡价:" :label-width="formLabelWidth">
          <el-input v-model="form.price3" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="备注:" :label-width="formLabelWidth">
          <el-input v-model="form.description" auto-complete="off"></el-input>
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
          positionName: "",
          categoryName: "",
          categoryId: '',
          price: "",
          price1: "",
          price2: "",
          price3: "",
          code:"",
          description:""
        },
        newList: [],
        search: {
          pageIndex: 0,
          pageSize: 10,
          itemName: "",
          categoryName: ""
        },
        tableData: [],
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
          url: url + "/clientFindMaterialListAndPaged",
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
        this.productGetAllRootCategoryList();
      },
      // 获取分类列表
      productGetAllRootCategoryList() {
        let that = this;
        this.$http({
          method: "post",
          url: url + "/clientGetMaterialCategoryId"
        }).then(function(res) {
          that.newList = JSON.parse(res.data).data;
          // console.log(JSON.parse(res.data).data)
          console.log(that.newList)
        });
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
          url: url + "/clientSaveMaterialInfo",
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
          url: url + "/clientSaveMaterialInfo",
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
      },
      //删除
      del(row) {
        console.log(row)
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
              url: url + "/clientDeleteMaterial",
              data: {
                wiId: row.wiId
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
        this.productGetAllRootCategoryList();

        // var _item = {}
        // for (var i in row) {
        //   _item[i] = row[i]
        // }
        // this.form = _item
        // console.log(row)

        let that = this;
        this.$http({
          method: "post",
          url: url + "/clientGetMaterialInfo",
          data: {
            wiId: row.wiId
          }
        }).then(function(res) {
          let data = JSON.parse(res.data).data;
          that.form = data;
          console.log(that.form)
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



