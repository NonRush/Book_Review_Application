<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryRef" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="Name" prop="bookName">
        <el-input
          v-model="queryParams.bookName"
          placeholder="Enter the book's name"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="Author" prop="author">
        <el-input
          v-model="queryParams.author"
          placeholder="Enter the author"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="Region" prop="regionId">
        <el-input
          v-model="queryParams.regionId"
          placeholder="Select the region"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="Category" prop="categoryId">
        <el-input
          v-model="queryParams.categoryId"
          placeholder="Select the category"
          clearable
          @keyup.enter="handleQuery"
        />
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
          v-hasPermi="['manager:book:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="Edit"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['manager:book:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="Delete"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['manager:book:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="Download"
          @click="handleExport"
          v-hasPermi="['manager:book:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar v-model:showSearch="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="bookList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="ID" type="index" align="center" prop="id" />
      <el-table-column label="Book name" align="center" prop="bookName" />
      <el-table-column label="Cover" align="center" prop="cover" width="100">
        <template #default="scope">
          <image-preview :src="scope.row.cover" :width="50" :height="50"/>
        </template>
      </el-table-column>
      <el-table-column label="Author" align="center" prop="author" />
      <el-table-column label="Publisher" align="center" prop="publisher" />
      <el-table-column label="Publication date" align="center" prop="publishDate" width="180">
        <template #default="scope">
          <span>{{ parseTime(scope.row.publishDate, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Price" align="center" prop="price" />
      <el-table-column label="Quantity" align="center" prop="quantity" />
      <el-table-column label="Region" align="center" prop="regionId" />
      <el-table-column label="Category" align="center" prop="categoryId" />
      <el-table-column label="Description" align="center" prop="remark" />
      <el-table-column label="Action" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <el-button link type="primary" icon="Edit" @click="handleUpdate(scope.row)" v-hasPermi="['manager:book:edit']">修改</el-button>
          <el-button link type="primary" icon="Delete" @click="handleDelete(scope.row)" v-hasPermi="['manager:book:remove']">删除</el-button>
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

    <!-- 添加或修改书籍管理对话框 -->
    <el-dialog :title="title" v-model="open" width="500px" append-to-body>
      <el-form ref="bookRef" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="Name" prop="bookName">
          <el-input v-model="form.bookName" placeholder="Enter the book's name" />
        </el-form-item>
        <el-form-item label="Cover" prop="cover">
          <image-upload v-model="form.cover"/>
        </el-form-item>
        <el-form-item label="Author" prop="author">
          <el-input v-model="form.author" placeholder="Enter the author" />
        </el-form-item>
        <el-form-item label="Publisher" prop="publisher">
          <el-input v-model="form.publisher" placeholder="Enter the Publisher" />
        </el-form-item>
        <el-form-item label="Publication Date" prop="publishDate">
          <el-date-picker clearable
            v-model="form.publishDate"
            type="date"
            value-format="YYYY-MM-DD"
            placeholder="Select date">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="Price" prop="price">
          <el-input v-model="form.price" placeholder="Enter price" />
        </el-form-item>
        <el-form-item label="Quantity" prop="quantity">
          <el-input v-model="form.quantity" placeholder="Enter quantity" />
        </el-form-item>
        <el-form-item label="Region" prop="regionId">
          <el-input v-model="form.regionId" placeholder="Enter region" />
        </el-form-item>
        <el-form-item label="Category" prop="categoryId">
          <el-input v-model="form.categoryId" placeholder="Enter category" />
        </el-form-item>
        <el-form-item label="Description" prop="remark">
          <el-input v-model="form.remark" type="textarea" placeholder="Enter here" />
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">Submit</el-button>
          <el-button @click="cancel">Cancel</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script setup name="Book">
import { listBook, getBook, delBook, addBook, updateBook } from "@/api/manager/book";

const { proxy } = getCurrentInstance();

const bookList = ref([]);
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
    bookName: null,
    author: null,
    regionId: null,
    categoryId: null,
  },
  rules: {
    bookName: [
      { required: true, message: "书籍名称不能为空", trigger: "blur" }
    ],
    author: [
      { required: true, message: "作者不能为空", trigger: "blur" }
    ],
    publisher: [
      { required: true, message: "出版社不能为空", trigger: "blur" }
    ],
    publishDate: [
      { required: true, message: "出版日期不能为空", trigger: "blur" }
    ],
    quantity: [
      { required: true, message: "数量不能为空", trigger: "blur" }
    ],
  }
});

const { queryParams, form, rules } = toRefs(data);

/** 查询书籍管理列表 */
function getList() {
  loading.value = true;
  listBook(queryParams.value).then(response => {
    bookList.value = response.rows;
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
    bookName: null,
    cover: null,
    author: null,
    publisher: null,
    publishDate: null,
    price: null,
    quantity: null,
    regionId: null,
    categoryId: null,
    createTime: null,
    dateTime: null,
    createBy: null,
    updateBy: null,
    remark: null
  };
  proxy.resetForm("bookRef");
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
  title.value = "添加书籍管理";
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset();
  const _id = row.id || ids.value
  getBook(_id).then(response => {
    form.value = response.data;
    open.value = true;
    title.value = "修改书籍管理";
  });
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs["bookRef"].validate(valid => {
    if (valid) {
      if (form.value.id != null) {
        updateBook(form.value).then(response => {
          proxy.$modal.msgSuccess("修改成功");
          open.value = false;
          getList();
        });
      } else {
        addBook(form.value).then(response => {
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
  proxy.$modal.confirm('是否确认删除书籍管理编号为"' + _ids + '"的数据项？').then(function() {
    return delBook(_ids);
  }).then(() => {
    getList();
    proxy.$modal.msgSuccess("删除成功");
  }).catch(() => {});
}

/** 导出按钮操作 */
function handleExport() {
  proxy.download('manager/book/export', {
    ...queryParams.value
  }, `book_${new Date().getTime()}.xlsx`)
}

getList();
</script>
