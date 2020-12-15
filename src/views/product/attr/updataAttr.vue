<template>
  <el-card style="margin: 20px 0">
    <el-form :inline="true" :model="attr" class="demo-form-inline">
      <el-form-item label="属性名">
        <el-input v-model="attr.attrName" :disabled = 'attr.attrValueList.length > 0'></el-input>
      </el-form-item>
    </el-form>
    <el-button
      type="primary"
      class="el-icon-plus"
      @click="addAttrValue"
      :disabled="!attr.attrName"
    >
      添加属性值</el-button
    >
    <!-- 属性展示 -->
    <el-table
      border
      style="width: 100%; margin: 20px 0"
      :data="attr.attrValueList"
    >
      <el-table-column label="序号" align="center" type="index" width="80">
      </el-table-column>
      <el-table-column label="属性值名称">
        <!-- 添加的值 -->
        <template v-slot="{ row, $index }">
          <!--
              事件修饰符：
                .native
                专门给组件绑定事件使用的
                会给组件中的第一个标签绑定相应的原生DOM事件
            -->
          <el-input
            v-if="row.edit"
            v-model="row.valueName"
            @blur="editCompleted(row, $index)"
            @keyup.enter.native="editCompleted(row, $index)"
            autofocus
            ref="input"
            size="mini"
            
          ></el-input>
          <!-- 直接给对象添加新属性不是响应式数据, 通过this.$set添加的属性才是响应式 -->
          <span v-else @click="edit(row)" style="display: block; width: 100%">{{
            row.valueName
          }}</span>
        </template>
      </el-table-column>

      <el-table-column label="操作">
        <template v-slot='{row,$index}'>
          <el-button  class="el-icon-delete" type="danger" size="small" @click="delAttr(row,$index)"></el-button>
        </template>
        
      </el-table-column>
    </el-table>
    <el-button type="primary" @click="addAttr"> 保存</el-button>
    <el-button @click="clearAttr"> 取消</el-button>
  </el-card>
</template>
<script>
export default {
  name: "updataAttr",
  data() {
    return {
      attr: {
        attrName: "",
        attrValueList: [],
      },
    };
  },
  methods: {
    //取消切换组件并清空属性数据值数据
    clearAttr(){
      this.attr.attrName = "",
      this.attr.attrValueList = []
      this.isShowAttr(true)
    },
    //更新属性数据的方法
    updataAttrValue(attrValueList){
      
      this.attr = JSON.parse(JSON.stringify(attrValueList)) 
    },
    //删除当前属性
    delAttr(row,index){
      
      const {attrValueList} = this.attr
      this.attr.attrValueList.splice(index,1)
    },
    //点击显示输入框并聚焦
    edit(row){
      row.edit = true
      this.$nextTick(() => {
        this.$refs.input.focus();
      })
    },
    //添加属性
    addAttrValue() {
      this.attr.attrValueList.push({ edit: true });
      this.$nextTick(() => {
        this.$refs.input.focus();
      });
    },
    //回车和失去焦点如果数据没有值则删除
    editCompleted(row, index) {
      if (!row.valueName) {
        this.attr.attrValueList.splice(index, 1);
        return;
      }
      row.edit = false;
    },
    //点击保存添加属性
    async addAttr() {
      const data = this.attr;
      data.categoryId = this.category.category3Id;
      data.categoryLevel = 3;
      const result = await this.$API.attrs.saveAttrInfo(data);
      if(result.code === 200){
        //清空输入框数据
        this.attr.attrValueList = []
        this.attr.attrName = ''
        //显示属性列表
        this.isShowAttr(true)
        //更新属性列表
        this.getAttrList(this.category.category1Id,this.category.category2Id,this.category.category3Id)
        this.$message.success("更新属性成功~");
      }else{
        this.$message.error(result.message);
      }
    },
  },
  props: {
    //是否显示
    isShowAttr: Function,
    //三级分类
    category:Object,
    //获取属性列表
    getAttrList:Function
  },
  mounted () {
    this.$bus.$on('updataAttrValue',this.updataAttrValue)
    
  }
};
</script>
<style lang='less' rel='stylesheet/less' scoped>
</style>