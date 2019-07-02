<template>
  <div>
    <Card>
      <p slot="title">
        <Icon type="md-pricetag"/>
        商品管理
      </p>
      <div>
        <template>
          <Row>
            <Col span="15">
              <Button type="info" icon="md-add" @click="openAddModel">添加商品</Button>
              <Button :disabled="setting.loading" type="success" icon="md-refresh" @click="getData">刷新数据</Button>
              <Button type="primary" icon="ios-download-outline" @click="exportData(1)">导出数据</Button>
              <Button :disabled="selections.length === 0 || setting.loading" type="error"
                      icon="md-trash" @click="removeModal=!removeModal">列表删除
              </Button>
            </Col>
            <Col span="9">
              <Input placeholder="请输入您想要搜索的内容..." class="margin-bottom-10">
                <Button slot="append" icon="ios-search"></Button>
              </Input>
            </Col>
          </Row>
          <Table border ref="table" @on-selection-change="selectionChange" :columns="columns"
                 :data="productData.records" :loading="setting.loading" class="margin-bottom-10"></Table>
          <Page :total="productData.total" class="tr" @on-change="pageChange" :current.sync="productData.current"
                :page-size="productData.size"
                @on-page-size-change="pageSizeChange" show-elevator show-sizer></Page>
        </template>
      </div>
    </Card>
    <Modal v-model="changeProductStatusModel" width="360">
      <p slot="header" style="color:#f60;text-align:center">
        <Icon type="information-circled"></Icon>
        <span>状态修改确认</span>
      </p>
      <div style="text-align:center">
        <p v-if="changeProductStatusData!=null">确定 <b>{{changeProductStatusData.status === 0? '下架':'上架'}}</b> 此商品</p>
      </div>
      <div slot="footer">
        <Button type="error" size="large" long :loading="setting.loading" @click="changeStatus">确认</Button>
      </div>
    </Modal>
    <Modal v-model="removeModal" width="360">
      <p slot="header" style="color:#f60;text-align:center">
        <Icon type="information-circled"></Icon>
        <span>删除确认</span>
      </p>
      <div style="text-align:center">
        <p>此操作为不可逆操作，是否确认删除？</p>
      </div>
      <div slot="footer">
        <Button type="error" size="large" long :loading="setting.loading" @click="removeProduct">确认删除</Button>
      </div>
    </Modal>
    <ProductModel v-if="addModel" :product-info="productInfo" :update-status="updateStatus"
                  @cancel="onModalCancel"></ProductModel>
    <Modal title="商品小图查看" v-model="visible">
      <div style="width: 100%;text-align: center">
        <img :src="visibleURL" style="height: 300px">
      </div>

    </Modal>
  </div>
</template>

