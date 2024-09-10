<template>
  <div class="flex flex-col gap-6">
    <section>
      <h2>集合 - Collection</h2>

      <!-- 搜索框和创建按钮 -->
      <div class="header-container">
        <el-button @click="openDialog" type="primary">创建集合</el-button>
        <span style="display: inline-flex">
          <el-input v-model="searchQuery" placeholder="请输入集合名称搜索" clearable style="margin-right: 5px" />
          <el-button @click="performSearch" type="primary">搜索</el-button>
        </span>
      </div>

      <!-- 集合展示表格 -->
      <el-table :data="collections" style="width: 100%">
        <el-table-column prop="id" label="ID" />
        <el-table-column prop="name" label="集合名称" />
        <el-table-column prop="vest_database" label="归属数据库" />
        <el-table-column prop="vest_tenant" label="归属学院 (英)" />
      </el-table>

      <!-- 弹窗：创建集合 -->
      <el-dialog
        title="创建新集合"
        v-model="isDialogVisible"
        width="500"
        @close="resetDialogForm"
        style="z-index: 9999">
        <el-form :model="newCollection">
          <el-form-item label="集合名称" required>
            <el-input v-model="newCollection.name" />
          </el-form-item>
          <el-form-item label="归属数据库" required>
            <el-input v-model="newCollection.database_name" />
          </el-form-item>
          <el-form-item label="归属学院 (英)" required>
            <el-input v-model="newCollection.tenant_name" />
          </el-form-item>
          <el-form-item label="额外信息" required>
            <el-input v-model="newCollection.metadata" />
          </el-form-item>
        </el-form>

        <template #footer>
          <el-button @click="isDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="createNewCollection">创建</el-button>
        </template>
      </el-dialog>
    </section>
    <section>
      <el-card>
        <template #header>
          <div class="card-header">
            <h2>上传文档</h2>
          </div>
        </template>
        <el-cascader class="w-full" v-model="collectionName" :options="options" />
        <UploadDocument :collectionName="collectionName" />
      </el-card>
    </section>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { getCollections, createCollection, getCollectionByName } from '@/apis/collection'
import UploadDocument from '@/components/rag/uploadDocument.vue'
import { OptionsType } from '#/ui'
export interface CollectionType {
  id: string
  name: string
  database_name: string
  tenant_name: string
}

onMounted(async () => {
  await fetchCollections()
})
const collectionName = ref('')
const options = ref<OptionsType[]>([])
const collections = ref<CollectionType[]>([])
const searchQuery = ref('')
const newCollection = ref({
  name: '',
  tenant_name: '',
  database_name: '',
  metadata: '',
})
const isDialogVisible = ref(false)

async function fetchCollections() {
  const response = await getCollections()
  const data: CollectionType[] = await response.json()
  collections.value = data // 确保数据结构与Collection匹配
  const names: OptionsType[] = []
  data.forEach((item) => {
    const { name } = item
    const optionItem: OptionsType = {
      label: name,
      value: name,
    }
    names.push(optionItem)
  })
  options.value = names
}
// 搜索集合 （input直接搜索）
async function searchCollections() {
  if (searchQuery.value) {
    const response = await getCollectionByName(searchQuery.value)
    const data = await response.json()
    collections.value = [data] // 确保返回的是Collection类型
  } else {
    fetchCollections()
  }
}

// 搜索集合（点击搜索按钮搜索）
async function performSearch() {
  if (searchQuery.value) {
    const response = await getCollectionByName(searchQuery.value)
    const data = await response.json()
    // getCollectionByName返回单个对象，放入数组
    collections.value = [data]
  } else {
    fetchCollections() // 如果没有搜索词，重新获取所有集合
  }
}

// 打开弹窗的函数
function openDialog() {
  console.log('Dialog Opened')
  isDialogVisible.value = true
}

// 重置弹窗表单的函数
function resetDialogForm() {
  newCollection.value = {
    name: '',
    tenant_name: '',
    database_name: '',
    metadata: '',
  }
}

// 创建新集合的函数
async function createNewCollection() {
  if (!newCollection.value.name || !newCollection.value.tenant_name || !newCollection.value.database_name) {
    return alert('请填写完整的信息')
  }
  const newData = newCollection.value
  const response = await createCollection({ ...newData, metadata: { descript: newData.metadata } })
  if (response.ok) {
    alert('集合创建成功')
    newCollection.value = { name: '', tenant_name: '', database_name: '', metadata: '' } // 清空表单
    fetchCollections() // 重新获取集合列表
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
