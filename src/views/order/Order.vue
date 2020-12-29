<template>
  <div class="order">
    <div class="header"><HeaderInfo></HeaderInfo></div>
    <div class="content">
      <div class="contentHeader">
        <div class="contentHeaderTitle">新建订单</div>
      </div>
      <div class="contentArea">
        <div class="mainTitle">新建订单</div>
        <div class="titleTip">新建订单后请到小程序查看订单状态</div>
        <!-- 表单 -->
        <div>
          <el-form label-position="top" label-width="80px">
            <el-form-item label="产品图片">
              <el-upload
                :action="requestUrl + '/file'"
                list-type="picture-card"
                :headers="uploadHeader"
                :data="{ type: 0 }"
                :auto-upload="true"
                :on-success="uploadSuccess"
                :file-list="fileList"
                :before-remove="doLogin"
              >
                <i slot="default" class="el-icon-plus"></i>
                <div slot="file" slot-scope="{ file }">
                  <img
                    class="el-upload-list__item-thumbnail"
                    :src="file.url"
                    alt=""
                  />
                  <span class="el-upload-list__item-actions">
                    <span
                      class="el-upload-list__item-preview"
                      @click="handlePictureCardPreview(file)"
                    >
                      <i class="el-icon-zoom-in"></i>
                    </span>
                    <span
                      v-if="!disabled"
                      class="el-upload-list__item-delete"
                      @click="handleRemove(file, fileList)"
                    >
                      <i class="el-icon-delete"></i>
                    </span>
                  </span>
                  <el-input v-model="file.name"></el-input>
                </div>
              </el-upload>
              <el-dialog :visible.sync="dialogVisible">
                <img width="100%" :src="dialogImageUrl" alt="" />
              </el-dialog>
            </el-form-item>
            <el-form-item label="选择客户">
              <div class="chooseCustomer">
                <el-select v-model="fkCustomerAccount" placeholder="请选择客户">
                  <el-option
                    v-for="item in customerList"
                    :key="item.id"
                    :label="item.name"
                    :value="item.account"
                  >
                  </el-option>
                </el-select>
                <el-button type="info" plain @click="toCreateUser"
                  >创建新客户</el-button
                >
              </div>
            </el-form-item>
          </el-form>
          <el-form label-position="top">
            <el-form-item label="大货订单号">
              <el-input v-model="orderNum" placeholder="请输入大货订单号">
              </el-input>
            </el-form-item>
            <el-form-item label="客户订单号">
              <el-input v-model="productNum" placeholder="请输入客户订单号">
              </el-input>
            </el-form-item>
          </el-form>
          <el-form label-position="left" label-width="80px">
            <div class="deliveryType">
              <el-form-item label="面料来源">
                <el-radio-group v-model="origin">
                  <el-radio label="加工" value="加工"></el-radio>
                  <el-radio label="成品" value="成品"></el-radio>
                </el-radio-group>
              </el-form-item>
            </div>
          </el-form>
          <el-form label-position="top" label-width="80px">
            <el-form-item label="面料品名">
              <el-input
                v-model="fabricPinming"
                placeholder="请输入面料品名(多个面料用;隔开)"
              >
              </el-input>
              <!-- <div class="chooseF">
                <el-select
                  v-model="fabricName"
                  placeholder="请选择面料"
                  clearable
                >
                  <el-option
                    v-for="item in fabricList"
                    :key="item.id"
                    :label="item.name"
                    :value="item.name"
                  >
                  </el-option>
                </el-select>
              </div> -->
            </el-form-item>
            <el-form-item label="跟单员">
              <div class="chooseF">
                <el-select
                  v-model="fkColorAccount"
                  placeholder="请选择跟单员"
                  clearable
                >
                  <el-option
                    v-for="item in fkColorList"
                    :key="item.id"
                    :label="item.name"
                    :value="item.name"
                  >
                  </el-option>
                </el-select>
              </div>
            </el-form-item>
            <el-row>
              <el-col :span="11">
                <el-form-item label="单价" class="priceDanwei">
                  <el-input v-model="unitPrice" placeholder="请输入单价">
                    <template slot="append">
                      <el-select
                        v-model="priceDanweiIndex"
                        class="priceDanweiSelect"
                      >
                        <el-option
                          v-for="item in priceDanweiList"
                          :key="item.id"
                          :label="item.value"
                          :value="item.id"
                        >
                        </el-option>
                      </el-select>
                    </template>
                  </el-input>
                </el-form-item>
              </el-col>
              <el-col :span="11" :offset="2">
                <el-form-item
                  :label="meterDanweiList[priceDanweiIndex].value + '数'"
                >
                  <el-input
                    v-model="meter"
                    :placeholder="
                      '请输入' + meterDanweiList[priceDanweiIndex].value + '数'
                    "
                  >
                    <template slot="append">{{
                      meterDanweiList[priceDanweiIndex].value
                    }}</template>
                  </el-input>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row>
              <!-- <el-col :span="12">
                <el-form-item label="是否加急">
                  <div class="chooseF">
                    <el-switch
                      v-model="expedite"
                      active-color="#13ce66"
                      :active-value="1"
                      :inactive-value="0"
                    >
                    </el-switch>
                  </div>
                </el-form-item>
              </el-col> -->
              <el-col :span="12">
                <el-form-item label="是否含税">
                  <el-switch
                    v-model="tax"
                    active-color="#13ce66"
                    :active-value="1"
                    :inactive-value="0"
                  >
                  </el-switch>
                </el-form-item>
              </el-col>
            </el-row>
            <el-form-item label="损益">
              <el-row>
                <el-col :span="11">
                  <el-input v-model="approximatelyLeft" placeholder="最小损益">
                    <template slot="append">%</template>
                  </el-input>
                </el-col>
                <el-col :span="2">
                  <div style="text-align: center">~</div>
                </el-col>
                <el-col :span="11">
                  <el-input v-model="approximatelyRight" placeholder="最大损益">
                    <template slot="append">%</template>
                  </el-input>
                </el-col>
              </el-row>
            </el-form-item>
            <el-form-item label="运输方式">
              <!-- <el-radio-group v-model="transportationType">
                <el-radio label="FOB上海" value="FOB上海"></el-radio>
                <el-radio label="CIF" value="CIF"></el-radio>
                <el-radio label="CNF" value="CNF"></el-radio>
                <el-radio
                  label="含税含运(国内)"
                  value="含税含运(国内)"
                ></el-radio>
              </el-radio-group> -->
              <div class="chooseF">
                <el-select
                  v-model="transportationType"
                  placeholder="请选择运输方式"
                  clearable
                >
                  <el-option
                    v-for="item in transportationList"
                    :key="item.id"
                    :label="item.name"
                    :value="item.name"
                  >
                  </el-option>
                </el-select>
              </div>
            </el-form-item>
            <el-form-item label="包缸费">
              <el-input v-model="cylinder" placeholder="包缸费">
                <template slot="append">元</template>
              </el-input>
            </el-form-item>
            <el-form-item label="收货时间">
              <!-- <el-date-picker
                v-model="deliveryTime"
                type="date"
                value-format="yyyy-MM-dd"
                placeholder="请选择收货时间"
              >
              </el-date-picker> -->
              <el-row>
                <el-col :span="11">
                  <el-input v-model="jiahuoTime" placeholder="最短收货时间">
                    <template slot="append">天</template>
                  </el-input>
                </el-col>
                <el-col :span="2">
                  <div style="text-align: center">~</div>
                </el-col>
                <el-col :span="11">
                  <el-input v-model="jiahuoTimeTwo" placeholder="最长收货时间">
                    <template slot="append">天</template>
                  </el-input>
                </el-col>
              </el-row>
            </el-form-item>
          </el-form>
          <el-form label-position="left" label-width="80px">
            <div class="deliveryType">
              <el-form-item label="快递方式">
                <el-radio-group v-model="deliveryType">
                  <el-radio label="物流" value="物流"></el-radio>
                  <el-radio label="自提" value="自提"></el-radio>
                  <el-radio label="海运" value="海运"></el-radio>
                  <el-radio label="陆运" value="陆运"></el-radio>
                </el-radio-group>
              </el-form-item>
            </div>
          </el-form>
          <div class="priceAll">
            <div>总金额:</div>
            <div class="priceAllInfo">{{ priceAll }}</div>
            <div class="priceAllRight" v-if="!tax">元(不含税)</div>
            <div class="priceAllRight" v-else>元(含税)</div>
          </div>
          <el-form label-position="top" label-width="80px">
            <div class="deliveryType">
              <el-form-item label="备注">
                <el-input type="textarea" v-model="remark"></el-input>
              </el-form-item>
            </div>
            <el-form-item>
              <el-button style="width=100%;" type="primary" @click="onSubmit"
                >确认提交</el-button
              >
            </el-form-item>
          </el-form>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import HeaderInfo from '../components/headerInfo'

