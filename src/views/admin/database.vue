<template>
  <div>
    <section>
      <h2>数据库-Database</h2>

      <!-- 搜索框和创建按钮 -->
      <div class="header-container">
        <el-button @click="isDialogVisible = true" type="primary">创建数据库</el-button>
        <span style="display: inline-flex;">
          <el-input v-model="searchQuery" placeholder="请输入数据库名称搜索" clearable style="margin-right:5px;" />
          <el-button @click="performSearch" type="primary">搜索</el-button>
        </span>
      </div>

      <el-table :data="databases" style="width: 100%">
        <el-table-column prop="id" label="ID" />
        <el-table-column prop="name" label="数据库名称" />
        <el-table-column prop="tenant_name" label="归属学院 (英)" />
      </el-table>

      <!-- 弹窗：创建数据库 -->
      <el-dialog title="创建新数据库" v-model="isDialogVisible" width="500" @close="resetDialogForm" style="z-index: 9999;">
        <el-form :model="newDatabase">
          <el-form-item label="归属数据库" required>
            <el-input v-model="newDatabase.database_name" />
          </el-form-item>
          <el-form-item label="归属学院 (英)" required>
            <el-input v-model="newDatabase.tenant_name" />
          </el-form-item>
        </el-form>

        <template #footer>
          <el-button @click="isDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="createNewDatabase">创建</el-button>
        </template>
      </el-dialog>
    </section>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { getDatabases, createDatabase, getDatabaseByName } from '@/apis/database'

export interface DatabaseType {
  id: string;
  database_name: string;
  tenant_name: string;
}

const databases = ref<DatabaseType[]>([])
const searchQuery = ref('')
const newDatabase = ref({
  tenant_name: '',
  database_name: ''
})
const isDialogVisible = ref(false)

// 页面加载时获取所有数据库
onMounted(async () => {
  await fetchDatabases()
})

// // 获取所有数据库的函数
async function fetchDatabases() {
  const response = await getDatabases()
  const data = await response.json()
  databases.value = data // 确保数据结构与Collection匹配
}


// 搜索数据库（点击搜索按钮搜索）
async function performSearch() {
  if (searchQuery.value) {
    const response = await getDatabaseByName(searchQuery.value)
    const data = await response.json()
    // getDatabaseByName返回单个对象，放入数组
    databases.value = [data];
  } else {
    fetchDatabases(); // 如果没有搜索词，重新获取所有集合
  }
}

// 重置弹窗表单的函数
function resetDialogForm() {
  newDatabase.value = {
    tenant_name: '',
    database_name: ''
  }
}

// 创建新集合的函数
async function createNewDatabase() {
  if (!newDatabase.value.tenant_name || !newDatabase.value.database_name) {
    return alert('请填写完整的信息')
  }

  const newData = newDatabase.value

  const response = await createDatabase({
    name: newData.database_name,
    tenant: newData.tenant_name
  })
  if (response.ok) {
    alert('集合创建成功')
    newDatabase.value = {tenant_name: '', database_name: '' } // 清空表单
    fetchDatabases() // 重新获取集合列表
    isDialogVisible.value = false // 关闭弹窗
  } else {
    alert('创建失败')
  }
}

</script>

<style scoped>
    .header-container {
      display: flex;
      justify-content: space-between;
      margin-bottom: 10px;
      flex-wrap: nowrap;
      margin-top: 5px;
    }
</style>
