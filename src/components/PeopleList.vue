<template>
  <el-form
    :model="form"
    status-icon
    ref="formRef"
    label-width="120px"
    class="demo-ruleForm"
  >
    <div class="form-row">
      <el-form-item label="参评人员姓名" prop="name">
        <el-input
          v-model="form.name"
          placeholder="请输入参评人员姓名"
          style="width: 700px"
        ></el-input>
      </el-form-item>
      <el-form-item label="岗位" prop="position">
        <el-select
          v-model="form.position"
          placeholder="请选择岗位"
          style="width: 700px"
        >
          <el-option
            v-for="option in positionOptions"
            :key="option.value"
            :label="option.label"
            :value="option.value"
          ></el-option>
        </el-select>
      </el-form-item>
    </div>
    <div style="width: 400px; margin: 0 auto">
      <el-form-item>
        <el-button type="primary">
          <el-icon><Search /></el-icon>搜索</el-button
        >
        <el-button type="success" @click="showAddDialog">
          <el-icon><Plus /></el-icon> 新增</el-button
        >
      </el-form-item>
    </div>
  </el-form>
  <!-- <el-button type="primary" @click="showAddDialog">新增</el-button> -->
  <div style="width: 1030px; height: 400px; margin: 0 auto">
    <el-table :data="personList" style="width: 100%" border>
      <el-table-column type="index" label="序号" width="180"></el-table-column>
      <el-table-column
        prop="name"
        label="参评人员姓名"
        width="300"
      ></el-table-column>
      <el-table-column prop="position" label="岗位" width="300">
        <template #default="{ row }">
          <el-select
            v-if="row.editing"
            v-model="row.position"
            size="small"
            @change="savePosition(row)"
          >
            <el-option
              v-for="option in positionOptions"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            >
            </el-option>
          </el-select>
          <span v-else>{{ row.position }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="250">
        <template #default="{ row }">
          <el-button type="success" @click="showEditDialog(row)">
            <el-icon><EditPen /></el-icon>编辑</el-button
          >
          <el-popconfirm
            title="确定要删除当前记录吗？"
            :model-value="deleteDialogVisible"
            @update:model-value="dialogVisibleChange"
            confirmButtonText="确定"
            cancelButtonText="取消"
            icon="el-icon-warning"
            iconColor="red"
            @confirm="confirmDelete"
            @cancel="cancelDelete"
          >
            <template #reference>
              <el-button type="danger" @click="showDeleteDialog(row)">
                <el-icon><Delete /></el-icon>删除</el-button
              >
            </template>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>
  </div>
  <!-- 新增人员对话框 -->
  <el-dialog
    title="新增人员"
    :visible="addDialogVisible"
    :model-value="addDialogVisible"
    @update:model-value="dialogVisibleChange"
    width="30%"
  >
    <el-form
      :model="newPerson"
      :rules="rules"
      ref="newPersonFormRef"
      label-width="120px"
      class="demo-ruleForm"
    >
      <el-form-item label="姓名" prop="name">
        <el-input v-model="newPerson.name" placeholder="请输入姓名"></el-input>
      </el-form-item>
      <el-form-item label="岗位" prop="position">
        <el-select v-model="newPerson.position" placeholder="请选择岗位">
          <el-option
            v-for="option in positionOptions"
            :key="option.value"
            :label="option.label"
            :value="option.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button @click="cancelAdd">取消</el-button>
      <el-button type="primary" @click="submitAdd">提交</el-button>
    </template>
  </el-dialog>

  <!-- 编辑岗位对话框 -->
  <el-dialog
    title="编辑岗位"
    :visible="editDialogVisible"
    :model-value="editDialogVisible"
    @update:model-value="dialogVisibleChange"
    width="30%"
  >
    <el-form
      :model="currentEditingPerson"
      :rules="rules"
      ref="editPersonFormRef"
      label-width="120px"
      class="demo-ruleForm"
    >
      <el-form-item label="岗位" prop="position">
        <el-select
          v-model="currentEditingPerson.position"
          placeholder="请选择岗位"
        >
          <el-option
            v-for="option in positionOptions"
            :key="option.value"
            :label="option.label"
            :value="option.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button @click="cancelEdit">取消</el-button>
      <el-button type="primary" @click="submitEdit">提交</el-button>
    </template>
  </el-dialog>
  <!-- 删除确认对话框 -->
  <!-- <el-dialog
    title="确认删除"
    :visible="deleteDialogVisible"
    :model-value="deleteDialogVisible"
    @update:model-value="dialogVisibleChange"
    width="30%"
  >
    <span>确定要删除当前记录吗？</span>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="cancelDelete">取消</el-button>
        <el-button type="primary" @click="confirmDelete">确定</el-button>
      </span>
    </template>
  </el-dialog> -->
  <!-- <el-popconfirm
    title="确定要删除当前记录吗？"
    :visible="deleteDialogVisible"
    :model-value="deleteDialogVisible"
    @update:model-value="dialogVisibleChange"
    confirmButtonText="确定"
    cancelButtonText="取消"
    icon="el-icon-warning"
    iconColor="red"
    @confirm="confirmDelete"
    @cancel="cancelDelete"
  >
  </el-popconfirm> -->
</template>

<script>
export default {
  data() {
    return {
      form: {
        name: "",
        position: "",
      },
      personList: [
        { id: 1, name: "张三", position: "经理", editing: false },
        { id: 2, name: "李四", position: "工程师", editing: false },
        { id: 3, name: "王五", position: "设计师", editing: false },
      ],
      newPerson: {
        name: "",
        position: "",
      },
      currentEditingPerson: null,
      positionOptions: [
        { value: "经理", label: "经理" },
        { value: "工程师", label: "工程师" },
        { value: "设计师", label: "设计师" },
      ],
      rules: {
        name: [{ required: true, message: "请输入姓名", trigger: "blur" }],
        position: [
          { required: true, message: "请选择岗位", trigger: "change" },
        ],
      },
      addDialogVisible: false, // 控制新增对话框的显示
      deleteDialogVisible: false, // 控制删除对话框的显示
      editDialogVisible: false,
      currentDeletingRow: null,
    };
  },
  mounted() {
    this.newPersonFormRef = this.$refs.newPersonFormRef;
    this.editPersonFormRef = this.$refs.editPersonFormRef;
  },
  methods: {
    editPosition(row) {
      row.editing = true;
    },
    savePosition(row) {
      row.editing = false;
    },
    cancelEdit() {
      this.editDialogVisible = false;
      // this.currentEditingPerson = null;
    },
    showEditDialog(row) {
      this.editDialogVisible = true;
      this.currentEditingPerson = { ...row };
    },
    showAddDialog() {
      this.addDialogVisible = true; // 显示新增人员对话框
    },
    submitAdd() {
      //if (!this.newPersonFormRef) return;
      // console.log(this.personList);

      const newRow = {
        id: this.personList.length + 1,
        name: this.newPerson.name,
        position: this.newPerson.position,
        editing: false,
      };
      this.personList.push(newRow);
      console.log(this.personList);

      this.addDialogVisible = false; // 提交后关闭对话框
      this.newPerson = { name: "", position: "" }; // 清空表单
    },
    submitEdit() {
      // if (!this.editPersonFormRef) return;

      const index = this.personList.findIndex(
        (person) => person.id === this.currentEditingPerson.id
      );
      if (index !== -1) {
        this.personList.splice(index, 1, this.currentEditingPerson);
      }
      this.editDialogVisible = false;
      //this.currentEditingPerson = null;
    },
    cancelAdd() {
      this.addDialogVisible = false; // 取消新增时关闭对话框
      this.newPerson = { name: "", position: "" }; // 清空表单
    },
    showDeleteDialog(row) {
      this.currentDeletingRow = row;
      this.deleteDialogVisible = true;
    },
    confirmDelete() {
      const index = this.personList.findIndex(
        (person) => person.id === this.currentDeletingRow.id
      );
      if (index !== -1) {
        this.personList.splice(index, 1);
      }
      this.deleteDialogVisible = false;
      this.currentDeletingRow = null;
    },
    cancelDelete() {
      this.deleteDialogVisible = false;
      this.currentDeletingRow = null;
    },
    dialogVisibleChange(value) {
      this.addDialogVisible = value;
      this.editDialogVisible = value;
      this.deleteDialogVisible = value;
    },
  },
};
</script>

<style scoped>
.demo-ruleForm {
  margin-top: 20px;
}
.form-row {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
