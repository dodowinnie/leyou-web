<template>
  <div>
    <v-card>
      <!--卡片头部-->
      <v-card-title>
        <v-btn color="primary" @click="addBrand">新增品牌</v-btn>
        <!--空间隔离组件-->
        <v-spacer/>

        <!--搜索框，与search属性关联-->
        <v-text-field label="输入关键字搜索" v-model="search" append-icon="search" hide-details/>
      </v-card-title>
      <!--分割线-->
      <v-divider/>
      <!--卡片中部-->
      <v-data-table
        :headers="headers"
        :items="brands"
        :search="search"
        :pagination.sync="pagination"
        :total-items="totalBrands"
        :loading="loading"
        class="elevation-1"
      >
        <template slot="items" slot-scope="props">
          <td>{{ props.item.id }}</td>
          <td class="text-xs-center">{{ props.item.name }}</td>
          <td class="text-xs-center">
            <img v-if="props.item.image" :src="props.item.image" width="130" height="40">
            <span v-else>无</span></td>
          <td class="text-xs-center">{{ props.item.letter }}</td>
          <td class="justify-center layout">
            <v-btn color="info">编辑</v-btn>
            <v-btn color="warning">删除</v-btn>
          </td>
        </template>
      </v-data-table>

    </v-card>
    <v-dialog max-width="500" v-model="show" persistent>
      <v-card>
        <!--对话框的标题-->
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>新增品牌</v-toolbar-title>
          <v-spacer/>
          <v-btn icon @click="closeWindow"><v-icon>close</v-icon></v-btn>
        </v-toolbar>
        <!--对话框的内容，表单-->
        <v-card-text class="px-5">
          <my-form @close-window="closeWindow"></my-form>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>

</template>

<script>
  // 导入自定义组件
  import MyBrandForm from './MyBrandForm'
  export default {
    name: "my-brand",
    components: {
      "my-form":MyBrandForm
    },

    data() {
      return {
        search: '', // 搜索过滤字段
        totalBrands: 0, // 总条数
        brands: [], // 当前页品牌数据
        loading: true, // 是否在加载中
        pagination: {

        }, // 分页信息
        headers: [ // 头信息
          {text: 'id', align: 'center', value: 'id'},
          {text: '名称', align: 'center', sortable: false, value: 'name'},
          {text: 'LOGO', align: 'center', sortable: false, value: 'image'},
          {text: '首字母', align: 'center', value: 'letter', sortable: true,},
          {text: '操作', align: 'center', value: 'id', sortable: false,}
        ],
        show:false
      }
    },
    methods: {
      getDataFromServer() { // 从服务的加载数据的方法
        this.$http.get("/item/brand/page",{
          params:{
            key: this.search,
            page: this.pagination.page,
            rows: this.pagination.rowsPerPage,
            sortBy: this.pagination.sortBy,
            desc: this.pagination.descending
          }
        }).then(resp =>{
          this.brands = resp.data.items;
          this.totalBrands = resp.data.total;
          // 完成赋值后，把加载状态赋值为false
          this.loading = false;
        })

      },

      addBrand(){
        this.show = true
      },
      closeWindow(){
        // 关闭窗口
        this.show = false;
        // 重新加载数据
        this.getDataFromServer();
      }


    },

    watch:{
      pagination:{ // 监视pagination属性的变化
        deep:true, // deep为true，会监视pagination的属性及属性中的对象属性变化
        handler(){
          this.getDataFromServer();
        }
      },

      search:{ // 监视搜索字段
        handler(){
          this.getDataFromServer();
        }
      }
    },

    mounted() { // 渲染后执行
      this.getDataFromServer()
    }
  }
</script>

<style scoped>

</style>
