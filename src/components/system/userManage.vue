<template>
  <div class="userManage" v-loading.fullscreen.lock="fullscreenLoading">
    <el-breadcrumb separator="/" style="margin-bottom:10px">
      <el-breadcrumb-item :to="{ path: '/' }"> 系统管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索 -->
    <div class="inputs">
      <el-form :inline="true" :model="search" class="demo-form-inline">
        <el-form-item label="用户名">
          <el-input v-model="search.userName" placeholder="请输入"></el-input>
        </el-form-item>
        <el-form-item label="用户状态">
          <el-select v-model="search.userStatus" placeholder="全部">
            <el-option label="全部" value=""></el-option>
            <el-option label="在线" value="1"></el-option>
            <el-option label="离线" value="2"></el-option>
          </el-select>
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
      <el-table-column prop="account" label="登录账号" width="100" show-overflow-tooltip>
      </el-table-column>
      <el-table-column  prop="realname" label="真实姓名" width="150">
      </el-table-column>
      <el-table-column prop="simpleSpelling" label="简称" width="130">
      </el-table-column>
      <el-table-column prop="gender " :formatter="gender" label="性别 " width="90">
      </el-table-column>
      <el-table-column prop="status" label="在线状态" :formatter="userOnline" width="160">
      </el-table-column>
      <el-table-column prop="mobile" label="手机" width="160">
      </el-table-column>
      <el-table-column prop="dName" label="部门名称" width="160">
      </el-table-column>
      <el-table-column prop="post" label="岗位" min-width="100" show-overflow-tooltip>
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
          userName:"",
          userStatus:""
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
      gender(data) {
        switch (data.gender) {
          case 1:
            return "男";
            break;
          case 2:
            return "女";
            break;

        }
      },
      //预约状态
      userOnline (data) {
        switch (data.userOnline) {
          case 1:
            return "在线";
            break;
          case 2:
            return "离线";
            break;
        }
      },
      getList() {
        this.loading = true;
        let that = this;
        this.$http({
          method: "post",
          url: url + "/clientFindBaseUserList",
          data: that.search
        }).then(function(res) {
          that.loading = false;
          that.tableData = res.data.data
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
  .userManage {
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



