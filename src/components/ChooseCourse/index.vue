<template>
  <div>
    课程详情
    <el-dialog
      class="dialog_body"
      title="选择课程"
      :visible.sync="dialogVisible"
      width="80%"
      top="5vh"
    >
      <el-container style="height:71vh margin-top:-30px; margin-button:-30px">
        <el-header>
          <div class="header">
            <el-input
              class="header_input"
              v-model="form.title"
              placeholder="标题"
              size="normal"
            ></el-input>
            <el-button type="primary" size="default" @click="headleFilter"
              >搜索</el-button
            >
          </div>
        </el-header>
        <el-container>
          <el-aside
            width="200px"
            style="height:60vh; 
          background-color: #fbfbfb; border:1px solid #eee;height:45vh; 
          padding:0px overflow-y:auto;"
          >
            <!-- 下拉框 -->
            <el-menu
              :default-active="key"
              @select="onAsideSelect"
              style="background-color: #fbfbfb; border: 0px"
            >
              <el-menu-item
                v-for="(item, index) in menus"
                :index="item.index"
                :key="index"
                >{{ item.name }}</el-menu-item
              >
            </el-menu>
          </el-aside>
          <el-container>
            <el-main style="height:45vh; padding:0px overflow-y:auto;">
              <el-table
                v-loading="listLoading"
                :data="list"
                fit
                highlight-current-row
                style="width: 100%"
                ref="multipleTable"
                @selection-change="handleSelectionChange"
              >
                <!-- 选择框 -->
                <el-table-column type="selection" width="55"> </el-table-column>
                <!-- 表格内容 -->
                <el-table-column label="内容" min-width="180px">
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
              </el-table>
            </el-main>
            <el-footer>
              <pagination
                :total="total"
                :page.sync="form.page"
                :limit.sync="form.limit"
                @pagination="getData"
                layout="total,prev,pager,next"
                style="padding=0; padding-top:10px margin:0px"
              />
            </el-footer>
          </el-container>
        </el-container>
      </el-container>
      <span slot="footer" class="dialog-footer">
        <el-button @click="close">取 消</el-button>
        <el-button type="primary" @click="confirm">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
let M = {}
function importAll(r){
  r.keys().forEach(key => M[key] = r(key))
}

importAll(require.context('@/api/',true,/\.js$/)) 

import { fetchList } from "@/api/media";
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination

export default {
  components: {
    Pagination,
  },
  data() {
    return {
      key: "media",
      menus: [
        {
          name: '图文',
          index: "media",
        },
        {
          name: "音频",
          index: "audio",
        },
        {
          name: "视频",
          index: "video",
        },
      ],
      // tablekey: 0 ,
      // :key="tableKey"

      list: [],
      dialogVisible: false,
      listLoading: true,
      multipleSelection: [],
      total: 0,
      callback:null,
      limit:-1,
      form: {
        title: "",
        page: 1,
        limit: 10,
      },
    };
  },
  methods: {
    getData() {
      this.listLoading = true;
      let fetchList = M[`./${this.key}.js`].fetchList
      fetchList(this.form).then((response) => {
        this.list = response.data.items;
        this.total = response.data.total;
        
        this.listLoading = false;
      });
    },
    open(callback,limit = -1) {
      this.getData();
      this.dialogVisible = true;
      this.limit=limit
      this.callback = callback;

    },
    close() {
      this.dialogVisible = false;
    },
    confirm() {
      if(this.multipleSelection === 0){
        return this.$message.error('你没有选中数据');
      }
      if(this.limit != -1 && this.multipleSelection > this.limit){
        return this.$message.error('最多可以选中'+this.limit+'个');
      }
      this.callback(this.multipleSelection);
      this.close();
      // 清空选中
      this.$refs.multipleTable.clearSelection()
    },
    headleFilter() {
      //header按钮搜索方法
      this.form.page = 1
      this.getData()
    },
    onAsideSelect(e) {
      //列表点击方法
      console.log(e);
      this.form.page = 1;
      this.key = e;
      this.getData();
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
  },
};
</script>

<style>
/* .dialog_body{
  top:5vh;
  margin-top:5px;
} */
/* .aside{
  height:60vh;
  background-color: #fbfbfb;
} */
.header {
  display: flex;
  justify-content: flex-end;
}
.header .header_input {
  width: 200px;
  margin-right: 10px;
}
</style>