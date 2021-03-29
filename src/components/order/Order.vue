<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row :gutter="10">
        <el-col :span="8">
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table :data="orderlist" stripe border>
        <el-table-column type="index"> </el-table-column>
        <el-table-column label="订单编号" prop="order_number">
        </el-table-column>
        <el-table-column label="订单价格" prop="order_price"> </el-table-column>
        <el-table-column label="是否付款">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.pay_status !== '0'" type="success"
              >已付款</el-tag
            >
            <el-tag v-else type="danger">未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send"> </el-table-column>
        <el-table-column label="下单时间">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showBox"
            ></el-button>
            <el-button
              type="success"
              icon="el-icon-location"
              size="mini"
              @click="showProgressBox"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 30, 50]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 修改地址对话框 -->
    <el-dialog
      title="修改地址"
      :visible.sync="addressVisible"
      width="50%"
      @close="addressDialogClosed"
    >
      <el-form
        ref="addressFormRef"
        :rules="addressFormRules"
        :model="addressForm"
        label-width="80px"
      >
        <el-form-item label="省市/区" prop="address1">
          <el-cascader :options="cityData" v-model="addressForm.address1">
          </el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressVisible = false">取 消</el-button>
        <el-button type="primary" @click="addressVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>

    <!-- 物流进度对话框 -->
    <el-dialog title="物流进度" :visible.sync="progressVisible" width="50%">
      <span>由于物流接口不可用了，暂无处理</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="progressVisible = false">取 消</el-button>
        <el-button type="primary" @click="progressVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
import cityData from "@/components/order/citydata";
export default {
  data() {
    return {
      // 请求数据时的查询对象
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10,
      },
      // 订单总数
      total: 0,
      // 订单列表
      orderlist: [],
      // 控制修改地址对话框的显示与隐藏
      addressVisible: false,
      addressForm: {
        address1: [],
        address2: "",
      },
      addressFormRules: {
        address1: [{ required: true, message: "请选择地区", trigger: "blur" }],
        address2: [
          { required: true, message: "请填写详细地址", trigger: "blur" },
        ],
      },
      cityData,
      // 控制物流进度对话框的显示与隐藏
      progressVisible: false,
      // 物流进度
      progressInfo: [],
    };
  },
  created() {
    this.getOrderList();
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get("orders", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取订单列表失败！");
      }
      console.log(res);
      this.total = res.data.total;
      this.orderlist = res.data.goods;
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getOrderList();
    },
    handleCurrentChange(newNum) {
      this.queryInfo.pagenum = newNum;
      this.getOrderList();
    },
    // 点击按钮，弹出修改地址对话框
    showBox() {
      this.addressVisible = true;
    },
    // 监听修改地址对话框的关闭事件
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields();
    },
    // 点击按钮，展示物流进度对话框
    async showProgressBox() {
      // 这个接口不可用了

      // const { data: res } = await this.$http.get("/kuaidi/1106975712662");
      // if (res.meta.status !== 200) {
      //   return this.$message.error("获取物流进度失败！");
      // }
      // this.progressInfo = res.data;
      // console.log(this.progressInfo);
      this.progressVisible = true;
    },
  },
};
</script>

<style scoped>
.el-cascader {
  width: 100%;
}
</style>