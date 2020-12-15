<template>
  <div>
    <!-- 分类 -->
    <Category :getAttrList="getAttrList" :disabled="disabled" />
    <UpdataAttr
      v-show="!isUpdata"
      :isShowAttr="isShowAttr"
      :category="category"
      :getAttrList="getAttrList"
    />
    <el-card style="margin: 20px 0" v-show="isUpdata">
      <el-button
        type="primary"
        class="el-icon-plus"
        @click="isShowAttr(false)"
        :disabled="!category.category3Id"
      >
        添加属性</el-button
      >
      <!-- 属性展示 -->
      <el-table :data="attrDada" border style="width: 100%; margin: 20px 0">
        <el-table-column label="序号" width="80" align="center" type="index">
        </el-table-column>
        <el-table-column prop="attrName" label="属性名称" width="160">
        </el-table-column>
        <el-table-column label="属性值列表">
          <template v-slot="{ row }">
            <el-tag
              v-for="attrValue in row.attrValueList"
              :key="attrValue.id"
              >{{ attrValue.valueName }}</el-tag
            >
          </template>
        </el-table-column>

        <el-table-column label="操作" width="160">
          <template v-slot="{ row }">
            <el-button
              class="el-icon-edit"
              type="warning"
              size="small"
              @click="isShowUpdata(row)"
            ></el-button>
            <el-popconfirm
              title="这是一段内容确定删除吗"
              @onConfirm="delAttr(row)"
            >
              <el-button
                class="el-icon-delete"
                type="danger"
                size="small"
                slot="reference"
              ></el-button>
            </el-popconfirm>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
import Category from "@/components/Category";
import UpdataAttr from "./updataAttr";
export default {
  name: "AttrList",
  data() {
    return {
      //所有属性数据
      attrDada: [],
      //是否显示添加属性
      isUpdata: true,
      category: {
        // 代表三个分类id数据
        category1Id: "",
        category2Id: "",
        category3Id: "",
      },
    };
  },
  computed: {
    disabled() {
      return this.isUpdata ? false : true;
    },
  },

  methods: {
    //切换到添加或修改
    isShowUpdata(row){
      this.isUpdata = false
      this.$bus.$emit('updataAttrValue',row)
    }
    ,
    //删除属性
    async delAttr(row) {
      const result = await this.$API.attrs.deleteAttr(row.id);
      if (result.code === 200) {
        this.$message.success("删除属性成功");
        const { category1Id, category2Id, category3Id } = this.category;
        this.getAttrList(category1Id, category2Id, category3Id);
        return;
      }
      this.$message.error(result.message);
    },
    //当选择第三级分类获取数据
    async getAttrList(category1Id, category2Id, category3Id) {
      const result = await this.$API.attrs.getAttrList({
        category1Id,
        category2Id,
        category3Id,
      });
      //获取ID值
      this.category = {
        category1Id,
        category2Id,
        category3Id,
      };
      if (result.code === 200) {
        this.$message.success("获取当前属性成功");
        this.attrDada = result.data;
      } else {
        this.$message.success(result.message);
      }
    },
    //点击添加属性
    isShowAttr(falg) {
      this.isUpdata = falg;
    },
    clearList() {
      // 清空数据
      // this.attrDada = [];
      // 禁用按钮
      this.category.category3Id = "";
    },
  },
  mounted() {
    this.$bus.$on("clearList", this.clearList);
  },
  components: {
    Category,
    UpdataAttr,
  },
};
</script>