<script>
  import dayjs from 'dayjs'
  import {post} from '@/libs/axios-cfg'
  import ProductModel from './components/addProduct.vue'

  export default {
    name: "index",
    data() {
      return {
        productInfo: {
          productId: "1082597742354153473",
          productName: "好吃的老雷2",
          productPrice: 9.9,
          productStock: 100,
          salesVolume: 0,
          productDescription: "好吃的老雷2",
          productIcon: "https://p1.pstatp.com/list/190x124/pgc-image/c5b1dfe45f5f4e0789d5c93572dbf728",
          productStatus: 0,
          categoryType: 200,
          unlimitedStock: false,
          productStandardStatus: false,
          productStandard: {
            productPriceAll: false,
            unlimitedStock: false,
            standardList: [
              {
                name: '',
                productPrice: null,
                originalPrice: null,
                stockUnlimited: false,
                productStock: 0
              }
            ]
          }
        },
        setting: {
          loading: true,
          showBorder: true
        },
        columns: [
          {
            type: 'selection',
            width: 60,
            align: 'center'
          },
          {
            title: '图片',
            key: 'productIcon',
            width: 80,
            render: (h, params) => {
              return h('div', [
                h('Avatar', {
                  props: {
                    src: params.row.productIcon
                  }
                })
              ])
            }
          },
          {
            title: '名称',
            key: 'productName',
            render: (h, params) => {
              return h('div', [
                h('Icon', {
                  props: {
                    type: 'person'
                  }
                }),
                h('strong', params.row.productName)
              ]);
            }
          },
          {
            title: '类目编号',
            key: 'categoryType'
          },
          {
            title: '单价',
            key: 'productPrice',
            sortable: true
          },
          {
            title: '库存',
            key: 'productStock',
            sortable: true
          },
          {
            title: '状态',
            key: 'productStatus',
            sortable: true,
            width: 80,
            render: (h, params) => {
              return h('div', [
                h('Button', {
                  props: {
                    type: params.row.productStatus === 0 ? 'primary' : 'error',
                    size: 'small',
                    ghost: true
                  },
                  on: {
                    click: () => {
                      this.changeProductStatusData = {
                        status: params.row.productStatus,
                        index: params.index,
                        productId: params.row.productId
                      }
                      this.changeProductStatusModel = true;
                    }
                  }
                }, params.row.productStatus === 0 ? '正常' : '下架')
              ]);
            }
          },
          {
            title: '销量',
            key: 'salesVolume',
            render: (h, params) => {
              return h('strong', params.row.salesVolume)
            },
            sortable: true
          },
          {
            title: '更新时间',
            key: 'updateTime',
            width: 200,
            render: (h, params) => {
              return h('span', dayjs(params.row.updateTime).format('YYYY年MM月DD日 HH:mm:ss'))
            },
            sortable: true
          },
          {
            title: '操作',
            key: 'action',
            width: 220,
            align: 'center',
            render: (h, params) => {
              return h('div', [
                h('Button', {
                  props: {
                    type: 'info',
                    size: 'small'
                  },
                  style: {
                    marginRight: '5px'
                  },
                  on: {
                    click: () => {
                      this.visible = true;
                      this.visibleURL = params.row.productIcon;
                    }
                  }
                }, '图片'),
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
                      this.productInfo = params.row;
                      console.log(params.row)
                      this.addModel = true;
                      this.updateStatus = true;
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
                      ids.push(params.row.productId)
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
        productData: {},
        dataFilter: {
          page: 1,
          pageSize: 10,
          categoryType: null
        },
        removeModal: false,
        removeObject: null,
        selections: [],
        changeProductStatusModel: false,
        changeProductStatusData: null,
        addModel: false,
        updateStatus: false,
        visible: false,
        visibleURL: ''
      }
    },
    components: {
      ProductModel
    },
    created() {
      this.getData();
    },
    methods: {
      /**
       * @description 关闭模态窗口
       * @param reload 是否重新加载数据
       */
      onModalCancel(type, reload = false) {
        console.log(type)
        this.addModel = false;
        if (reload) {
          this.getData();
        }
      },
      /**
       * @description 清除数据
       */
      cleanInfo() {
        this.productInfo = {
          productId: null,
          productName: null,
          productPrice: null,
          productStock: null,
          salesVolume: null,
          productDescription: null,
          productIcon: null,
          productStatus: null,
          categoryType: null,
          originalPrice: null,
          unlimitedStock: false,
          standardStatus: false,
          productStandard: {
            productPriceAll: false,
            unlimitedStock: false,
            standardList: [
              {
                name: '',
                productPrice: null,
                originalPrice: null,
                stockUnlimited: false,
                productStock: 0
              }
            ]
          }
        }
      },
      /**
       * @description 列表数据删除
       */
      remove(index) {
        if (index != null) {
          this.productData.records.splice(index, 1);
        }
      },
      /**
       * @description 批量选择回调
       */
      selectionChange(list) {
        this.selections = list;
        let ids = [];
        this.selections.forEach(e => {
          ids.push(e.productId)
        })
        this.removeObject = {
          ids: ids,
          index: null
        }
      },
      /**
       * @description 分页更换事件回调
       */
      pageChange(p) {
        this.dataFilter.page = p;
        this.getData();
      },
      /**
       * @description 分页每页显示数量改变事件回调
       */
      pageSizeChange(p) {
        this.dataFilter.pageSize = p;
        this.getData();
      },
      /**
       * @description 获取商品列表
       */
      async getData() {
        this.setting.loading = true;
        try {
          let res = await post('/system/productInfo/select/page/categoryType', {
            categoryType: this.categoryType,
            page: this.dataFilter.page,
            pageSize: this.dataFilter.pageSize
          })
          this.productData = res.data;
        } catch (error) {
          this.$throw(error)
        }
        this.setting.loading = false;
      },
      /**
       * @description post商品上架
       */
      changeStatus() {
        this.setting.loading = true;
        this.changeProductStatusModel = false;
        if (this.changeProductStatusData == null) {
          this.$Message.warning("修改商品为空，无法继续执行！");
          return false;
        }

        if (this.changeProductStatusData.status === 0) {
          this.offSale()
        } else {
          this.onSale()
        }
      },
      /**
       * @description post商品上架
       */
      async removeProduct() {
        this.removeModal = false;
        if (this.removeObject == null) {
          this.$Message.warning("删除对象为空，无法继续执行！");
          return false;
        }
        this.setting.loading = true;
        try {
          let res = await post('/system/productInfo/delete', this.removeObject.ids);
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
       * @description post商品上架
       */
      async onSale() {
        this.changeProductStatusModel = false;
        try {
          let res = await post('/system/productInfo/onSale', {
            productId: this.changeProductStatusData.productId
          })
          if (res.code === 1) {
            this.$set(this.productData.records[this.changeProductStatusData.index], 'productStatus', 0)
          }
        } catch (error) {
          this.$throw(error)
        }
        this.setting.loading = false;
      },
      /**
       * @description post商品下架
       */
      async offSale() {
        this.changeProductStatusModel = false;
        try {
          let res = await post('/system/productInfo/offSale', {
            productId: this.changeProductStatusData.productId
          });
          if (res.code === 1) {
            this.$set(this.productData.records[this.changeProductStatusData.index], 'productStatus', 1)
          }
        } catch (error) {
          this.$throw(error)
        }
        this.setting.loading = false;
      },
      /**
       * @description 打开添加窗口
       */
      openAddModel() {
        this.cleanInfo();
        this.addModel = true;
        this.updateStatus = false;
      },
      /**
       * @description 导出表格CSV
       */
      exportData(type) {
        if (type === 1) {
          this.$refs.table.exportCsv({
            filename: '商品数据-' + new Date().getTime(),
            columns: this.columns.filter((col, index) => index > 0 && index < this.columns.length - 1),
            data: this.productData.records
          });
        }
      }
    }
  }
</script>

<style scoped>

</style>
