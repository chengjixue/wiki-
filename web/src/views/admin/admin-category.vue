<template>
  <a-layout>
    <a-layout-content style="background:#fff;padding:24px;margin: 0;minHeight:280px" class="ant-layout-content">

      <p>
        <a-form layout="inline" :model="param">
          <a-form-item>
            <a-button type="primary" @click="handleQuery()" size="large">查询</a-button>
          </a-form-item>
          <a-button type="primary" @click="add()" size="large">添加</a-button>
        </a-form>

      </p>

      <a-table
          :columns="columns"
          :row-key="record => record.id"
          :data-source="level1"
          :loading="loading"
          :pagination="false"
      >
        <template #cover="{ text: cover }">
          <img v-if="cover" :src="cover" alt="avatar"/>
        </template>
        <template v-slot:action="{ text, record }">
          <a-space size="small">
            <a-button type="primary" @click="edit(record)">
              编辑
            </a-button>
            <a-popconfirm
                title="删除后不可恢复，确认删除？"
                ok-text="是"
                cancel-text="否 "
                @confirm="handleDelete(record.id)"
            >
              <a-button type="danger">
                删除
              </a-button>
            </a-popconfirm>
          </a-space>
        </template>
      </a-table>

    </a-layout-content>

  </a-layout>

  <a-modal
      title="分类表单"
      v-model:visible="modalVisible"
      :confirm-loading="modalLoading"
      @ok="handleModalOK"
  >
    <a-form :model="category" :label-col="{ span: 6 }">
      <a-form-item label="名称">
        <a-input v-model:value="category.name"/>
      </a-form-item>
      <a-form-item label="父分类">
        <a-input v-model:value="category.parent"/>
        <a-select
            v-model:value="category.parent"
            ref="select"
        >
          <a-select-option :value="0">
            无
          </a-select-option>
          <a-select-option v-for="c in level1" :key="c.id" :value="c.id" :disabled="category.id === c.id">
            {{c.name}}
          </a-select-option>
        </a-select>
      </a-form-item>
      <a-form-item label="顺序">
        <a-input v-model:value="category.sort"/>
      </a-form-item>
    </a-form>
  </a-modal>

</template>

<script lang="ts">
import {defineComponent, onMounted, ref} from 'vue';
import axios from 'axios';
import {message} from "ant-design-vue";
import {Tool} from "@/util/tool";


export default defineComponent({
  name: 'AdminCategory',
  setup() {
    const param = ref();
    param.value = {};
    const categorys = ref();
    const loading = ref(false);
    const columns = [
      {
        title: '名称',
        dataIndex: 'name'
      },
      {
        title: '父分类',
        key: 'parent',
        dataIndex: 'parent',
      },
      {
        title: '顺序',
        dataIndex: 'sort'
      },
      {
        title: 'Action',
        key: 'action',
        slots: {customRender: 'action'}
      }
    ];
    /*
    * 一级分类树，children是二级分类
    * [{
    * id:"",
    * name:"",
    * children:[{
    * id:"",
    * name:"",
    * }]
    * */
    const level1 = ref();
    /**
     * 数据查询
     **/
    const handleQuery = () => {
      loading.value = true;
      //如果不清空数据，则编辑保存重新加载数据后在点编辑会显示未编辑前的数据
      level1.value = [];
      axios.get("/category/all",).then((response) => {
        loading.value = false;
        const data = response.data;
        if (data.success) {
          categorys.value = data.content;
          console.log("原始数据",categorys.value);
          //构建一级分类树
          level1.value=[];
          level1.value = Tool.array2Tree(categorys.value,0);
          console.log("树形结构",level1.value);
        } else {
          message.error(data.message);
        }
      });
    };
    //表单
    const category = ref({});
    const modalVisible = ref(false);
    const modalLoading = ref(false);
    const handleModalOK = () => {
      modalLoading.value = true;
      axios.post("/category/save", category.value).then((response) => {
        modalLoading.value = false;
        const data = response.data; // data => CommonResp
        if (data.success) {
          modalVisible.value = false;
          // 重新加载列表
          handleQuery();
        }
      });
    }
    // 编辑
    const edit = (record: any) => {
      modalVisible.value = true;
      category.value = Tool.copy(record);
    }
    //添加
    const add = () => {
      modalVisible.value = true;
      category.value = {};
    }
    //删除
    const handleDelete = (id: number) => {
      axios.delete("/category/delete/" + id).then((response) => {
        const data = response.data; // data => CommonResp
        if (data.success) {
          // 重新加载列表
          handleQuery();
        }
      });
    }
    onMounted(() => {
      handleQuery();
    });
    return {
      param,
      // categorys,
      level1,
      columns,
      loading,
      edit,
      add,
      category,
      modalVisible,
      modalLoading,
      handleModalOK,
      handleDelete,
      handleQuery
    }
  }
});
</script>

<style scoped>
img {
  width: 50px;
  height: 50px;
}
</style>
