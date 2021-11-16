<template>
  <div class="app-container">
    <el-card shadow="never">
      <div class="title_body">
        <img :src="detail.cover" alt="" />
        <div class="text_body" style="flex: 1">
          <div
            class="text"
            style="display: flex; justify-content: space-between"
          >
            <h3>{{ detail.title }}</h3>
            <small>{{ detail.isSend ? "已完结" : "连载中" }}</small>
          </div>
          <p>
            <small>{{ detail.try }}</small>
          </p>
          <p style="color: red">¥{{ detail.price }}</p>
          <el-button-group>
            <el-button type="warning">{{
              detail.status ? "下架" : "上架"
            }}</el-button>
            <el-button type="info">{{
              detail.isSend ? "连载中" : "已完结"
            }}</el-button>
          </el-button-group>
        </div>
      </div>
    </el-card>

    <!-- 新增操控栏-->
    <div
      class="filter-container"
      style="display: flex; justify-content: space-between"
    >
      <el-button
        class="filter-item"
        style="margin-left: 10px"
        type="primary"
        icon="el-icon-edit"
      >
        新增目录
      </el-button>
      <div>
        <el-select
          v-model="listQuery.status"
          style="width: 110px"
          class="filter-item"
          @change="handleFilter"
          placeholder="商品状态"
        >
          <el-option
            v-for="(item, k) in statusOptions"
            :key="k"
            :label="item"
            :value="k"
          />
        </el-select>
        <el-input
          v-model="listQuery.title"
          placeholder="标题"
          style="width: 200px; margin-left: 10px"
          class="filter-item"
          @keyup.enter.native="handleFilter"
        />
        <el-button
          v-waves
          class="filter-item"
          type="primary"
          icon="el-icon-search"
          style="margin-left: 10px"
          @click="handleFilter"
        >
          搜索
        </el-button>
      </div>
    </div>
    <!-- 新增拖动表格 -->
    <el-table
      ref="dragTable"
      v-loading="listLoading"
      :data="list"
      row-key="id"
      border
      fit
      highlight-current-row
      style="width: 100%"
    >
      <!-- 序号列 -->
      <el-table-column label="序号" type="index" align="center" width="80">
      </el-table-column>
      <!-- 单品名称列 -->
      <el-table-column label="单品名称" min-width="200px">
        <template slot-scope="{ row }">
          <div style="display: flex">
            <img
              src="row.cover"
              style="width: 100px; height: 50px; margin-right: 10px"
            />
            <div
              style="
                display: flex;
                flex-direction: column;
                justify-content: space-between;
              "
            >
              <span>{{ row.title }}</span>
              <span style="color: orange">¥{{ row.price }}</span>
            </div>
          </div>
        </template>
      </el-table-column>
      <!-- 类型列 -->
      <el-table-column min-width="150px" label="类型">
        <template slot-scope="{ row }">
          <span>{{ row.title }}</span>
        </template>
      </el-table-column>
      <!-- 订阅量 -->
      <el-table-column label="订阅量" align="center" width="100">
        <template slot-scope="{ row }">
          <span
            v-if="row.sub_count"
            class="link-type"
            @click="handleFetchPv(row.sub_count)"
            >{{ row.sub_count }}</span
          >
          <span v-else>0</span>
        </template>
      </el-table-column>

      <el-table-column label="状态" class-name="status-col" width="100">
        <template slot-scope="{ row }">
          <el-tag :type="row.status ? 'seccess' : 'danger'">
            {{ row.status | statusFilter }}
          </el-tag>
        </template>
      </el-table-column>

      <el-table-column
        label="操作"
        align="center"
        width="230"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-popconfirm
            title="是否删除该记录？"
            @onConfirm="handleDelete(row, $index)"
            style="margin-left: 10px"
          >
            <el-button
              v-if="row.status != 'deleted'"
              size="mini"
              type="danger"
              slot="reference"
            >
              删除
            </el-button>
          </el-popconfirm>
        </template>
      </el-table-column>

      <el-table-column align="center" label="移动" width="80">
        <template slot-scope="{}">
          <svg-icon class="drag-handler" icon-class="drag" />
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>
<script>
let id = 0;
const statusOptions = {
  0: "已下架",
  1: "已上架",
};
import { fetchDetail, fetchDetailCourse } from "@/api/column.js";
import waves from "@/directive/waves"; // waves directive
import Sortable from "sortablejs";
export default {
  directives: { waves },
  filters: {
    statusFilter(status) {
      return statusOptions[status];
    },
  },
  beforeRouteEnter(to, from, next) {
    next();
    id = to.query.id;
  },
  handleUpdate(row) {
    this.temp = Object.assign({}, row); // copy obj
    this.temp.timestamp = new Date(this.temp.timestamp);
    this.dialogStatus = "update";
    this.dialogFormVisible = true;
    this.$nextTick(() => {
      this.$refs["dataForm"].clearValidate();
    });
  },
  handleDelete(row, index) {
    deleteMedio(row).then((response) => {
      this.$notify({
        title: "提示",
        message: "删除成功",
        type: "success",
        duration: 2000,
      });
      this.list.splice(index, 1);
    });
  },
  data() {
    return {
      list: [],
      listLoading: true,
      detail: {
        content: "",
        cover: "",
        created_time: "",
        id: 0,
        isSend: 0,
        price: 0,
        status: 1,
        sub_count: 0,
        title: "",
        try: "",
        updata_time: "",
      },
      listQuery: {
        importance: undefined,
        title: undefined,
      },
      statusOptions,
    };
  },

  created() {
    this.getData();
    this.getList();
  },
  methods: {
    handleFilter() {
      //先默认为空
    },
    getData() {
      fetchDetail({
        id,
      }).then((res) => {
        // console.log(res);
        this.detail = res.data;
        // console.log(res);
      });
    },
    changeDetailStatus() {
      this.detail.status = this.detail.status ? 0 : 1;
    },
    changeDetailIsSend() {
      this.detail.isSend = this.detail.isSend ? 1 : 0;
    },
    async getList() {
      this.listLoading = true;
      const { data } = await fetchDetailCourse(this.listQuery);
      this.total = data.total;
      this.list = data.items;
      // console.log(this.list);
      this.listLoading = false;
      this.oldList = this.list.map((v) => v.id);
      this.newList = this.oldList.slice();
      this.$nextTick(() => {
        this.setSort();
      });
    },
    setSort() {
      const el = this.$refs.dragTable.$el.querySelectorAll(
        ".el-table__body-wrapper > table > tbody"
      )[0];
      this.sortable = Sortable.create(el, {
        ghostClass: "sortable-ghost", // Class name for the drop placeholder,
        setData: function (dataTransfer) {
          // to avoid Firefox bug
          // Detail see : https://github.com/RubaXa/Sortable/issues/1012
          dataTransfer.setData("Text", "");
        },
        onEnd: (evt) => {
          const targetRow = this.list.splice(evt.oldIndex, 1)[0];
          this.list.splice(evt.newIndex, 0, targetRow);

          // for show the changes, you can delete in you code
          const tempIndex = this.newList.splice(evt.oldIndex, 1)[0];
          this.newList.splice(evt.newIndex, 0, tempIndex);
        },
      });
    },
  },
};
</script>

<style>
.title_body {
  display: flex;
  align-items: flex-start;
  flex: 1;
}
.title_body img {
  width: 200px;
  height: 100px;
  margin-right: 20px;
}
/* .title_body .text-body {
      flex: 1;

    }
    .title_body .text-body .text {
      display: flex;
      justify-content: space-between;
      color:red
    } */
.text small {
  color: #bbb;
}
.text h3 {
  margin-top: 5px;
  margin-bottom: 0px;
}
.title_body p {
  margin: 8px 0;
}
.filter-container {
  margin-top: 20px;
}
.sortable-ghost {
  opacity: 0.8;
  color: #fff !important;
  background: #42b983 !important;
}
</style>

<style scoped>
.icon-star {
  margin-right: 2px;
}
.drag-handler {
  width: 20px;
  height: 20px;
  cursor: pointer;
}
.show-d {
  margin-top: 15px;
}
