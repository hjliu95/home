<template>
    <div class="list-page-main">
      <h2 class="lpm-title">数据源列表</h2>
      <div class="lpm-content">
        <div class="lpm-content-top">
          <el-button type="primary" @click="addConfig">新建</el-button>
          <div>
            <el-input
              v-model="queryText"
              clearable
              placeholder="请输入查询关键字"
              style="width: 200px; margin-right: 20px"
            ></el-input>
            <el-button type="primary" @click="queryList">查询</el-button>
          </div>
        </div>
        <el-table v-loading="loading" :data="tableDataObject.list">
          <el-table-column prop="name" label="数据源名称" />
          <el-table-column prop="id" label="数据源ID" />
          <el-table-column prop="remark" label="数据源备注" />
          <el-table-column prop="createDate" label="创建日期" />
          <el-table-column prop="updateDate" label="更新时间" />
          <el-table-column prop="operate" label="操作" width="180">
            <template #default="scope">
              <el-button type="primary" text @click="handleDetail(scope.row)"
                >编辑</el-button
              >
              <el-button type="primary" text @click="handleDelete(scope.row)"
                >删除</el-button
              >
            </template>
          </el-table-column>
        </el-table>
        <div v-if="tableDataObject.list.length" class="pagination-footer">
          <el-pagination
            v-model:currentPage="pageIndex"
            v-model:pageSize="pageSize"
            :page-sizes="[10, 20, 50, 100]"
            layout="total,prev,pager,next,sizes,jumper"
            :total="tableDataObject.total"
            background
            @size-change="pageSizeChange"
            @current-change="pageIndexChange"
          ></el-pagination>
        </div>
      </div>
    </div>
    <el-dialog v-model="dialogFormVisible" :title="isAdd ? '新增数据源' : '编辑数据源'" width="500">
      <el-form :model="form">
        <el-form-item label="数据源名称">
          <el-input v-model="form.name" autocomplete="off" />
        </el-form-item>
        <el-form-item label="数据源备注">
          <el-input v-model="form.remark" autocomplete="off" />
        </el-form-item>
        <!-- <el-form-item label="Zones" :label-width="formLabelWidth">
          <el-select v-model="form.region" placeholder="Please select a zone">
            <el-option label="Zone No.1" value="shanghai" />
            <el-option label="Zone No.2" value="beijing" />
          </el-select>
        </el-form-item> -->
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取消</el-button>
          <el-button type="primary" @click="saveConfig" :loading="confirmLoading">
            确定
          </el-button>
        </div>
      </template>
    </el-dialog>
  </template>
  <script setup lang="ts">
  import { onMounted, ref, reactive } from "vue";
  import { usePagination } from "@/composition/use-pagination";
  import { ElMessageBox } from "element-plus";
  import { ElMessage } from "element-plus";
  import apiServer from "@/utils/axios";
  import { log } from "console";
  
  const loading = ref(false);
  const dialogFormVisible = ref(false);
  const queryText = ref("");
  const isAdd = ref(false)
  const form = reactive({
    name: "",
    remark: "",
    id: ""
  });
  async function findPage() {
    loading.value = true;
    const payload = {
      pageIndex: pageIndex.value,
      pageSize: pageSize.value,
      queryText: queryText.value,
    };
    try {
      console.log(JSON.stringify(payload, null, 2));
      // await new Promise((r) => setTimeout(r, 500));
      const res: any = await apiServer.get("/dataSource/list", {
        params: payload,
      });
      console.log("res", res);
  
      // const res = { list: [{}, {}], total: 100 };
      tableDataObject.list = res.list;
      tableDataObject.total = res.total;
    } catch (e) {
      console.log(e);
    } finally {
      loading.value = false;
    }
  }
  
  const {
    pageIndex,
    pageSize,
    tableDataObject,
    resetPage,
    pageSizeChange,
    pageIndexChange,
  } = usePagination({
    defaultPageSize: 20,
    sizeChange: () => findPage(),
    currentChange: () => findPage(),
  });
  
  function queryList() {
    resetPage();
    findPage();
  }
  
  onMounted(() => {
    findPage();
  });
  

  async function handleDetail(row: any) {
    let { id, name } = row;
    console.log(row, 'row====>')
    isAdd.value = false
    const data: any = await apiServer.get(`/dataSource/detail/${id}`);
    form.name = data.name
    form.id = data.id
    form.remark = data.remark
    dialogFormVisible.value = true
  }
  
  async function handleDelete(row: any) {
    try {
      const title = "提示";
      const msg = "您确认要删除吗？";
      const options: any = { type: "info" };
      await ElMessageBox.confirm(msg, title, options);
      loading.value = true;
      // ElMessage.info("TODO!");
      const { id } = row
      const res = await apiServer.post('/dataSource/delete', {id})
      await new Promise((r) => setTimeout(r, 500));
      findPage();
      ElMessage.success("删除成功!");
    } catch (e) {
      console.log(e);
    } finally {
      loading.value = false;
    }
  }
  
  function addConfig() {
    dialogFormVisible.value = true;
    isAdd.value = true
  }
  function resetAddForm() {
    dialogFormVisible.value = false;
    Object.assign(form, {
      name: "",
      remark: "",
      id: ""
    });
  }
  
  const confirmLoading = ref(false);
  async function saveConfig() {
    try {
      confirmLoading.value = true;
      if (!form.name.trim()) {
        ElMessage.warning('请输入配置名称')
        return
      }
      if(isAdd.value) {
        const data: any = await apiServer.post("/dataSource/add", {
          name: form.name,
          remark: form.remark,
        });
      } else {
        const data: any = await apiServer.post("/dataSource/update", {
          name: form.name,
          remark: form.remark,
          id: form.id
        });
      }
      
      // success
      ElMessage.success(`${isAdd.value ? '添加' : '修改'}成功!`);
      findPage();
      resetAddForm();
      // "data": {
      //     "name": "3343",
      //     "remark": "3433434sdfdsf ",
      //     "bid": "4dl91VMMBL8vWuOh",
      //     "createDate": "2024/03/15 00:11:02",
      //     "_id": "65f321961effb4ffb79e29b6"
      // },
      // gotoConfigPage(data);
    } catch (e) {
      console.log(e);
    } finally {
      confirmLoading.value = false;
    }
    // console.log("res", res);
    // gotoConfigPage();
  }
  
  function gotoConfigPage({ bid, name }) {
    const newName = encodeURIComponent(name);
    //   window.open(`http://127.0.0.1:3000?bid=${randomId}`, "_blank");
    window.open(
      `${import.meta.env.VITE_DESIGN_URL}?bid=${bid}&name=${newName}`,
      "_blank"
    );
  }
  
  function preview({ bid, name }) {
    // ElMessage.info("预览");
    window.open(
      `${import.meta.env.VITE_VUE3_PREVIEW}/view?bid=${bid}&name=${name}`,
      "_blank"
    );
  }
  </script>
  
  <style lang="scss" scoped>
  .lpm-title {
    margin: 0 0 20px;
    padding-top: 20px;
    color: #262626;
    font-size: 18px;
    font-weight: bold;
    line-height: 24px;
  }
  
  .lpm-content {
    border-radius: 8px;
    //   padding: 20px;
    //  box-sizing: border-box;
    //   width: calc(100% - 40px);
    min-height: 500px;
    margin: 0 auto 20px;
    background: #fff;
    // box-shadow: 0 1px 4px -1px rgba(0, 0, 0, 0.05);
    .lpm-content-top {
      display: flex;
      justify-content: space-between;
      margin-bottom: 20px;
    }
  }
  
  .pagination-footer {
    margin-top: 16px;
    display: flex;
    justify-content: flex-end;
  }
  
  :deep(table.el-table__header) {
    margin: 0;
  }
  :deep(table.el-table__body) {
    margin: 0;
  }
  </style>
  