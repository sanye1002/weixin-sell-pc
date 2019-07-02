<template>
  <div>
    <Card>
      <p slot="title">
        <Icon type="md-pricetag"/>
        类目管理
      </p>
      <div>
        <template>
          <Row>
            <Col span="15" class="margin-bottom-10">
              <Button type="info" icon="md-add" @click="showAddCategoryModel">添加类目</Button>
              <Button :disabled="setting.loading" @click="getData" type="success" icon="md-refresh">刷新数据</Button>
              <Button type="primary" icon="ios-download-outline" @click="exportData(1)">导出数据</Button>
              <Button :disabled="selections.length==0 || setting.loading" @click="removeModal = !removeModal" type="error"
                      icon="md-trash">列表删除
              </Button>
            </Col>

          </Row>
          <Table border ref="table" @on-selection-change="selectionChange" :columns="columns"
                 :data="categoryList" :loading="setting.loading"></Table>
        </template>
      </div>
    </Card>

    <Modal v-model="removeModal" width="360">
      <p slot="header" style="color:#f60;text-align:center">
        <Icon type="information-circled"></Icon>
        <span>删除确认</span>
      </p>
      <div style="text-align:center">
        <p>此操作为不可逆操作，是否确认删除？</p>
      </div>
      <div slot="footer">
        <Button type="error" size="large" long :loading="setting.loading" @click="removeCategory">确认删除</Button>
      </div>
    </Modal>
    <Modal v-model="addCategoryModel" :mask-closable="false" :closable="false" title="添加类目" draggable>
      <Form :label-width="80">
        <FormItem label="类目名称">
          <Input v-model="productCategory.categoryName" placeholder="类目名称"></Input>
        </FormItem>
        <FormItem label="类目编号">
          <Input v-model.number="productCategory.categoryType" placeholder="类目编号 只能输入纯数字"></Input>
        </FormItem>
      </Form>
      <div slot="footer">
        <Button type="default" :disabled="setting.loading" @click="addCategoryModel = !addCategoryModel">取消</Button>
        <Button type="primary" :loading="setting.loading" @click="insertCategory">确定</Button>
      </div>
    </Modal>
    <Modal v-model="updateCategoryModel" :mask-closable="false" :closable="false" title="修改类目名称" draggable>
      <Form :label-width="80">
        <FormItem label="类目名称">
          <Input v-model="productCategory.categoryName" placeholder="类目名称"></Input>
        </FormItem>
      </Form>
      <div slot="footer">
        <Button type="default" :disabled="setting.loading" @click="updateCategoryModel = !updateCategoryModel">取消
        </Button>
        <Button type="primary" :loading="setting.loading" @click="updateCategory">修改</Button>
      </div>
    </Modal>
  </div>

</template>

