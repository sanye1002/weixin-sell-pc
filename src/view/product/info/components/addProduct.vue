<template>
  <div>
    <Modal v-model="show" :title="updateStatus? '更新商品':'添加商品'" :loading="loading"
           :mask-closable="false" :closable="false" width="660">

      <Form ref="modalForm" :model="productData" :rules="ruls"
            label-position="top">
        <Row>
          <Col span="8">
            <FormItem label="图片" prop="productIcon">
              <template>
                <div class="product-icon-box">
                  <div class="product-icon-item">
                    <div class="product-icon margin-bottom-10"
                         :class="progressShow?'border-dashed':''">
                      <img v-if="productData.productIcon" :src="productData.productIcon" alt="">
                      <Icon class="icon-position" size="40" type="ios-camera"
                            v-if="!progressShow&&!productData.productIcon"/>
                      <template>
                        <Progress class="progress-upload" v-if="progressShow" :stroke-width="5"
                                  :percent="progressNumber"
                                  status="active" hide-info></Progress>
                      </template>
                      <template>
                        <Upload class="margin-bottom-10"
                                ref="upload"
                                action="https://news.immnc.com/oa/upload/img/user"
                                :show-upload-list="false"
                                :before-upload="beforeUpload"
                                :on-success="handleSuccess"
                                :format="['jpg','jpeg','png']"
                                :max-size="2048"
                                :on-format-error="handleFormatError"
                                :on-exceeded-size="handleMaxSize">
                          <div class="upload-button-cover">
                            <Icon type="ios-camera" size="40"></Icon>
                            <!--<Button type="text" icon="ios-cloud-upload-outline">{{productData.productIcon?'修改商品图片':'上传商品图片'}}</Button>-->
                          </div>
                        </Upload>
                      </template>
                    </div>
                  </div>
                </div>
              </template>
            </FormItem>
          </Col>
          <Col span="16">
            <FormItem label="商品名称" prop="productName">
              <Input v-model.trim="productData.productName"></Input>
            </FormItem>
            <FormItem label="商品描述" prop="productDescription">
              <Input type="textarea" v-model.trim="productData.productDescription" :autosize="{minRows: 4,maxRows: 4}"
                     placeholder="商品描述内容..."></Input>
            </FormItem>
          </Col>
        </Row>
        <FormItem label="类目编号" prop="categoryType">
          <Select v-model.trim="productData.categoryType" style="width:100%" filterable>
            <Option v-for="item in categoryList"
                    :value="item.categoryType" :key="item.categoryId">{{ item.categoryName }}
            </Option>
          </Select>
        </FormItem>
        <FormItem label="规格设置">
          <Tooltip content="规格设置可以在页面中商品选购中进行规格选择" placement="top-start" max-width="200">
            <i-switch v-model="productData.standardStatus" size="large">
              <span slot="open">添加</span>
              <span slot="close">无</span>
            </i-switch>
          </Tooltip>
        </FormItem>
        <Row v-if="!productData.standardStatus">
          <Col span="5">
            <FormItem label="商品价钱" prop="productPrice">
              <Input prefix="logo-usd" placeholder="0.00" v-model.trim="productData.productPrice" style="width:100%"/>
            </FormItem>
          </Col>
          <Col span="5">
            <FormItem label="商品原价">
              <Input prefix="logo-yen" placeholder="选填" v-model.trim="productData.originalPrice" style="width:100%"/>
            </FormItem>
          </Col>
          <Col span="7" offset="1">
            <FormItem label="商品库存" prop="productStock">
              <Input v-model.trim="productData.productStock" :disabled="productData.unlimitedStock" style="width:100%">
                <Checkbox v-model="productData.unlimitedStock" slot="append">不限</Checkbox>
              </Input>
            </FormItem>
          </Col>
        </Row>
        <AddStandard v-if="productData.standardStatus" @get-standard-data="getStandard"
                     :values="productData.productStandard"></AddStandard>
      </Form>
      <div slot="footer">
        <Button type="default" :disabled="loading" @click="cancel(false)">取消</Button>
        <Button type="primary" :loading="loading" @click="ok">确定</Button>
      </div>
    </Modal>
  </div>