export default {
  name: "Order",
  data () {
    return {
      customerInfo: {
        account: '',
        password: '111111',
        name: '',
        companyName: '',
        address: ''
      },
      dialogImageUrl: '',
      dialogVisible: false,
      disabled: false,
      fileList: [],
      token: '',
      uploadHeader: {},
      //客户id
      fkCustomerId: '',
      //客户列表
      customerList: [],
      fkCustomerAccount: '',
      // 调色员列表
      fkColorList: [],
      fkColorAccount: '',
      //面料列表
      fabricList: [],
      // 配置方案列表
      configList: [],
      // 选中的配置方案
      configName: '',
      //选中的面料规格
      fabricName: '',
      //米数
      meter: '',
      //单价
      unitPrice: '',
      //收货时间
      deliveryTime: '',
      //快递方式
      deliveryType: '物流',
      //运输方式
      transportationType: '',
      transportationList: [
        { id: 0, name: 'FOB上海' },
        { id: 1, name: 'CIF' },
        { id: 2, name: 'CNF' },
        { id: 3, name: '含税含运(国内)' }
      ],
      remark: "",
      //面料来源
      origin: '加工',
      // 花型路径
      flowerPath: '',
      fileSize: 20,
      // requestUrl:'http://192.168.0.104:9999',
      // requestUrl: 'http://192.168.1.115:9911',//测试
      // requestUrl: 'http://47.111.6.13:10000/test',//测试
      requestUrl: 'https://yc.jiangmiantex.com/jiangjin',//线上
      again: false,
      //0不加急1加急
      expedite: 0,
      arrJingxiao: [],
      arrJiagong: [],
      // 浆料配方列表
      sizingList: [],
      //浆料名
      sizingName: '',
      sizingIndex: '',
      tax: 0,
      cylinder: '',
      // approximately: '',
      priceAll: 0,
      //大货订单号
      orderNum: '',
      //产品拉钩编号
      productNum: '',
      //面料品名
      fabricPinming: '',
      //单价单位
      priceDanweiIndex: 0,
      priceDanweiList: [
        { id: 0, value: '元/米' },
        { id: 1, value: '元/码' },
        { id: 2, value: '元/片' },
        { id: 3, value: '元/件' }
      ],
      meterDanweiList: [
        { id: 0, value: '米' },
        { id: 1, value: '码' },
        { id: 2, value: '片' },
        { id: 3, value: '件' }
      ],
      //损益
      approximatelyLeft: '',
      approximatelyRight: '',
      jiahuoTime: '',
      jiahuoTimeTwo: ''
    };
  },
  components: {
    HeaderInfo: HeaderInfo
  },
  methods: {
    // 时间加
    dateAddDays (dateStr, dayCount) {
      var tempDate = new Date(dateStr.replace(/-/g, "/"));//把日期字符串转换成日期格式
      var resultDate = new Date((tempDate / 1000 + (86400 * dayCount)) * 1000)
      var resultDateStr = resultDate.getFullYear() + "-" + (resultDate.getMonth() + 1) + "-" + (resultDate.getDate());//将日期转化为字符串格式
      console.log(resultDateStr)
      return resultDateStr
    },
    onSubmit () {
      //验证
      if (this.fileList.length == 0) {
        this.$message.error("请选择产品图片");
      } else {
        let str = '';
        for (var i = 0; i < this.fileList.length; i++) {
          if (!this.fileList[i].name) {
            this.$message.error("请输入产品拉钩编号");
            return;
          }
          if (this.again && this.fileList[i].urlNext) {
            str += this.fileList[i].name + "," + this.fileList[i].urlNext + ',' + this.fileList[i].num + ';';
          } else {
            this.fileList[i].name = this.fileList[i].name.slice(0, this.fileList[i].name.lastIndexOf('.'));
            str += this.fileList[i].name + "," + this.fileList[i].response.data + ',' + this.fileList[i].num + ';';
          }
        }

        if (!this.fkCustomerAccount) {
          this.$message.error("请选择客户");
        } else if (!this.fabricPinming) {
          this.$message.error("请输入面料品名");
        } else if (!this.meter) {
          this.$message.error(`请输入${this.meterDanweiList[this.priceDanweiIndex].value}数`);
        } else if (!Number(this.meter)) {
          this.$message.error(`请输入正常的${this.meterDanweiList[this.priceDanweiIndex].value}数`);
        } else if (!this.unitPrice) {
          this.$message.error("请输入单价");
        } else if (!this.orderNum) {
          this.$message.error("请输入大货订单号");
        } else if (!this.productNum) {
          this.$message.error("请输入客户订单号");
        } else if (!Number(this.unitPrice)) {
          this.$message.error("请输入正常的单价");
        } else if (!this.jiahuoTime || !this.jiahuoTimeTwo) {
          this.$message.error("请输入收货时间");
        } else if (!this.cylinder) {
          this.$message.error("请输入包缸价");
        } else if (!this.approximatelyLeft || !this.approximatelyRight) {
          this.$message.error("请输入损益范围")
        } else if (!this.fkColorAccount) {
          this.$message.error("请选择跟单员")
        } else {
          var customer = '';
          var fabricInfo = '';
          let colorDetail = ''
          this.customerList.forEach((item, index) => {
            if (item.account == this.fkCustomerAccount) {
              customer = item.userDetail
            }
          })
          this.fkColorList.forEach(item => {
            if (item.name == this.fkColorAccount) {
              colorDetail = item;
            }
          })
          this.fabricList.forEach(item => {
            if (item.name == this.fabricName) {
              fabricInfo = item
            }
          })
          const d = new Date()
          const nowDate = `${d.getFullYear()}-${d.getMonth() + 1}-${d.getDate()}`
          const jiahuoTime = this.dateAddDays(nowDate, this.jiahuoTime)
          const jiahuoTimeTwo = this.dateAddDays(nowDate, this.jiahuoTimeTwo)
          const obj = {
            companyName: customer.companyName,
            fkCustomerId: customer.fkUserId,
            contacts: customer.contacts,
            phone: customer.phone,
            address: customer.address,
            fabricCustom: this.fabricPinming,
            // fabricName: this.fabricName,
            // fkFabricId: fabricInfo.id,
            meter: this.meter,
            unitPrice: this.unitPrice,
            origin: this.origin,
            note: this.remark,
            deliveryTime: jiahuoTime + ' 23:59:59',
            deliveryTimeStart: jiahuoTimeTwo + ' 23:59:59',
            deliveryType: this.deliveryType,
            detailStrs: str,
            colorName: colorDetail.name,
            fkColorId: colorDetail.id,
            expedite: this.expedite,
            tax: this.tax,
            extraCharges: this.cylinder,
            otherChargesRate: this.approximatelyLeft + '~' + this.approximatelyRight,
            bigOrderNum: this.orderNum,
            productNum: this.productNum,
            unit: this.priceDanweiIndex,
            transportationType: this.transportationType
          }
          this.$post('/order', obj).then((data) => {
            this.$message.success(data.message);
            localStorage.removeItem('carList');
            this.$router.push({
              path: '/messageResult',
              params: { resultData: data }
            })
          })
        }
      }
    },
    toCreateUser () {
      this.$router.push({
        name: 'AddCustomer',
        params: {
          isLine: 1  //判断是否是从订单页面跳转
        }
      })
    },
    handleRemove (file, fileList) {
      var fileListResult = [];
      for (var i = 0; i < this.fileList.length; i++) {
        if (file.name != this.fileList[i].name || file.response.data != this.fileList[i].response.data) {
          fileListResult.push(this.fileList[i]);
        }
      }
      this.fileList = fileListResult;
    },
    //图片放大
    handlePictureCardPreview (file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    doLogin () {
      return false;
    },
    uploadSuccess (response, file, fileList) {
      var types = 'jpg,jpeg,png,JPG,PNG,JPEG'
      if (types.indexOf(file.name.substring(file.name.indexOf('.') + 1, file.name.length)) <= -1) {
        this.$message.error('请上传图片格式的文件')
        fileList = fileList.slice(0, fileList.length - 1)
      } else if (file.name.indexOf('=') !== -1 || file.name.indexOf('&') !== -1 || file.name.indexOf(',') !== -1 || file.name.indexOf(';') !== -1) {
        this.$message.error('上传的图片名字中不能包含"="、";"、"&"或","')
        fileList = fileList.slice(0, fileList.length - 1)
      } else {
        fileList[fileList.length - 1].url = response.wwwFileBaseUrl + response.data;
        fileList[fileList.length - 1].num = fileList[fileList.length - 1].name;
      }
      this.fileList = fileList;
    },
    //获取客户列表
    getCustomerList () {
      this.$get('/customer', {
        page: 1,
        rows: 999999
      }).then((data) => {
        this.customerList = data.data.list;
        if (this.fkCustomerId != '') {
          this.customerList.forEach((item, index) => {
            if (item.id == this.fkCustomerId) {
              this.fkCustomerAccount = item.account;
            }
          })
        }
      })
    },
    // 获取调色员列表
    getFkColorList () {
      this.$get('/user', {
        page: 1,
        rows: 99999,
        role: 7
      }).then((data) => {
        this.fkColorList = data.data.list
      })
    },
    //获取面料列表
    getFabricList () {

      this.$get('/fabric/select', {
        page: 1,
        rows: 999999,
        fkCustomerId: -10
      }).then((data) => {
        this.fabricList = data.data
      })
    },
    //总价
    priceAllM () {
      if (this.meter && this.unitPrice && this.cylinder) {
        this.priceAll = this.meter * this.unitPrice + Number(this.cylinder)
      }
    }
  },
  mounted () {
    this.token = localStorage.getItem("token");
    this.uploadHeader = { 'token': this.token }
  },
  created () {
    this.getFabricList()
    this.getCustomerList()
    this.getFkColorList()
    if (localStorage.getItem("carList")) {
      this.fileList = JSON.parse(localStorage.getItem("carList"))
      this.fileList.forEach(item => {
        item.num = item.name
      })
    }
    var _this = this;
    document.onkeydown = function (e) {
      let key = window.event.keyCode;
      if (key == 8) {
        _this.doLogin();
      }
    }
  },
  watch: {
    tax () {
      this.priceAllM()
    },
    meter () {
      this.priceAllM()
    },
    unitPrice () {
      this.priceAllM()
    },
    cylinder () {
      this.priceAllM()
    }
  }
}
</script>
<style>
.order .el-form-item__label {
  line-height: 100%;
}

.order .el-upload-list--picture-card .el-upload-list__item {
  overflow: inherit !important;
}
*:focus {
  outline: none;
}
.order .el-upload-list__item .el-input__inner {
  border: none;
}
.order .el-upload-list__item > :first-child {
  height: 100%;
}
.chooseCustomer .el-select > .el-input {
  width: 343px;
}
.chooseF .el-select > .el-input {
  width: 460px;
}
.el-date-editor.el-input,
.el-date-editor.el-input__inner {
  width: 100%;
}
.deliveryType .el-form-item__label {
  line-height: 40px;
}
</style>
<style scoped>
.el-button--primary {
  width: 100%;
}
.order {
  min-width: 1440px;
}
.header {
  box-shadow: 0 2px 35px 0 rgba(28, 34, 38, 0.1);
  position: relative;
}
.content {
  width: 1200px;
  margin: 51px auto;
}
.contentHeader {
  display: flex;
  margin-bottom: 22px;
}
.contentHeaderTitle {
  font-family: PingFangSC-Medium;
  font-size: 20px;
  color: rgba(0, 0, 0, 0.85);
  line-height: 28px;
}
.contentArea {
  width: 460px;
  background: #ffffff;
  box-shadow: 0 2px 35px 0 rgba(28, 34, 38, 0.05);
  border-radius: 6px;
  padding: 63px 370px 67px;
}
.contentArea .mainTitle {
  font-family: PingFangSC-Regular;
  font-size: 40px;
  color: #000000;
  letter-spacing: -1.33px;
  line-height: 40px;
}
.titleTip {
  font-family: PingFangSC-Regular;
  font-size: 20px;
  color: #999999;
  letter-spacing: -0.55px;
  margin-top: 10px;
  margin-bottom: 65px;
}
.copyright {
  font-family: PingFangSC-Regular;
  font-size: 12px;
  color: rgba(0, 0, 0, 0.45);
  line-height: 20px;
  margin-top: 30px;
  text-align: center;
  margin-bottom: 30px;
}
.priceAll {
  height: 40px;
  display: flex;
}
.priceAllInfo {
  margin-left: 10px;
}
.priceAllRight {
  margin-left: 10px;
}
.priceDanweiSelect {
  width: 80px;
}
</style>