<script>
  import dayjs from 'dayjs'
  import {post} from '@/libs/axios-cfg'

  export default {
    name: "index",
    data() {
      return {
        updateCategoryModel: false,
        addCategoryModel: false,
        removeModal: false,
        removeObject: null,
        productCategory: {
          categoryId: null,
          categoryName: null,
          categoryType: null
        },
        columns: [
          {
            type: 'selection',
            width: 60,
            align: 'center'
          },
          {
            title: 'ID',
            key: 'categoryId',
            sortable: true
          },
          {
            title: '名称',
            key: 'categoryName',
            render: (h, params) => {
              return h('div', [
                h('Icon', {
                  props: {
                    type: 'person'
                  }
                }),
                h('strong', params.row.categoryName)
              ]);
            }
          },
          {
            title: '编号',
            key: 'categoryType',
            sortable: true
          },
          {
            title: '添加时间',
            key: 'createTime',
            render: (h, params) => {
              return h('span', dayjs(params.row.createTime).format('YYYY年MM月DD日 HH:mm:ss'))
            },
            sortable: true
          },
          {
            title: '更新时间',
            key: 'updateTime',
            render: (h, params) => {
              return h('span', dayjs(params.row.updateTime).format('YYYY年MM月DD日 HH:mm:ss'))
            },
            sortable: true
          },
          {
            title: '操作',
            key: 'action',
            width: 150,
            align: 'center',
            render: (h, params) => {
              return h('div', [
                h('Button', {
                  props: {
                    type: 'primary',
                    size: 'small'
                  },
                  style: {
                    marginRight: '5px'
                  },
                  on: {
                    click: () => {
                      this.productCategory={
                        categoryId:params.row.categoryId,
                        categoryName:params.row.categoryName,
                        categoryType:params.row.categoryType
                      }
                      this.updateCategoryModel = true;
                    }
                  }
                }, '编辑'),
                h('Button', {
                  props: {
                    type: 'error',
                    size: 'small'
                  },
                  on: {
                    click: () => {
                      let ids = [];
                      ids.push(params.row.categoryType)
                      this.removeObject = {
                        ids: ids,
                        index: params.index
                      }
                      this.removeModal = true
                    }
                  }
                }, '删除')
              ]);
            }
          }
        ],
        setting: {
          loading: true,
          showBorder: true
        },
        categoryList: [],
        selections: [],
        noticeData: {
          title: "",
          nodesc: ""
        }
      }
    },
    created() {
      this.getData()
    },
    methods: {
      showAddCategoryModel(){
        this.productCategory = {
          categoryId: null,
          categoryName: null,
          categoryType: null
        }
        this.addCategoryModel = true;
      },
      /**
       * @description 批量选择回调
       */
      selectionChange(list) {
        this.selections = list;
        let ids = [];
        this.selections.forEach(e => {
          ids.push(e.categoryType)
        })
        this.removeObject = {
          ids: ids,
          index: null
        }
      },
      /**
       * @description 列表数据删除
       */
      remove(index) {
        if (index != null) {
          this.categoryList.splice(index, 1);
        }
      },
      /**
       * @description 获取类目列表
       */
      async getData() {
        this.setting.loading = true;
        try {
          let res = await post('/system/productCategory/list')
          this.categoryList = res.data;
        } catch (error) {
          this.$throw(error)
        }
        this.setting.loading = false;
      },
      /**
       * @description 删除类目
       */
      async removeCategory() {
        this.removeModal = false;
        if (this.removeObject == null) {
          this.$Message.warning("删除对象为空，无法继续执行！");
          return false;
        }
        this.setting.loading = true;
        try {
          let res = await post('/system/productCategory/delete', this.removeObject.ids);
          if (this.removeObject.ids.length > 1) {
            this.getData();
          }
          this.selections = [];
          this.$Message.destroy()
          this.$Message.success({
            content: res.msg,
            duration: 1.5
          });
          this.remove(this.removeObject.index)
          this.removeObject = null;
        } catch (error) {
          this.$throw(error)
        }
        this.setting.loading = false;
      },
      /**
       * @description 添加类目
       */
      async insertCategory() {

        this.loading = true;
        try {
          if (this.productCategory.categoryName == null) {
            this.$Message.warning('类目名称必须填写');
          }
          if (this.productCategory.categoryType == null) {
            this.$Message.warning('类目编号必须填写');
          }
          let res = await post('/system/productCategory/add', this.productCategory)
          if (res.code == 1) {
            this.$Message.success("类目 " + this.productCategory.categoryName + " 添加成功");
          }
          this.productCategory = {
            categoryId: null,
            categoryName: null,
            categoryType: null
          }
          this.addCategoryModel = false;
          this.getData()
        } catch (error) {
          this.$throw(error)
        }
        this.loading = false;
      },
      /**
       * @description 添加类目
       */
      async updateCategory() {

        this.loading = true;
        try {
          if (this.productCategory.categoryName == null) {
            this.$Message.warning('类目名称必须填写');
          }
          let res = await post('/system/productCategory/update', this.productCategory)
          if (res.code == 1) {
            this.$Message.success("类目 " + this.productCategory.categoryName + " 修改成功");
          }
          this.productCategory = {
            categoryId: null,
            categoryName: null,
            categoryType: null
          }
          this.updateCategoryModel = false;
          this.getData()
        } catch (error) {
          this.$throw(error)
        }
        this.loading = false;
      },
      /**
       * @description 通知
       */
      noticeWarning() {
        this.$Notice.warning({
          title: this.noticeData.title ? '' : '警告提示',
          desc: this.noticeData.nodesc ? '' : '当前操作有误'
        });
      },
      /**
       * @description 导出表格CSV
       */
      exportData(type) {
        if (type === 1) {
          this.$refs.table.exportCsv({
            filename: '类目数据-' + new Date().getTime(),
            columns: this.columns.filter((col, index) => index > 0 && index < this.columns.length - 1),
            data: this.categoryList
          });
        }
      }
    }
  }
</script>

<style scoped>

</style>