</template>
<script>
  import {post} from '@/libs/axios-cfg'
  import AddStandard from '../components/AddProductStandard.vue'

  export default {
    data() {
      const validateConfirmStock = (rule, value, callback) => {
        if (!this.productData.unlimitedStock && !this.productData.productStock) {
          callback(new Error("请选择库存方式"));
        } else {
          callback();
        }
      };
      const validateConfirmIMG = (rule, value, callback) => {
        if (!this.productData.productIcon) {
          callback(new Error("请上传图片"));
        } else {
          callback();
        }
      };
      return {
        show: true,
        loading: false,
        ruls: {
          productName: [{required: true, message: "商品名称不能为空"}],
          productPrice: [{required: true, message: "请商品价格"}],
          productStock: [{validator: validateConfirmStock}],
          productIcon: [{validator: validateConfirmIMG}],
          categoryType: [{required: true, message: "必须选择商品类目"}],
          productDescription: [{required: true, message: "商品介绍不能为空"}]
        },
        categoryList: [],
        progressShow: false,
        uploadList: [],
        progressNumber: 0,
        progressId: null,
        imgShow: false,
        productData: {
          productId: "",
          productName: "",
          productPrice: 0,
          originalPrice: null,
          productStock: 100,
          salesVolume: 0,
          productDescription: "好吃的老雷2",
          productIcon: "https://p1.pstatp.com/list/190x124/pgc-image/c5b1dfe45f5f4e0789d5c93572dbf728",
          productStatus: 0,
          categoryType: 200,
          unlimitedStock: false,
          standardStatus: false,
          productStandard: {}
        },
        sureData: false
      };
    },
    components: {
      AddStandard
    },
    props: {
      productInfo: {
        productId: "",
        productName: "",
        productPrice: 0,
        originalPrice: null,
        productStock: 100,
        salesVolume: 0,
        productDescription: "好吃的老雷2",
        productIcon: "https://p1.pstatp.com/list/190x124/pgc-image/c5b1dfe45f5f4e0789d5c93572dbf728",
        productStatus: 0,
        categoryType: 200,
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
      },
      updateStatus: false
    },
    created() {
      this.getData()
    },
    methods: {
      /**
       * @description 关闭Modal
       * @param reload 是否重新加载数据
       */
      cancel(reload = false) {
        this.$emit("cancel", "add", reload);
      },
      /**
       * @description 确定按钮单击回调
       */
      ok() {
        this.$refs.modalForm.validate(valid => {
          if (valid) {
            if (this.productData.standardStatus) {
              console.log(this.sureData)
              if (!this.sureData) {
                this.$Message.error("请保存规格信息")
                return false;
              }
            }
            let data = JSON.parse(JSON.stringify(this.productData));
            this.add(data);
          }
        });
      },
      /**
       * @description 获取类目列表
       */
      async getData() {
        this.loading = true;
        try {
          let res = await post('/system/productCategory/list')
          this.categoryList = res.data;
        } catch (error) {
          this.$throw(error)
        }
        this.loading = false;
      },
      /**
       * @description 添加用户数据请求
       */
      async add(data) {
        this.loading = true;
        try {
          let res = null;
          if (this.updateStatus) {
            res = await post('/system/productInfo/update', data)
          } else {
            res = await post('/system/productInfo/add', data)
          }

          if (res.code === 1) {
            let message = "商品 " + data.productName;
            message += this.updateStatus ? '更新成功':'添加成功';
            this.$Notice.success({
              title: this.updateStatus ? '更新成功':'添加成功',
              desc: message
            })
            this.cancel(true)
          }
        } catch (error) {
          this.$throw(error)
        }
        this.loading = false;
      },
      /**
       * @description 文件上传进度条
       */
      handelProgress() {

        console.log("文件上传进度条");

      },
      /**
       * @description 文件上传之前
       */
      beforeUpload() {
        this.imgShow = true;
        this.progressShow = true;
        this.uploadList = [];
        this.progressNumber = 10;
        this.progressId = setInterval(this.progressNumberChange, 150)
        console.log("文件上传之前");
      },
      handleSuccess(res, file) {
        this.progressNumber = 100;
        this.productData.productIcon = res.data.src;
        this.$Message.success("图片上传成功");
        setTimeout(() => {
          if (this.progressId != null) {
            clearInterval(this.progressId)
          }
          this.progressShow = false;
        }, 500);
      },
      handleFormatError(file) {
        this.$Notice.warning({
          title: '图片格式错误',
          desc: '请选择 jpg , jpeg , png 格式的图片.'
        });
      },
      handleMaxSize(file) {
        this.$Notice.warning({
          title: '图片文件过大',
          desc: '请选择2M一下的图片.'
        });
      },
      progressNumberChange() {
        if (this.progressNumber <= 80) {
          this.progressNumber = this.progressNumber + 10
        }
      },
      /**
       * @description 获取数据
       */
      getStandard(val, flag) {
        this.productData.productStandard = val;
        this.sureData = flag;
      }
    },
    mounted() {
      this.productData = this.productInfo;
    }
  };
</script>
<style scoped>
  .product-icon-box {
    display: flex;
    width: 100%;
    justify-content: center;

  }

  .product-icon-item {
    display: flex;
    width: 100%;
    flex-direction: column;
    justify-content: center;
    align-items: center;

  }

  .border-dashed {
    border: 1px dashed #dcdee2;
  }

  .product-icon {
    width: 180px;
    height: 180px;
    border-radius: 2px;
    position: relative;
    background-color: #f8f8f9;
  }

  .product-icon img {
    width: 180px;
    height: 180px;
  }

  .progress-upload {
    position: absolute;
    top: 75px;
  }

  .icon-position {
    position: absolute;
    top: 70px;
    left: 70px;
  }

  .upload-button-cover {
    border-radius: 2px;
    display: none;
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    background: rgba(0, 0, 0, .6);
    line-height: 180px;
    text-align: center;

  }

  .product-icon:hover .upload-button-cover {

    display: block;
  }

  .upload-button-cover i {
    color: #fff;
    cursor: pointer;
    margin: 0 2px;
  }
</style>

