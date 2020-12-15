<template>
  <el-card>
    <el-form :inline="true" :model="cetegory" class="demo-form-inline">
      <el-form-item label="一级分类">
        <el-select
          v-model="cetegory.cetegory1ID"
          placeholder="请选择"
          @change="getCetegory2Id(cetegory.cetegory1ID)"
          :disabled='disabled'
        >
          <el-option
            :label="cetegory1.name"
            :value="cetegory1.id"
            v-for="cetegory1 in cetegory1List"
            :key="cetegory1.id1"
            
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="二级分类">
        <el-select
          v-model="cetegory.cetegory2ID"
          placeholder="请选择"
          @change="getCetegory3Id(cetegory.cetegory2ID, cetegory.cetegory3ID)"
          :disabled='disabled'
        >
          <el-option
            :label="cetegory2.name"
            :value="cetegory2.id"
            v-for="cetegory2 in cetegory2List"
            :key="cetegory2.id"
            
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="三级分类">
        <el-select
          v-model="cetegory.cetegory3ID"
          placeholder="请选择"
          @change="getAttrList(cetegory.cetegory1ID,cetegory.cetegory2ID,cetegory.cetegory3ID)"
          :disabled='disabled'
        >
          <el-option
            :label="cetegory3.name"
            :value="cetegory3.id"
            v-for="cetegory3 in cetegory3List"
            :key="cetegory3.id"
            
          ></el-option>
        </el-select>
      </el-form-item>
    </el-form>
  </el-card>
</template>
<script>
export default {
  name: "category",
  data() {
    return {
      cetegory: {
        cetegory1ID: "", //一级分类选中
        cetegory2ID: "",
        cetegory3ID: "",
      },
      cetegory1List: [], //一级分类数据
      cetegory2List: [],
      cetegory3List: [],
      //定义一个变量
    };
  },
  props: {
    // 获取当前分类所有属性
    getAttrList: Function,
    disabled:Boolean
  },
  methods: {
    //获取二级分类数据
    async getCetegory2Id(cetegory1ID) {
      this.cetegory.cetegory2ID = "";
      this.cetegory.cetegory3ID = "";
      this.cetegory2List = [];
      this.cetegory3List = [];
      const result = await this.$API.attrs.getCategorys2(cetegory1ID);
      if (result.code === 200) {
        this.cetegory2List = result.data;
        this.$bus.$emit("clearList");
      }
    },
    //获取三级分类数据
    async getCetegory3Id(cetegory2ID, cetegory3ID) {
      this.cetegory.cetegory3ID = "";
      this.cetegory.cetegory3List = [];
      const result = await this.$API.attrs.getCategorys3(
        cetegory2ID,
        cetegory3ID
      );
      if (result.code === 200) {
        this.cetegory3List = result.data;
        this.$bus.$emit("clearList");
      }
    },
    
  },
  async mounted() {
    //获取一级分类数据
    const result = await this.$API.attrs.getCategorys1();
    if (result.code === 200) {
      this.cetegory1List = result.data;
    }
  },
};
</script>
<style lang='less' rel='stylesheet/less' scoped>
</style>