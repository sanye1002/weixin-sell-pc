<template>
  <div>
    <Card>
      <p slot="title">
        <Icon type="md-pricetag"/>
        商品推荐管理
      </p>
      <div>
        <template>
          <Row>
            <Col span="15">
              <RadioGroup v-model="recommendStatus" @on-change="updateRecommendStatus" type="button"
                          style="margin-right: 5px">
                <Radio label="-1" :disabled="recommendStatus==-1">全部</Radio>
                <Radio label="0" :disabled="recommendStatus==0">未推荐</Radio>
                <Radio label="1" :disabled="recommendStatus==1">已推荐</Radio>
              </RadioGroup>

              <Button :disabled="selections.length === 0 || setting.loading" type="primary"
                      icon="md-trending-up" @click="listOnRecommend">上推荐
              </Button>
              <Button :disabled="selections.length === 0 || setting.loading" type="warning"
                      icon="md-trending-down" @click="listOffRecommend">下推荐
              </Button>
              <Button :disabled="setting.loading" type="success" icon="md-refresh" @click="getData">刷新数据</Button>

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
    <Modal title="商品小图查看" v-model="visible">
      <div style="width: 100%;text-align: center">
        <img :src="visibleURL" style="height: 300px">
      </div>

    </Modal>
    <Modal v-model="changeRecommendStatusModel" width="360">
      <p slot="header" style="color:#f60;text-align:center">
        <Icon type="information-circled"></Icon>
        <span>状态修改确认</span>
      </p>
      <div style="text-align:center">
        <p v-if="changeRecommendStatusObject">确定 <b>{{changeRecommendStatusObject.status === 0? '推荐':'撤销'}}</b> 此商品</p>
      </div>
      <div slot="footer">
        <Button type="error" size="large" long :loading="setting.loading" @click="changeStatus">确认</Button>
      </div>
    </Modal>
  </div>
</template>

<script>
  import dayjs from 'dayjs'
  import {post} from '@/libs/axios-cfg'

  export default {
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
        productData: {},
        dataFilter: {
          page: 1,
          pageSize: 10
        },
        selections: [],
        removeModal: false,
        removeObject: null,
        recommendStatus: -1,
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
            title: '推荐状态',
            key: 'recommendStatus',
            sortable: true,
            render: (h, params) => {
              return h('div', [
                h('Button', {
                  props: {
                    type: params.row.recommendStatus === 0 ? 'warning' : 'primary',
                    size: 'small',
                    ghost: true
                  },
                  on: {
                    click: () => {
                      this.changeRecommendStatusObject = {
                        status: params.row.recommendStatus,
                        index: params.index,
                        productId: params.row.productId,
                        reload: false
                      }
                      this.ids = [params.row.productId]
                      this.changeRecommendStatusModel = true;
                    }
                  }
                }, params.row.recommendStatus === 0 ? '未推荐' : '已推荐')
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
                      this.productInfo = params.row;
                      console.log(params.row)
                      this.visible = true;
                      this.visibleURL = params.row.productIcon;
                    }
                  }
                }, '图片')

              ]);
            }
          }
        ],
        visible: false,
        visibleURL: '',
        changeRecommendStatusModel: false,
        changeRecommendStatusObject: null,
        ids: null
      }
    },
    created() {
      this.getData()
    },
    mounted() {

    },
    methods: {
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
        this.ids = ids;
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
          let res = await post('/system/productInfo/select/page/recommendStatus', {
            recommendStatus: this.recommendStatus,
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
       * @description 获取商品列表
       */
      updateRecommendStatus(val) {
        this.recommendStatus = val;
        this.getData()
      },
      /**
       * @description post商品推荐设置
       */
      changeStatus() {
        this.setting.loading = true;
        this.changeRecommendStatusModel = false;
        if (this.changeRecommendStatusObject == null) {
          this.$Message.warning("修改商品为空，无法继续执行！");
          return false;
        }

        if (this.changeRecommendStatusObject.status === 0) {
          this.onRecommend();
        } else {
          this.offRecommend()
        }
      },
      /**
       * @description post商品上推荐
       */
      async onRecommend() {
        this.changeRecommendStatusModel = false;
        if (this.ids == null) {
          this.$Message.warning("修改商品为空，无法继续执行！");
          return false;
        }
        try {
          let res = await post('/system/productInfo/onRecommend', this.ids)
          if (res.code === 1) {
            if (this.changeRecommendStatusObject.reload) {
              this.getData()
            } else {
              this.$set(this.productData.records[this.changeRecommendStatusObject.index], 'recommendStatus', 1)
            }
            this.$Message.success(res.msg);
            this.changeRecommendStatusObject = null;
            this.ids = null;
            this.selections = [];
          }
        } catch (error) {
          this.$throw(error)
        }
        this.setting.loading = false;
      },
      /**
       * @description post商品下推荐
       */
      async offRecommend() {
        this.changeRecommendStatusModel = false;
        if (this.ids == null) {
          this.$Message.warning("修改商品为空，无法继续执行！");
          return false;
        }
        try {
          let res = await post('/system/productInfo/offRecommend', this.ids)
          if (res.code === 1) {
            if (this.changeRecommendStatusObject.reload) {
              this.getData()
            } else {
              this.$set(this.productData.records[this.changeRecommendStatusObject.index], 'recommendStatus', 0)
            }
            this.changeRecommendStatusObject = null;
            this.ids = null;
            this.selections = [];
            this.$Message.success(res.msg);
          }
        } catch (error) {
          this.$throw(error)
        }
        this.setting.loading = false;
      },
      /**
       * @description 商品下推荐
       */
      listOnRecommend() {
        this.changeRecommendStatusObject = {
          status: 0,
          reload: true
        }
        this.changeRecommendStatusModel = true;
      },
      listOffRecommend() {
        this.changeRecommendStatusObject = {
          status: 1,
          reload: true
        }
        this.changeRecommendStatusModel = true;
      }

    }
  }
</script>

<style scoped>

</style>
