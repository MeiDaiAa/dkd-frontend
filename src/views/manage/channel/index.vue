<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryRef" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="货道编号" prop="channelCode">
        <el-input
          v-model="queryParams.channelCode"
          placeholder="请输入货道编号"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="商品Id" prop="skuId">
        <el-input
          v-model="queryParams.skuId"
          placeholder="请输入商品Id"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="售货机Id" prop="vmId">
        <el-input
          v-model="queryParams.vmId"
          placeholder="请输入售货机Id"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="售货机软编号" prop="innerCode">
        <el-input
          v-model="queryParams.innerCode"
          placeholder="请输入售货机软编号"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="货道最大容量" prop="maxCapacity">
        <el-input
          v-model="queryParams.maxCapacity"
          placeholder="请输入货道最大容量"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="货道当前容量" prop="currentCapacity">
        <el-input
          v-model="queryParams.currentCapacity"
          placeholder="请输入货道当前容量"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="上次补货时间" prop="lastSupplyTime">
        <el-date-picker clearable
          v-model="queryParams.lastSupplyTime"
          type="date"
          value-format="YYYY-MM-DD"
          placeholder="请选择上次补货时间">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="创建时间" prop="createTime">
        <el-date-picker clearable
          v-model="queryParams.createTime"
          type="date"
          value-format="YYYY-MM-DD"
          placeholder="请选择创建时间">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="修改时间" prop="updateTime">
        <el-date-picker clearable
          v-model="queryParams.updateTime"
          type="date"
          value-format="YYYY-MM-DD"
          placeholder="请选择修改时间">
        </el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="Plus"
          @click="handleAdd"
          v-hasPermi="['manage:channel:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="Edit"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['manage:channel:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="Delete"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['manage:channel:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="Download"
          @click="handleExport"
          v-hasPermi="['manage:channel:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar v-model:showSearch="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="channelList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="主键" align="center" prop="id" />
      <el-table-column label="货道编号" align="center" prop="channelCode" />
      <el-table-column label="商品Id" align="center" prop="skuId" />
      <el-table-column label="售货机Id" align="center" prop="vmId" />
      <el-table-column label="售货机软编号" align="center" prop="innerCode" />
      <el-table-column label="货道最大容量" align="center" prop="maxCapacity" />
      <el-table-column label="货道当前容量" align="center" prop="currentCapacity" />
      <el-table-column label="上次补货时间" align="center" prop="lastSupplyTime" width="180">
        <template #default="scope">
          <span>{{ parseTime(scope.row.lastSupplyTime, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <el-button link type="primary" icon="Edit" @click="handleUpdate(scope.row)" v-hasPermi="['manage:channel:edit']">修改</el-button>
          <el-button link type="primary" icon="Delete" @click="handleDelete(scope.row)" v-hasPermi="['manage:channel:remove']">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    
    <pagination
      v-show="total>0"
      :total="total"
      v-model:page="queryParams.pageNum"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改售货机货道对话框 -->
    <el-dialog :title="title" v-model="open" width="500px" append-to-body>
      <el-form ref="channelRef" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="货道编号" prop="channelCode">
          <el-input v-model="form.channelCode" placeholder="请输入货道编号" />
        </el-form-item>
        <el-form-item label="商品Id" prop="skuId">
          <el-input v-model="form.skuId" placeholder="请输入商品Id" />
        </el-form-item>
        <el-form-item label="售货机Id" prop="vmId">
          <el-input v-model="form.vmId" placeholder="请输入售货机Id" />
        </el-form-item>
        <el-form-item label="售货机软编号" prop="innerCode">
          <el-input v-model="form.innerCode" placeholder="请输入售货机软编号" />
        </el-form-item>
        <el-form-item label="货道最大容量" prop="maxCapacity">
          <el-input v-model="form.maxCapacity" placeholder="请输入货道最大容量" />
        </el-form-item>
        <el-form-item label="货道当前容量" prop="currentCapacity">
          <el-input v-model="form.currentCapacity" placeholder="请输入货道当前容量" />
        </el-form-item>
        <el-form-item label="上次补货时间" prop="lastSupplyTime">
          <el-date-picker clearable
            v-model="form.lastSupplyTime"
            type="date"
            value-format="YYYY-MM-DD"
            placeholder="请选择上次补货时间">
          </el-date-picker>
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script setup name="Channel">
import { listChannel, getChannel, delChannel, addChannel, updateChannel } from "@/api/manage/channel";

const { proxy } = getCurrentInstance();

const channelList = ref([]);
const open = ref(false);
const loading = ref(true);
const showSearch = ref(true);
const ids = ref([]);
const single = ref(true);
const multiple = ref(true);
const total = ref(0);
const title = ref("");

const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    channelCode: null,
    skuId: null,
    vmId: null,
    innerCode: null,
    maxCapacity: null,
    currentCapacity: null,
    lastSupplyTime: null,
    createTime: null,
    updateTime: null
  },
  rules: {
    channelCode: [
      { required: true, message: "货道编号不能为空", trigger: "blur" }
    ],
    vmId: [
      { required: true, message: "售货机Id不能为空", trigger: "blur" }
    ],
    innerCode: [
      { required: true, message: "售货机软编号不能为空", trigger: "blur" }
    ],
    maxCapacity: [
      { required: true, message: "货道最大容量不能为空", trigger: "blur" }
    ],
  }
});

const { queryParams, form, rules } = toRefs(data);

/** 查询售货机货道列表 */
function getList() {
  loading.value = true;
  listChannel(queryParams.value).then(response => {
    channelList.value = response.rows;
    total.value = response.total;
    loading.value = false;
  });
}

// 取消按钮
function cancel() {
  open.value = false;
  reset();
}

// 表单重置
function reset() {
  form.value = {
    id: null,
    channelCode: null,
    skuId: null,
    vmId: null,
    innerCode: null,
    maxCapacity: null,
    currentCapacity: null,
    lastSupplyTime: null,
    createTime: null,
    updateTime: null
  };
  proxy.resetForm("channelRef");
}

/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.pageNum = 1;
  getList();
}

/** 重置按钮操作 */
function resetQuery() {
  proxy.resetForm("queryRef");
  handleQuery();
}

// 多选框选中数据
function handleSelectionChange(selection) {
  ids.value = selection.map(item => item.id);
  single.value = selection.length != 1;
  multiple.value = !selection.length;
}

/** 新增按钮操作 */
function handleAdd() {
  reset();
  open.value = true;
  title.value = "添加售货机货道";
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset();
  const _id = row.id || ids.value
  getChannel(_id).then(response => {
    form.value = response.data;
    open.value = true;
    title.value = "修改售货机货道";
  });
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs["channelRef"].validate(valid => {
    if (valid) {
      if (form.value.id != null) {
        updateChannel(form.value).then(response => {
          proxy.$modal.msgSuccess("修改成功");
          open.value = false;
          getList();
        });
      } else {
        addChannel(form.value).then(response => {
          proxy.$modal.msgSuccess("新增成功");
          open.value = false;
          getList();
        });
      }
    }
  });
}

/** 删除按钮操作 */
function handleDelete(row) {
  const _ids = row.id || ids.value;
  proxy.$modal.confirm('是否确认删除售货机货道编号为"' + _ids + '"的数据项？').then(function() {
    return delChannel(_ids);
  }).then(() => {
    getList();
    proxy.$modal.msgSuccess("删除成功");
  }).catch(() => {});
}

/** 导出按钮操作 */
function handleExport() {
  proxy.download('manage/channel/export', {
    ...queryParams.value
  }, `channel_${new Date().getTime()}.xlsx`)
}

getList();
</script>
