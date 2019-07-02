<template>
  <div>
    <Modal v-model="show" :title="updateStatus? '更新商品':'添加商品'" :loading="loading"
           :mask-closable="false" :closable="false">
      <template>
        <div class="product-icon-box">
          <div class="product-icon-item">
            <div v-if="imgShow||updateStatus" class="product-icon margin-bottom-10"
                 :class="progressShow?'border-dashed':''">
              <img v-if="productData.productIcon" :src="productData.productIcon" alt="">
              <template>
                <Progress class="progress-upload" v-if="progressShow" :stroke-width="5" :percent="progressNumber"
                          status="active" hide-info></Progress>
              </template>
            </div>

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
                <Button icon="ios-cloud-upload-outline">{{productData.productIcon?'修改商品图片':'上传商品图片'}}</Button>
              </Upload>
            </template>
          </div>
        </div>
      </template>
      <Form ref="modalForm" :model="productData" :rules="ruls"
            :label-width="80">
        <FormItem label="商品名称" prop="productName">
          <Input v-model.trim="productData.productName"></Input>
        </FormItem>
        <FormItem label="商品单价" prop="productPrice">
          <InputNumber :min="0" :step="1" v-model.trim="productData.productPrice" style="width:100%"/>
        </FormItem>
        <FormItem label="商品库存" prop="productStock">
          <InputNumber :min="0" :step="1" v-model.trim="productData.productStock" style="width:100%"/>
        </FormItem>
        <FormItem label="类目编号" prop="categoryType">
          <Select v-model.trim="productData.categoryType" style="width:100%" filterable>
            <Option v-for="item in categoryList"
                    :value="item.categoryType" :key="item.categoryId">{{ item.categoryName }}
            </Option>
          </Select>
        </FormItem>
        <FormItem label="商品描述" prop="productDescription">
          <Input type="textarea" v-model.trim="productData.productDescription" :autosize="{minRows: 2,maxRows: 5}"
                 placeholder="商品描述内容..."></Input>
        </FormItem>
        <FormItem label="规格设置">
          <Tooltip content="规格设置可以在页面中商品选购中进行规格选择" placement="top-start" max-width="200">
            <i-switch v-model="productStandardStatus" size="large">
              <span slot="open">添加</span>
              <span slot="close">无</span>
            </i-switch>
          </Tooltip>
        </FormItem>
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

  export default {
    data() {
      return {
        show: true,
        loading: false,
        ruls: {
          productName: [{required: true, message: "商品名称不能为空"}],
          productPrice: [{type: 'number', required: true, message: "请商品单价"}],
          productStock: [{type: 'number', required: true, message: "请商品库存"}],
          productIcon: [{required: true, message: "请上传商品图片"}],
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
          productStock: 100,
          salesVolume: 0,
          productDescription: "好吃的老雷2",
          productIcon: "https://p1.pstatp.com/list/190x124/pgc-image/c5b1dfe45f5f4e0789d5c93572dbf728",
          productStatus: 0,
          categoryType: 200
        }
      };
    },
    props: {
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
        productStandardStatus:false,
        productStandard:{}
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
            let data = JSON.parse(JSON.stringify(this.productData));
            this.add(data)
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
            this.cancel(true)
            this.$Message.success("商品 " + data.productName + this.updateStatus?'更新成功':'添加成功');
          }
        } catch (error) {
          this.$throw(error)
        }
        console.log(data)
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
    width: 120px;
    height: 120px;
    border-radius: 2px;
    position: relative;
  }

  .product-icon img {
    width: 120px;
    height: 120px;
  }

  .progress-upload {
    position: absolute;
    top: 55px;
  }
</style>

