<template>
  <div>
    <!-- 添加商品按钮 -->
    <el-button type="primary" class="el-icon-plus" @click="add">
      添加</el-button
    >
    <!-- 商品展示表格 -->
    <el-table
      :data="ageList.records"
      border
      style="width: 100%; margin: 20px 0"
    >
      <el-table-column type="index" label="序号" width="80" align="center">
      </el-table-column>
      <el-table-column prop="tmName" label="品牌名称"> </el-table-column>
      <el-table-column prop="logoUrl" label="品牌LOGO">
        <template v-slot="{ row }">
          <img :src="row.logoUrl" style="width: 130px" />
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template v-slot="{ row }">
          <el-button
            type="warning"
            class="el-icon-edit"
            @click="alterDate(row)"
          >
            修改</el-button
          >
          <el-button type="danger" class="el-icon-delete" @click="del(row)">
            删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <!-- //分页 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page.sync="page"
      :page-sizes="[3, 6, 9]"
      :page-size.sync="limit"
      layout="prev, pager, next, jumper,->,sizes,total"
      :total="ageList.total"
    >
    </el-pagination>
    <!-- 添加品牌 或 修改-->
    <el-dialog title="添加品牌" :visible.sync="visible" width="50%">
      <el-form
        :rules="rules"
        ref="ruleForm"
        label-width="100px"
        :model="addBrand"
      >
        <el-form-item label="品牌名称" prop="brand">
          <el-input v-model="addBrand.tmName"></el-input>
        </el-form-item>
        <el-form-item label="品牌LOGO" prop="logoUrl">
          <!-- 上传图片区域 -->
          <el-upload
            class="avatar-uploader"
            :action="`${this.$BASE_API}/admin/product/fileUpload`"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img
              v-if="addBrand.logoUrl"
              :src="addBrand.logoUrl"
              class="avatar"
            />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
          <span>只能上传jpg/png文件，且不超过50kb</span>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="visible = false">取 消</el-button>
        <el-button type="primary" @click="resetForm('ruleForm')"
          >确 定</el-button
        >
      </span>
    </el-dialog>
    <!-- 删除弹窗 -->
    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
      <span>您确定要删除 {{ this.addBrand.tmName }} 吗</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="delTrue">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "TrademarkList",
  data() {
    return {
      //每页品牌商品数据
      ageList: {},
      //当前页码和显示参数
      page: 1,
      limit: 3,
      //弹框显示隐藏
      visible: false,
      //删除弹窗
      dialogVisible: false,
      //校验规则
      rules: {
        tmName: [
          { required: true, message: "请输入品牌名称", trigger: "blur" },
        ],
        logoUrl: [{ required: true, message: "请添加图片", trigger: "blur" }],
      },
      //图片url和品牌名称
      addBrand: {
        logoUrl: "",
        tmName: "",
      },
    };
  },
  methods: {
    //封装请求当前数据
    async reqGetPageList(page, limit) {
      const result = await this.$API.trademark.getPageList(page, limit);
      if (result.code === 200) {
        this.ageList = result.data;
        this.$message.success("获取数据成功");
      } else {
        this.$message.error("获取数据失败");
      }
    },
    //修改分页数触发
    handleSizeChange(limit) {
      this.reqGetPageList(this.page, limit);
      // this.limit = limit;
    },
    //修改当前页码触发
    handleCurrentChange(page) {
      this.reqGetPageList(page, this.limit);
      // this.page = page;
    },
    //点击确认是否表单校验成功发请求
    resetForm(formName) {
      this.$refs[formName].validate(async (valid) => {
        if (valid) {
          //验证成功
          if (this.addBrand.id) {
            //有ID就调修改接口
            const result = await this.$API.trademark.updateTrademark(
              this.addBrand
            );
            if (result.code === 200) {
              this.$message.success("修改成功");
              //重新获得当前页面数据
              this.reqGetPageList(this.page, this.limit);

              this.visible = false;
            }
          } else {
            //更新接口
            const result = await this.$API.trademark.addTrademark(
              this.addBrand
            );
            if (result.code === 200) {
              this.$message.success("更新成功");
              //重新获得当前页面数据
              this.reqGetPageList(this.page, this.limit);

              this.visible = false;
            }
          }
        } else {
          //验证失败

          return false;
        }
      });
    },
    //图片上传后验证成功
    handleAvatarSuccess(res, file) {
      this.addBrand.logoUrl = res.data;
    },
    //图片上传前验证
    beforeAvatarUpload(file) {
      console.log(file);
      const imgType = ["image/jpeg", "image/png", "image/jpg"];
      const isJPG = imgType.indexOf(file.type) > -1;
      const isLt50k = file.size / 1024 < 50;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt50k) {
        this.$message.error("上传头像图片大小不能超过 50kb!");
      }
      return isJPG && isLt50k;
    },
    //点击添加并清空数据并显示弹框
    add() {
      this.visible = true;
      this.addBrand = {
        logoUrl: "",
        tmName: "",
      };
    },
    //点击品牌修改数据
    alterDate({ id }) {
      console.log(id);
      const addBrand = this.ageList.records.find((record) => {
        return record.id === id;
      });
      this.addBrand = {
        ...addBrand,
      };
      this.visible = true;
    },
    //点击删除品牌数据
    del({ id }) {
      this.dialogVisible = true;
      const addBrand = this.ageList.records.find((record) => {
        return record.id === id;
      });
      this.addBrand = addBrand;
    },
    // 确定删除
    async delTrue() {
      const result = await this.$API.trademark.deleteTrademark(
        this.addBrand.id
      );

      
      console.log(this.page);
      if (result.code === 200) {

        this.dialogVisible = false;
        this.$message.success("删除数据成功");

        //重新获得当前页面数据
        console.log(555,this.ageList.records.length);
        this.page = this.ageList.records.length === 1 ? this.page - 1 : this.page
        console.log(666,this.ageList.records.length);
        console.log(111,this.page);
        this.reqGetPageList(this.page, this.limit);
        console.log(222,this.page);
      } else {
        this.$message.error("删除数据失败");
      }
    },
  },
  mounted() {
    //接收当前页参数发请求得到数据
    this.reqGetPageList(this.page, this.limit);
  },
};
</script>
<style scoped>
.el-pagination {
  text-align: center;
}
>>> .avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
>>> .avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
>>> .avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
>>> .avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
