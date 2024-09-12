<template>
  <el-table :data="personList" style="width: 100%" border>
    <el-table-column type="index" label="序号" width="80"></el-table-column>
    <el-table-column
      prop="name"
      label="参评人员姓名"
      width="200"
    ></el-table-column>
    <el-table-column prop="position" label="岗位" width="200">
      <span>{{ row.position }}</span>
    </el-table-column>
    <el-table-column label="操作" width="150">
      <template #default="{ row }">
        <el-button type="text" @click="showEditDialog(row)">编辑</el-button>
        <el-button type="text" @click="showDeleteDialog(row)">删除</el-button>
      </template>
    </el-table-column>
  </el-table>

  <el-button type="primary" @click="showAddDialog">新增</el-button>

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
      <el-form-item label="姓名">
        <el-input v-model="currentEditingPerson.name" disabled></el-input>
      </el-form-item>
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
  <el-dialog
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
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      personList: [
        { id: 1, name: "张三", position: "经理", editing: false },
        { id: 2, name: "李四", position: "工程师", editing: false },
        { id: 3, name: "王五", position: "设计师", editing: false },
      ],
      newPerson: {
        name: "",
        position: "",
      },
      currentEditingPerson: null, // 当前编辑的人员
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
      editDialogVisible: false, // 控制编辑对话框的显示
      deleteDialogVisible: false, // 控制删除对话框的显示
      currentDeletingRow: null,
    };
  },
  mounted() {
    this.newPersonFormRef = this.$refs.newPersonFormRef;
    this.editPersonFormRef = this.$refs.editPersonFormRef;
  },
  methods: {
    showAddDialog() {
      this.addDialogVisible = true;
    },
    submitAdd() {
      if (!this.newPersonFormRef) return;

      this.newPersonFormRef.validate((valid) => {
        if (valid) {
          const newRow = {
            id: this.personList.length + 1,
            name: this.newPerson.name,
            position: this.newPerson.position,
            editing: false,
          };
          this.personList.push(newRow);
          this.addDialogVisible = false;
          this.newPerson = { name: "", position: "" };
        } else {
          console.log("表单验证失败");
          return false;
        }
      });
    },
    cancelAdd() {
      this.addDialogVisible = false;
      this.newPerson = { name: "", position: "" };
    },
    showEditDialog(row) {
      this.currentEditingPerson = { ...row };
      this.editDialogVisible = true;
    },
    submitEdit() {
      if (!this.editPersonFormRef) return;

      this.editPersonFormRef.validate((valid) => {
        if (valid) {
          const index = this.personList.findIndex(
            (person) => person.id === this.currentEditingPerson.id
          );
          if (index !== -1) {
            this.personList.splice(index, 1, this.currentEditingPerson);
          }
          this.editDialogVisible = false;
          this.currentEditingPerson = null;
        } else {
          console.log("表单验证失败");
          return false;
        }
      });
    },
    cancelEdit() {
      this.editDialogVisible = false;
      this.currentEditingPerson = null;
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
</style>
