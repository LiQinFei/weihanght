<template>
  <div class="roleManages" v-loading.fullscreen.lock="fullscreenLoading">
    <el-breadcrumb separator="/" style="margin-bottom:10px">
      <el-breadcrumb-item :to="{ path: '/' }"> 系统管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索 -->
    <div class="inputs">
      <el-form :inline="true" :model="search" class="demo-form-inline">
        <el-form-item label="角色名称">
          <el-input v-model="search.roleName" placeholder="请输入"></el-input>
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="searchgoods">查询</el-button>
        </el-form-item>
      </el-form>
    </div>
    <!-- 列表 -->
    <el-table ref="multipleTable" :data="tableData.dataList" border style="width: 100%" v-loading="loading">
      <el-table-column  type="index" width="50">
      </el-table-column>
      <el-table-column  prop="roleName" label="角色名称" width="150">
      </el-table-column>
      <el-table-column prop="sort" label="排序码" width="100" show-overflow-tooltip>
      </el-table-column>
      <el-table-column prop="deleteMark" label="删除标记" width="130">
      </el-table-column>
      <el-table-column prop="enabledMark" label="有效标志" width="130">
      </el-table-column>
      <el-table-column prop="createDate" label="创建日期" width="100" show-overflow-tooltip>
      </el-table-column>


      <el-table-column prop="description" label="备注" min-width="100" show-overflow-tooltip>
      </el-table-column>
      <el-table-column fixed="right" label="操作" width="150">
        <template scope="scope">
          <el-button @click="delGoods(scope.row)" v-if="scope.row.status == 5" type="text" size="small">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination @current-change="handleCurrentChange" :page-size="20" layout="prev, pager, next, jumper" :page-count="tableData.totalPage">
    </el-pagination>
  </div>
</template>


<script>
  export default {
    data() {
      return {
        loading: true,
        fullscreenLoading: false,
        dialogVisible: false,
        showVisible: false,
        formLabelWidth: "120px",
        form: {},
        value: "",
        search: {
          pageIndex: 0,
          pageSize: 20,
          roleName:''
        },
        tableData: [],
        showData: {},
        showUer: {},
        goodsList: []
      };
    },
    created() {
      this.getList();
    },
    methods: {
      //预约类型
      reservationType(data) {
        switch (data.reservationType) {
          case 1:
            return "汽车清洗";
            break;
          case 2:
            return "汽车维修";
            break;
          case 3:
            return "汽车保养";
            break;
          case 4:
            return "美容装潢";
            break;
        }
      },
      //预约状态
      status(data) {
        switch (data.status) {
          case 1:
            return "未确认";
            break;
          case 2:
            return "已确认";
            break;
          case 3:
            return "已取消";
            break;
          case 4:
            return "已完成";
            break;
        }
      },
      getList() {
        this.loading = true;
        let that = this;
        this.$http({
          method: "post",
          url: url + "/clientFindRoleListAndPaged",
          data: that.search
        }).then(function(res) {
          that.loading = false;
          that.tableData = JSON.parse(res.data);
          console.log(that.tableData);
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
      //删除
      delGoods(row) {
        let that = this;
        this.$confirm("是否删除?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            that
              .$http({
                method: "post",
                url: url + "/clientDeleteReservationOrder",
                data: {
                  roId: row.roId
                }
              })
              .then(function(res) {
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
      }
    }
  };
</script>

<style lang="scss" scoped>

</style>
<style lang="scss">
  .roleManages {
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



