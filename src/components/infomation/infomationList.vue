<template>
  <div class="adPosition" v-loading.fullscreen.lock="fullscreenLoading">
    <el-breadcrumb separator="/" style="margin-bottom:10px">
      <el-breadcrumb-item :to="{ path: '/' }"> 资讯列表</el-breadcrumb-item>
      <el-breadcrumb-item>资讯管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索 -->
    <div class="inputs">
      <el-form :inline="true" :model="search" class="demo-form-inline">
        <el-form-item label="资讯标题">
          <el-input v-model="search.title" placeholder="请输入"></el-input>
        </el-form-item>
        <el-form-item label="资讯类型">
          <el-select v-model="search.type">
            <el-option label="全部" value=""></el-option>
            <el-option label="新闻" value="1"></el-option>
            <el-option label="公告" value="2"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="searchgoods">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="warning" @click="newEdits">新增资讯</el-button>
        </el-form-item>
      </el-form>
    </div>
    <!-- 列表展示 -->
    <el-table ref="multipleTable" :data="tableData.dataList" border style="width: 100%" v-loading="loading">
      <el-table-column type="index" width="50">
      </el-table-column>
      <el-table-column prop="title" label="标题">
        <template scope="scope">
          <el-button type="text" size="small" @click="showDel(scope.row)">
            {{scope.row.title}}
          </el-button>
        </template>
      </el-table-column>
      <el-table-column prop="typeId" :formatter="judge" label="类型" >
      </el-table-column>
      <el-table-column prop="pv" label="浏览量">
      </el-table-column>
      <el-table-column prop="releaseTime" label="发布时间">
      </el-table-column>
      <el-table-column prop="createDate" label="创建日期">
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

    <!-- 新建资讯弹出层 -->
    <el-dialog class="Edit" title="新建资讯" :visible.sync="newdialogFormVisible">
      <el-form :model="newEdit">
        <el-form-item label="资讯标题:" :label-width="formLabelWidth">
          <el-input v-model="newEdit.title" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="资讯类型:" :label-width="formLabelWidth">
          <el-select v-model="newEdit.typeId">
            <el-option label="新闻" value="1"></el-option>
            <el-option label="公告" value="2"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="资讯内容:" :label-width="formLabelWidth">
          <vue-editor v-model="newEdit.content"></vue-editor>
        </el-form-item>
        <el-row :gutter="20">
          <el-col :span="10">
            <el-form-item label="PC端大图:" :label-width="formLabelWidth">
              <el-upload class="avatar-uploader" :action="url+'/imageUpload'" name="file"  :data="datas" :show-file-list="false" :on-success="uploadSussBig" :before-upload="beforeAvatarUpload">
                  <img v-if="imageUrl.big" :src="imageUrl.big" class="avatar">
                  <i v-else class="el-icon-plus avatar-uploader-icon"></i>
              </el-upload>
              <span style="color:red;">*只能上传jpg/png文件<br>图片宽高比例保持3:1</span>
            </el-form-item>
          </el-col>
          <el-col :span="10">
            <el-form-item label="APP端小图:" :label-width="formLabelWidth">
               <el-upload class="avatar-uploader" :action="url+'/imageUpload'" name="file"  :data="datas" :show-file-list="false" :on-success="uploadSussSml"  :before-upload="beforeAvatarUpload">
                  <img v-if="imageUrl.sml" :src="imageUrl.sml" class="avatar">
                  <i v-else class="el-icon-plus avatar-uploader-icon"></i>
              </el-upload>
              <span style="color:red;">*只能上传jpg/png文件<br>图片宽高比例保持3:1</span>
            </el-form-item>
          </el-col>
        </el-row>


      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="newdialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="sendNew">确 定</el-button>

      </div>
    </el-dialog>

    <!-- 编辑弹出层 -->
    <el-dialog class="Edit" title="资讯编辑" :visible.sync="dialogFormVisible">
      <el-form :model="form">

        <el-form-item label="资讯名称:" :label-width="formLabelWidth">
          <el-input v-model="form.title" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="资讯类型:" :label-width="formLabelWidth">
          <el-select v-model="form.typeId" >
            <el-option label="新闻" :value="1"></el-option>
            <el-option label="公告" :value="2"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="资讯内容:" :label-width="formLabelWidth">
          <vue-editor v-model="form.content"></vue-editor>
        </el-form-item>

        <el-row :gutter="20">
          <el-col :span="10">
            <el-form-item label="PC端大图:" :label-width="formLabelWidth">
              <el-upload class="avatar-uploader" :action="url+'/imageUpload'" name="file"  :data="datas" :show-file-list="false" :on-success="editUploadSussBig" >
                  <img v-if="imageUrl.big" :src="imageUrl.big" class="avatar">
                  <i v-else class="el-icon-plus avatar-uploader-icon"></i>
              </el-upload>
            </el-form-item>
          </el-col>
          <el-col :span="10">
            <el-form-item label="APP端小图:" :label-width="formLabelWidth">
               <el-upload class="avatar-uploader" :action="url+'/imageUpload'" name="file"  :data="datas" :show-file-list="false" :on-success="editUploadSussSml" >
                  <img v-if="imageUrl.sml" :src="imageUrl.sml" class="avatar">
                  <i v-else class="el-icon-plus avatar-uploader-icon"></i>
              </el-upload>
            </el-form-item>
          </el-col>
        </el-row>


      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="clientProductModifySortInfo">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 展示单个资讯详情 -->
    <el-dialog title="资讯详情" :visible.sync="showVisible">
      <el-form>
        <el-form-item label="资讯标题" :label-width="formLabelWidth">
          <el-input v-model="showData.title" auto-complete="off" disabled></el-input>
        </el-form-item>

        <el-form-item label="资讯类型" :label-width="formLabelWidth">
          <el-select v-model="showData.typeId" disabled>
            <el-option label="新闻" :value="1"></el-option>
            <el-option label="公告" :value="2"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="资讯内容" :label-width="formLabelWidth">
          <el-input type="textarea" v-model="showData.content" auto-complete="off" disabled></el-input>
        </el-form-item>

        <el-row :gutter="20">
          <el-col :span="10">
            <el-form-item label="PC端大图:" :label-width="formLabelWidth">
              <img :src="url+ '/'+imageUrl.big" class="avatar">
            </el-form-item>
          </el-col>
          <el-col :span="10">
            <el-form-item label="APP端小图:" :label-width="formLabelWidth">
               <img :src="url+ '/'+imageUrl.sml" class="avatar">
            </el-form-item>
          </el-col>
        </el-row>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="showVisible = false">取 消</el-button>
      </div>
    </el-dialog>

  </div>
