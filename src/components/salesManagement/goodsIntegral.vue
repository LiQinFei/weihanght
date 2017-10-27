<template>
  <div class="goodsIntegral" v-loading.fullscreen.lock="fullscreenLoading">
    <el-breadcrumb separator="/" style="margin-bottom:10px">
      <el-breadcrumb-item :to="{ path: '/' }">营销管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品积分</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="inputs">
      <el-form :inline="true" :model="search" class="demo-form-inline">
        <el-form-item label="商品sku">
          <el-input v-model="search.skuName" placeholder="请输入"></el-input>
        </el-form-item>
        <el-form-item label="状态">
          <el-select v-model="search.status" placeholder="全部">
            <el-option label="全部" value=""></el-option>
            <el-option label="有效" value="1"></el-option>
            <el-option label="无效" value="2"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="searchgoods">查询</el-button>
          <el-button type="warning" @click="newVisible = true">新增商品积分</el-button>
        </el-form-item>
      </el-form>
    </div>

    <el-table ref="multipleTable" @selection-change="handleSelectionChange" :data="tableData.dataList" border style="width: 100%" v-loading="loading">
      <el-table-column type="index" width="50">
      </el-table-column>
      <el-table-column prop="skuName" label="商品sku" width="140" show-overflow-tooltip>
      </el-table-column>
      <el-table-column prop="integralValue" label="积分数" width="120" show-overflow-tooltip>
      </el-table-column>

      <el-table-column prop="status" label="状态" width="160" :formatter="status">
      </el-table-column>
      <el-table-column prop="description" min-width="150" show-overflow-tooltip label="备注">
      </el-table-column>

      <el-table-column fixed="right" label="操作" width="150">
        <template scope="scope">
          <el-button @click="handleEdit(scope.row)" type="text" size="small">编辑</el-button>
          <el-button type="text" style="color:#13CE66"  size="small" v-if="scope.row.status ==2" @click="switchStatus(scope.row,1)">有效</el-button>
          <el-button type="text" style="color:#FF4949" size="small" v-if="scope.row.status == 1" @click="switchStatus(scope.row,2)">无效</el-button>
          <el-button type="text" style="color:#FF4949" size="small" @click="del(scope.row)">删除</el-button>

        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination @current-change="handleCurrentChange" :page-size="20" layout="prev, pager, next, jumper" :page-count="tableData.totalPage">
    </el-pagination>

    <!--新增-->

    <el-dialog title="新增权益" :visible.sync="newVisible" class="Edit">
      <el-form :model="newEdit">
        <el-form-item label="商品sku" :label-width="formLabelWidth">
          <el-input v-model="newEdit.skuName" @focus="skuIdFun" auto-complete="off" placeholder="请选择商品"></el-input>
        </el-form-item>
        <el-form-item label="积分数" :label-width="formLabelWidth">
          <el-input v-model="newEdit.integralValue" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="状态" :label-width="formLabelWidth">
          <el-select v-model="newEdit.status">
            <el-option label="有效 " :value="1"></el-option>
            <el-option label="无效" :value="2"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="备注" :label-width="formLabelWidth">
          <el-input v-model="newEdit.description" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="newVisible = false">取 消</el-button>
        <el-button type="primary" @click="sendNewAdd">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 编辑弹出层 -->
    <el-dialog class="Edit" title="积分编辑" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="商品sku" :label-width="formLabelWidth">
          <el-input v-model="form.skuName" @focus="skuIdFun" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="积分数" :label-width="formLabelWidth">
          <el-input v-model="form.integralValue" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="状态" :label-width="formLabelWidth">
          <el-select v-model="form.status">
            <el-option label="有效" :value="1"></el-option>
            <el-option label="无效" :value="2"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="备注" :label-width="formLabelWidth">
          <el-input v-model="form.description" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="newVisible = false">取 消</el-button>
        <el-button type="primary" @click="sendEditAdd">确 定</el-button>
      </div>
    </el-dialog>


    <!-- 活动设置弹出层 -->
    <el-dialog title="选择sku" :visible.sync="goodsSetVisible">
      <el-row>
        <el-col :span="6">
          <!-- //树节点 -->
          <el-tree class="filter-tree" accordion disabled highlight-current @node-click="treeClick" :data="treeList">
          </el-tree>
        </el-col>
        <el-col :span="18" style="padding:0 5px">
          <el-table ref="multipleTable" :data="goodsList.dataList" border style="width: 100%">
            <el-table-column prop="productName" label="商品名称">
            </el-table-column>
            <el-table-column prop="brandName" label="品牌">
            </el-table-column>
            <el-table-column prop="productPrice" label="价格">
            </el-table-column>
            <el-table-column fixed="right" label="操作" width="100">
              <template scope="scope">
                <el-button type="primary" size="small" @click="getGoodsAttr(scope.row)">选择</el-button>
              </template>
            </el-table-column>
          </el-table>
          <!-- //商品下一页  -->
          <el-pagination @current-change="goodsChange" :page-size="5" layout="prev, pager, next, jumper" :page-count="goodsList.totalPage">
          </el-pagination>
        </el-col>
      </el-row>
      <!-- 商品属性设置 -->
      <el-table class="goodSet" empty-text="请增加商品" ref="multipleTable" :data="goodaAttrList" border style="width: 100%;margin-top:10px">

        <el-table-column prop="skuName" label="商品sku">
        </el-table-column>
        <el-table-column prop="productPrice1" label="普卡价">
        </el-table-column>
        <el-table-column prop="productPrice2" label="金卡价">
        </el-table-column>
        <el-table-column prop="productPrice3" label="钻卡价">
        </el-table-column>
        <el-table-column fixed="right" label="操作" width="100">
          <template scope="scope">
            <el-button type="primary" size="small" @click="getSkuSle(scope.row)">选择</el-button>
          </template>
        </el-table-column>
      </el-table>


    </el-dialog>


  </div>
