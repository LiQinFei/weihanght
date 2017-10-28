<template>
  <div class="memberEquity" v-loading.fullscreen.lock="fullscreenLoading">
    <el-breadcrumb separator="/" style="margin-bottom:10px">
      <el-breadcrumb-item :to="{ path: '/' }"> 会员管理</el-breadcrumb-item>
      <el-breadcrumb-item>会员权益</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索 -->
    <div class="inputs">
      <el-form :inline="true" :model="search" class="demo-form-inline">

        <el-form-item>
          <el-button type="warning" @click="newVisible = true">新增权益</el-button>
        </el-form-item>
      </el-form>
    </div>

    <!-- 列表 -->
    <el-table ref="multipleTable" :data="tableData.dataList" border style="width: 100%" v-loading="loading">
      <el-table-column type="index" width="50">
      </el-table-column>
      <el-table-column prop="benefitName" label="权益名称" width="150">
        <template scope="scope">
          <el-button type="text" size="small" @click="showDels(scope.row)">
            {{scope.row.benefitName}}
          </el-button>
        </template>
      </el-table-column>
      <el-table-column prop="level" label="等级" width="90">
      </el-table-column>
      <el-table-column prop="upgradeTotal" label="权益升级界限" width="130">
      </el-table-column>
      <el-table-column prop="enabledMark" label="有效标志" :formatter="enabledMark" width="160">
      </el-table-column>

      <el-table-column prop="description" label="权益说明" min-width="100" show-overflow-tooltip>
      </el-table-column>
      <el-table-column fixed="right" label="操作" width="150">
        <template scope="scope">
          <el-button @click="editShow(scope.row)" type="text" size="small">编辑</el-button>
          <el-button @click="delGoods(scope.row)" type="text" size="small">删除</el-button>
          <el-button @click="enabled(scope.row,2)" v-if="scope.row.enabledMark == 1" type="text" size="small">无效</el-button>
          <el-button @click="enabled(scope.row,1)" v-if="scope.row.enabledMark == 2" type="text" size="small">有效</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination @current-change="handleCurrentChange" :page-size="20" layout="prev, pager, next, jumper" :page-count="tableData.totalPage">
    </el-pagination>

    <!--新增-->
    <el-dialog title="新增权益" :visible.sync="newVisible" class="Edit">
      <el-form :model="newEdit">
        <el-form-item label="权益名称" :label-width="formLabelWidth">
          <el-input v-model="newEdit.benefitName" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="等级" :label-width="formLabelWidth">

          <el-input v-model="newEdit.level" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标" :label-width="formLabelWidth">
          <el-upload class="avatar-uploader" :action="url+'/imageUpload'" name="file" :data="datas" :show-file-list="false" :on-success="uploadSuss">
            <img v-if="imageUrl" :src="imageUrl" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </el-form-item>
        <el-form-item label="权益升级界限" :label-width="formLabelWidth">
          <el-input v-model="newEdit.upgradeTotal" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="权益说明" :label-width="formLabelWidth">
          <el-input v-model="newEdit.description" auto-complete="off"></el-input>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="newVisible = false">取 消</el-button>
        <el-button type="primary" @click="sendNewAdd">确 定</el-button>
      </div>
    </el-dialog>

    <!--编辑-->
    <el-dialog title="编辑权益" :visible.sync="editVisible" class="Edit">
      <el-form :model="form">
        <el-form-item label="权益名称" :label-width="formLabelWidth">
          <el-input v-model="form.benefitName" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="等级" :label-width="formLabelWidth">
          <el-input v-model="form.level" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标" :label-width="formLabelWidth">
          <el-upload class="avatar-uploader" :action="url+'/imageUpload'" name="file" :data="datas" :show-file-list="false" :on-success="edituploadSuss">
            <img v-if="imageUrl" :src="imageUrl" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </el-form-item>
        <el-form-item label="权益升级界限" :label-width="formLabelWidth">
          <el-input v-model="form.upgradeTotal" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="权益说明" :label-width="formLabelWidth">
          <el-input v-model="form.description" auto-complete="off"></el-input>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button type="primary" @click="editSure">确 定</el-button>
      </div>
    </el-dialog>


    <!--权益详情 -->
    <el-dialog title="权益详情" :visible.sync="showDelVisible">
      <el-form label-position="left" inline class="demo-table-expand">
        <el-form-item label="权益名称" :label-width="formLabelWidth">
          <span>{{showData.benefitName}}</span>
        </el-form-item>
        <el-form-item label="等级" :label-width="formLabelWidth">
          <span v-if="showData.level == 1">普通 </span>
          <span v-if="showData.level == 2">白金 </span>
          <span v-if="showData.level == 3">蓝金 </span>
          <span v-if="showData.level == 4">白钻 </span>
          <span v-if="showData.level == 5">蓝钻 </span>
        </el-form-item>
        <el-form-item label="权益升级界限" :label-width="formLabelWidth">
          <span>{{showData.upgradeTotal}}</span>
        </el-form-item>
        <el-form-item label="有效标志" :label-width="formLabelWidth">
          <span v-if="showData.enabledMark == 1">有效</span>
          <span v-if="showData.enabledMark == 2">无效</span>
        </el-form-item>
        <el-form-item label="权益说明" :label-width="formLabelWidth">
          <span>{{showData.description}}</span>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="showDelVisible = false">取 消</el-button>
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
        dialogVisible: false,
        showVisible: false,
        newVisible: false,
        editVisible: false,
        showDelVisible: false,
        formLabelWidth: "120px",
        form: {},
        value: "",
        search: {
          pageIndex: 0,
          pageSize: 20
        },
        tableData: [],
        showData: {},
        showUer: {},
        goodsList: [],

        newEdit: {
          benefitName: '',
          level: '',
          upgradeTotal: '',
          description: '',
        },
        datas: {
          fileType: 2
        },
        imageUrl: '',
        url: ''
      };
    },
    created() {
      this.url = url
      this.getList();
    },
    methods: {

      //预约状态
      enabledMark(data) {
        switch (data.enabledMark) {
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
          url: url + "/clientFindAccountBenefitList",
          data: that.search
        }).then(function (res) {
          that.loading = false;
          that.tableData = res.data.data

          console.log(res.data.data)
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
                url: url + "/clientDeleteAccountBenefit",
                data: {
                  abId: row.abId
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
//      有效无效
      enabled(row, status) {
        let that = this;
        this.$confirm("是否更改?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            that
              .$http({
                method: "post",
                url: url + "/clientUpdateAccountBenefitStatus",
                data: {
                  abId: row.abId,
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
      //确定新增
      sendNewAdd() {
        let that = this
        this.$http({
          method: "post",
          url: url + "/clientModifyAccountBenefit",
          data: this.newEdit
        }).then(function (res) {
          if (res.data.apiStatus == 1) {
            that.newVisible = false
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
      //确定编辑
      editSure() {
        let that = this
        this.$http({
          method: "post",
          url: url + "/clientModifyAccountBenefit",
          data: this.form
        }).then(function (res) {

          if (res.data.apiStatus == 1) {
            that.editVisible = false
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
      //上传成功
      uploadSuss(response, file, fileList) {
        this.imageUrl = URL.createObjectURL(file.raw);
        this.newEdit.icon = response.data;

      },
      edituploadSuss(response, file, fileList) {
        this.imageUrl = URL.createObjectURL(file.raw);
        this.form.icon = response.data;
      },
      //编辑弹出
      editShow(row) {
        this.editVisible = true
        let that = this
        this.$http({
          method: "post",
          url: url + "/clientFindAccountBenefitByAbId",
          data: {
            abId: row.abId
          }
        }).then(function (res) {
          that.imageUrl = url + '/' + res.data.data.icon
          that.form = res.data.data

        });
      },
      //详情展示
      showDels(row) {
        let that = this
        this.showDelVisible = true
        this.$http({
          method: "post",
          url: url + "/clientFindAccountBenefitByAbId",
          data: {
            abId: row.abId
          }
        }).then(function (res) {
          that.showData = res.data.data

        });
      }
    }
  };
</script>

<style lang="scss" scoped>

</style>
<style lang="scss">
  .memberEquity {
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

    .avatar-uploader .el-upload {
      border: 1px dashed #d9d9d9;
      border-radius: 6px;
      cursor: pointer;
      position: relative;
      overflow: hipositionTypeen;
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