</template>


<script>
import { VueEditor } from 'vue2-editor'
export default {
  components: {
    VueEditor
  },
  data() {
    return {
      url: '',
      loading: true,
      showVisible: false,
      fullscreenLoading: false,
      dialogFormVisible: false,
      newdialogFormVisible: false,
      formLabelWidth: "120px",
      imageUrl:{
        big:'',
        sml:''
      },
      datas: {
        fileType: 4,
      },
      form: {},
      value: "",
      newEdit: {
        typeId: "",
        title: "",
        content: "",
        pv: '',
        bigImg: "",
        smallImg: "",
      },
      showData: {},
      newList: [],
      search: {
        pageIndex: 0,
        pageSize: 10,
        title: "",
        type: ""
      },
      tableData: []
    };
  },
  created() {
     this.url = url
    this.getList();
  },
  methods: {
    judge(data) {
      //taxStatus 布尔值
      switch (data.typeId) {
        case 1:
          return "新闻";
          break;
        case 2:
          return "公告";
          break;
        default:
          return "无";
      }
    },
    getList() {
      this.loading = true;
      let that = this;
      this.$http({
        method: "post",
        url: url + "/clientFindNewsList",
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
    //新增资讯
    sendNew() {

      // console.log(this.newEdit.typeId)
      let that = this;
      this.$http({
        method: "post",
        url: url + "/clientSaveNewsInfo",
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
     //大图上传成功
    uploadSussBig(response, file, fileList) {
      this.imageUrl.big = URL.createObjectURL(file.raw);//获得本地图片路径，用户显示
      this.newEdit.bigImg = response.data//把上传成功的路径存入数据库
    },
    //上传小图
    uploadSussSml(response, file, fileList) {
      this.imageUrl.sml = URL.createObjectURL(file.raw);
      this.newEdit.smallImg = response.data
    },
       //大图更新成功
    editUploadSussBig(response, file, fileList) {
      this.imageUrl.big = URL.createObjectURL(file.raw);//获得本地图片路径，用户显示
      this.form.bigImg = response.data//把上传成功的路径存入数据库
    },
    //小图更新成功
   editUploadSussSml(response, file, fileList) {
      this.imageUrl.sml = URL.createObjectURL(file.raw);
      this.form.smallImg = response.data
    },
    // 编辑完成确定
    clientProductModifySortInfo() {
      let that = this;
      this.$http({
        method: "post",
        url: url + "/clientSaveNewsInfo",
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
        console.log(res)
      });
    },
    //删除
    del(row) {
      let that = this;
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          that.fullscreenLoading = true;
          that
            .$http({
              method: "post",
              url: url + "/clientDeleteNewsInfo",
              data: {
                nId: row.nId
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
      let that = this;
      this.$http({
        method: "post",
        url: url + "/clientFindNewsInfo",
        data: {
          nId: row.nId
        }
      }).then(function(res) {
        let data = JSON.parse(res.data).data;
        that.form = data;


        that.imageUrl={
         big:url+'/'+data.bigImg,
         sml:url+'/'+data.smallImg
        }
        console.log(that.form)

      });

    },// 显示展示
    showDel(row) {
      this.showVisible = true
      let that = this
      this.$http({
        method: 'post',
        url: url + "/clientFindNewsInfo",
        data: {
          nId: row.nId
        }
      }).then(function(res) {
        //that.showData = res.data.data.nId
        let data = JSON.parse(res.data).data;
        that.showData = data;
        that.imageUrl={
         big:data.bigImg,
         sml:data.smallImg
        }

        console.log(that.imageUrl)
      })
    },// 只能上传jpg、png图片，且比例位3:1
    beforeAvatarUpload(file) {
      const isJPG = file.type === 'image/jpeg';
      const isPNG = file.type === 'image/png';
      const isLt2M = file.size / 1024 / 1024 < 2;
      let imgBili = (file.innerWidth/file.innerHeigh) == 3;
      console.log(imgBili)
      if (!isJPG && !isPNG) {
        this.$message.error('只能上传jpg/png文件!');
      }else if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!');
      }else if(imgBili!=3){
        this.$message.error('图片宽高比例需要保持3:1!');
      }
      return  isLt2M && imgBili && (isJPG || isPNG);
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



<style>
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
</style>