</template>
<script>
  export default {
    data() {
      return {
        loading: true,
        fullscreenLoading: false,
        dialogVisible: false,
        dialogFormVisible: false,
        goodsSetVisible: false,
        newVisible: false,
        showVisible: false,
        formLabelWidth: "120px",
        goodsList: [],
        treeList: [],
        form: {},
        value: "",
        newEdit: {
          integralValue: "",
          skuId: "",
          status: 1,
          description: "",
          skuName: ""
        },
        newList: [],
        goodaAttrList: [],
        search: {
          pageIndex: 0,
          pageSize: 20,
          status: "",
          skuName: ""
        },
        goodsSearch: {
          pageIndex: 0,
          pageSize: 5,
          categoryId: ""
        },
        tableData: [],
        multipleSelection: [],
        showData: {}
      };
    },
    created() {
      this.getList();
    },
    methods: {
      status(data) {
        switch (data.status) {
          case 1:
            return "有效";
            break;
          case 2:
            return "无效";
            break;
        }
      },
      getList() {
        this.loading = true;
        let that = this;
        this.$http({
          method: "post",
          url: url + "/clientFindProductIntegralList",
          data: that.search
        }).then(function (res) {
          that.loading = false;
          that.tableData = res.data.data;
          console.log(res.data.data);
        });
      },
      getNewList() {
        this.newVisible = true;
        this.productGetAllRootCategoryList();
      },
      // 获取分类列表
      productGetAllRootCategoryList() {
        let that = this;
        this.$http({
          method: "post",
          url: url + "/productGetAllRootCategoryList"
        }).then(function (res) {
          that.newList = JSON.parse(res.data).data;
        });
      },
      handleCurrentChange(val) {
        this.search.pageIndex = val - 1;
        this.getList();
      },
      //商品下一页
      goodsChange() {
        this.goodsSearch.pageIndex = val - 1;
        this.getgoodsList();
      },
      //查询
      searchgoods() {
        this.getList();
      },
      //显示sku选择
      skuIdFun(event) {
        this.goodsSetVisible = true;
        let that = this;
        this.$http({
          method: "post",
          url: url + "/clientProductCategoryTreeData"
        }).then(function (res) {
          that.treeList = JSON.parse(res.data);
        });
      },
      //树节点点击
      treeClick(data) {
        this.goodsSearch.categoryId = data.id;
        if (data.isLeaf == 1) {
          this.getgoodsList();
          this.goodaAttrList = [];
        }
      }, //获取商品列表
      getgoodsList() {
        let that = this;
        this.$http({
          method: "post",
          url: url + "/clientFindProductListPaged",
          data: this.goodsSearch
        }).then(function (res) {
          that.goodsList = JSON.parse(res.data);
        });
      },
      // 获取商品属性列表
      getGoodsAttr(row) {
        let that = this;
        this.$http({
          method: "post",
          url: url + "/clientFindSkuListByProductId",
          data: {
            productId: row.productId
          }
        }).then(function (res) {
          // that.goodsList = JSON.parse(res.data)
          let goodaAttrList = res.data.data;
          that.goodaAttrList = goodaAttrList;
        });
      },
      getSkuSle(row) {
        this.goodsSetVisible = false
        this.newEdit.skuId = row.skuId
        this.newEdit.skuName = row.skuName
        console.log(row)
      },
      //新增确定
      sendNewAdd() {
        let that = this;
        this.$http({
          method: "post",
          url: url + "/clientModifyProductIntegral",
          data: that.newEdit
        }).then(function (res) {
          if (res.data.apiStatus == 1) {
            that.newVisible = false;
            that.newEdit= {
              integralValue: "",
                skuId: "",
                status: 1,
                description: "",
                skuName: ""
            }
            that.getList();
            that.$message({
              message: res.data.msg,
              type: "success"
            });
          } else {
            that.$message.error(res.data.msg);
          }
        });
      },
      //编辑确定
      sendEditAdd() {
        let that = this;
        this.$http({
          method: "post",
          url: url + "/clientModifyProductIntegral",
          data: that.form
        }).then(function (res) {
          if (res.data.apiStatus == 1) {
            that.dialogFormVisible = false;
            that.getList();
            that.$message({
              message: res.data.msg,
              type: "success"
            });
          } else {
            that.$message.error(res.data.msg);
          }
        });
      },

      // 编辑分类信息
      clientProductModifySortInfo() {
        let that = this;
        this.$http({
          method: "post",
          url: url + "/clientProductModifySortInfo",
          data: that.form
        }).then(function (res) {
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

      del(row) {
        let that = this;
        this.$confirm("此操作将删除, 是否继续?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            that.fullscreenLoading = true;
            that
              .$http({
                method: "post",
                url: url + "/clientDeleteProductIntegral",
                data: {
                  productIntegralId: row.productIntegralId
                }
              })
              .then(function (res) {
                if (res.data.apiStatus == 1) {
                  that.fullscreenLoading = false;
                  that.getList();
                  that.$message({
                    message: res.data.msg,
                    type: "success"
                  });
                } else {
                  that.$message.error(res.data.msg);
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
      switchStatus(row, status) {
        let that = this;
        this.$confirm("是否更改状态, 是否继续?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {

            that
              .$http({
                method: "post",
                url: url + "/clientModifyProductIntegral",
                data: {
                  productIntegralId: row.productIntegralId,
                  status: status
                }
              })
              .then(function (res) {
                if (res.data.apiStatus == 1) {
                  that.getList();
                  that.$message({
                    message: res.data.msg,
                    type: "success"
                  });
                } else {
                  that.$message.error(res.data.msg);
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
      handleEdit(row) {
        this.dialogFormVisible = true;
        var _item = {};
        for (var i in row) {
          _item[i] = row[i];
        }
        this.form = _item;
        console.log(row);
      },
      handleSelectionChange(val) {
        this.multipleSelection = val;

        // console.log(this.multipleSelection)
      },
      showDataFun(row) {
        this.showVisible = true;
        let that = this;
        this.$http({
          method: "post",
          url: url + "/clientFindIntegralRuleByRuleId",
          data: {
            integralRuleId: row.integralRuleId
          }
        }).then(function (res) {
          that.showData = res.data.data;
          // console.log(that.showData);
          console.log(that.showData);
        });
      }

    }
  };
</script>

<style lang="scss" scoped>

</style>
<style lang="scss">
  .goodsIntegral {
    .demo-table-expand {
      font-size: 0;
    }
    .demo-table-expand label {
      width: 90px;
      color: #99a9bf;
    }
    .demo-table-expand .el-form-item {
      margin-right: 0;
      margin-bottom: 0;
      width: 50%;
    }
    .lists {
      .el-form-item {
        text-align: center;
        margin-right: 0;
        margin-bottom: 0;
        width: 100%;
      }
    }
    .attrdata .el-input__inner {
      width: 100px;
    }
    .el-table .cell {
      text-align: center;
    }
    .avatar-uploader .el-upload {
      border: 1px dashed #d9d9d9;
      border-radius: 6px;
      cursor: pointer;
      position: relative;
      overflow: hidden;
    }

    .avatar-uploader .el-upload:hover {
      border-color: #20a0ff;
    }

    .avatar-uploader-icon {
      font-size: 28px;
      color: #8c939d;
      width: 178px;
      height: 178px;
      line-height: 178px;
      text-align: center;
    }

    .avatar {
      width: 178px;
      height: 178px;
      display: block;
    }
    .allList {
      .el-form-item__content {
        width: 70%;
      }
    }

    .el-table th > .cell {
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
    .el-input__inner {
      width: 300px;
    }
  }
</style>



