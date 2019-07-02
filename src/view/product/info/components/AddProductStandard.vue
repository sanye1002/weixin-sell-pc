<template>
  <div ref="standrad">
    <Row class="margin-bottom-10">
      <Col span="4">
        <b>商品规格</b>
      </Col>
      <Col span="4" offset="1">
        <Row>
          <Col span="6"><b>价格</b></Col>
          <Col span="18">
            <!--<Checkbox v-model="addData.productPriceAll" >统一修改</Checkbox>-->
          </Col>
        </Row>
      </Col>
      <Col span="4" offset="1">
        <Row>
          <Col span="6"><b>原价</b></Col>
          <Col span="18">
            <!--<Checkbox v-model="addData.productPriceAll">统一修改</Checkbox>-->
          </Col>
        </Row>

      </Col>
      <Col span="7" offset="1">
        <Row>
          <Col span="4"><b>库存</b></Col>
          <Col span="20">
            <Checkbox v-model="addData.unlimitedStock" @on-change="checkAll">统一修改</Checkbox>
          </Col>
        </Row>
      </Col>
      <Col span="2">
        <Button size="small" @click="getStandardData" :type="sureData?'warning':'success'"
                v-if="addData.standardList"> {{sureData?'修改':'确认'}}
        </Button>
      </Col>
    </Row>
    <Row v-for="(item, index) in addData.standardList" v-if="addData.standardList" class="margin-bottom-10" type="flex"
         justify="center"
         align="middle" :key="index">
      <Col span="4">
        <Input v-model="item.name" :disabled="sureData"></Input>
      </Col>
      <Col span="4" offset="1">
        <Input v-model="item.productPrice" :disabled="addData.productPriceAll||sureData" prefix="logo-usd"
               placeholder="0.00" style="width:100%"/>
      </Col>
      <Col span="4" offset="1">
        <Input v-model="item.originalPrice" :disabled="addData.productPriceAll||sureData" prefix="logo-usd"
               placeholder="非必填" style="width:100%"/>
      </Col>
      <Col span="7" offset="1">
        <Input v-model="item.productStock" :disabled="item.stockUnlimited||allChooseUnlimitedStock||sureData"
               style="width:100%">
          <Checkbox v-model="item.stockUnlimited" slot="append" @on-change="checkStockUnlimited(index)">不限</Checkbox>
        </Input>
      </Col>
      <Col span="1" offset="1">
        <Icon style="cursor: pointer" type="ios-trash" size="20" @click="deleteStandardModel(index)"></Icon>
      </Col>
    </Row>
    <Row type="flex" justify="center" align="middle">
      <Col span="4">
        <Button icon="md-add" @click="addStandardModel">增加规格</Button>
      </Col>
    </Row>


  </div>

</template>

<script>
  export default {
    name: "AddProductStandard",
    data() {
      return {
        sureData: false,
        allChooseUnlimitedStock: false,
        addData: {
          productPriceAll: false,
          unlimitedStock: false,
          standardList: [
            {}
          ]
        }

      }
    },
    props: {
      values: {}
    },
    methods: {
      /**
       * @description 添加数据
       */
      addStandardModel() {
        this.addData.standardList.push({
          name: '',
          productPrice: null,
          originalPrice: null,
          stockUnlimited: false,
          productStock: 0
        })
      },
      /**
       * @description 删除数据
       */
      deleteStandardModel(index) {
        this.addData.standardList.splice(index, 1)
      },
      /**
       * @description 获取数据
       */
      getStandardData() {
        this.sureData = !this.sureData;
        if (this.sureData) {
          let add = this.addData;
          this.$emit("get-standard-data", add, this.sureData);
        } else {
          let add = {}
          this.$emit("get-standard-data", add, this.sureData);
        }

      },
      checkAll(val) {
        this.allChooseUnlimitedStock = val;
      },
      checkStockUnlimited(index) {
        console.log(this.addData.standardList[index].stockUnlimited)
      }
    },
    mounted() {
      this.addData = this.values;
    }
  }
</script>

<style scoped>

</style>
