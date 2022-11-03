<script setup>
import { NCard, NDataTable, NButton, NSpace, NModal, NForm, NFormItem, NInput } from 'naive-ui';
import { h, ref, onMounted, computed } from 'vue';
import axios from 'axios';
// import { FontAwesome } from '@vicons/fa';

// import { Icon } from '@vicons/utils';

// export default {
//   components: {
//     Icon,
//     FontAwesome
//   }
// }

const data = ref([])
const currentPage = ref(1)
const isLoading = ref(false)
const totalPage = ref(1)
const perPage = 10

const editId = ref(null)
const formData = ref({
  common_name: '',
  scientific_name: ''
})

const showModal = ref(false)

const deleteLoadingId = ref(null)

const onEditPegawai = (pegawai) => {
  editId.value = pegawai.id
  formData.value.common_name = pegawai.common_name
  formData.value.scientific_name = pegawai.scientific_name
  showModal.value = true
}

const resetForm = () => {
  editId.value = null
  formData.value.common_name = ''
  formData.value.scientific_name = ''
  showModal.value = false
}

const onSubmitForm = () => {
  if (editId.value) {
    editPegawai()
  } else {
    insertPegawai()
  }
}

const onDeletePegawai = (pegawai) => {
  if (!deleteLoadingId.value) {
    deletePegawai(pegawai.id)
  }
}

const insertPegawai = () => {
  axios({
    method: 'post',
    baseURL: 'http://localhost:3000/',
    url: '/posts',
    data: formData.value
  }).then((response) => {
    resetForm()
    getPegawai(1)
  }).catch((error) => {
    console.log(error.message)
  })
}
const editPegawai = () => {
  axios({
    method: 'patch',
    baseURL: 'http://localhost:3000/',
    url: '/posts/' + editId.value,
    data: formData.value
  }).then((response) => {
    resetForm()
    getPegawai(currentPage.value)
  }).catch((error) => {
    console.log(error.message)
  })
}
const deletePegawai = (id) => {
  deleteLoadingId.value = id
  axios({
    method: 'delete',
    baseURL: 'http://localhost:3000/',
    url: '/posts/' + id
  }).then((response) => {
    getPegawai(currentPage.value)
  }).catch((error) => {
    console.log(error.message)
  }).finally(() => {
    deleteLoadingId.value = null
  })
}
const getPegawai = (page) => {
  currentPage.value = page
  isLoading.value = true
  axios({
    method: 'get',
    baseURL: 'http://localhost:3000/',
    url: '/posts',
    params: {
      _page: page,
      _limit: perPage,
      _sort: 'id',
      _order: 'desc'
    }
  }).then((response) => {
    totalPage.value = Math.ceil(response.headers['x-total-count'] / perPage)
    data.value = response.data
  }).catch((error) => {
    console.log(error.message)
  }).finally(() => {
    isLoading.value = false
  })
}

onMounted(() => {
  getPegawai(1)
})

const columns = [
  { title: "No", key: 'id', size: 2 },
  { title: "Common Name", key: 'common_name' },
  { title: "Scientific Name", key: 'scientific_name' },
  {
    title: "Action",
    key: 'action',
    render: (row) => {
      return h(NSpace, {}, [
        h(
          NButton,
          {
            type: 'info',
            ghost: true,
            onClick: () => onEditPegawai(row)
          },
          { default: () => 'Edit' }
        ),
        h(
          NButton,
          {
            type: 'error',
            ghost: true,
            onClick: () => onDeletePegawai(row),
            loading: row.id == deleteLoadingId.value
          },
          { default: () => 'Delete' }
        )
      ])
    }
  },
]

const pagination = computed(() => {
  return {
    pageSize: perPage,
    pageCount: totalPage.value,
    page: currentPage.value
  }
})

const handlePageChange = (page) => {
  getPegawai(page)
}
</script>

<template>
  <NCard title="Pegawai">
    <NButton @click="showModal = true">Add New</NButton>
    <NDataTable jabatan
      remote
      :loading="isLoading"
      :data="data"
      :columns="columns"
      :pagination="pagination"
      @update:page="handlePageChange" />
  </NCard>
  <NModal v-model:show="showModal">
    <NCard style="width: 50%" title="Data Pegawai">
      <NForm
        @submit.prevent="onSubmitForm"
        :model="formData"
        label-placement="left"
        label-width="auto"
        require-mark-placement="right-hanging"
        size="medium">
        <NFormItem label="Common Name" path="common_name">
          <NInput v-model:value="formData.common_name" />
        </NFormItem>
        <NFormItem label="Scientific Name" path="scientific_name">
          <NInput v-model:value="formData.scientific_name" />
        </NFormItem>
        <NFormItem label="&nbsp;">
          <NButton type="info" ghost attr-type="submits">Save</NButton>
        </NFormItem>
      </NForm>
    </NCard>
  </NModal>
</template>
