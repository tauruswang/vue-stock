<template>
  <Row type="flex" justify="center">
    <i-col span="24" v-if="controller.length">
      <i-form inline>
        <Form-item label="持有期" :label-width="100">
          <i-input type="text" v-model="controller[0]" readonly></i-input>
        </Form-item>
        <Form-item label="回测区间" :label-width="100">
          <i-input type="text" v-model="controller[1]" readonly></i-input>
        </Form-item>
        <Form-item label="买入费率（‱）" :label-width="120">
          <i-input type="text" v-model="controller[2]" readonly></i-input>
        </Form-item>
        <Form-item label="卖出费率（‱）" :label-width="120">
          <i-input type="text" v-model="controller[3]" readonly></i-input>
        </Form-item>
        <Form-item label="模型压力费率（‱）" :label-width="150">
          <i-input type="text" v-model="controller[4]" readonly></i-input>
        </Form-item>
        <Form-item label="每日最大持股数" :label-width="120">
          <i-input type="text" v-model="controller[5]" readonly></i-input>
        </Form-item>
      </i-form>
    </i-col>
    <i-col span="24">
      <el-table :data="tableData" v-loading="loading" element-loading-text="正在获取智能回测结果..." border size="small"
                style="width: 100%" :empty-text="dataText"
                @sort-change="sortChange">
        <el-table-column type="index" align="center" :width="60"></el-table-column>
        <el-table-column align="center" prop="time" label="日期" v-if="buttonDisabled"></el-table-column>
        <el-table-column align="center" prop="time" sortable="custom" label="日期" v-else></el-table-column>
        <el-table-column align="center" prop="stock_index" label="入市指标">
          <template slot-scope="scope">
            <el-tooltip placement="right" :enterable="false">
              <div class="show-overflow">{{scope.row.stock_index.join('')}}</div>
              <div slot="content">
                <ul>
                  <li class="text-center" v-for="intoMarket in scope.row.stock_index">
                    {{intoMarket}}
                  </li>
                </ul>
              </div>
            </el-tooltip>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="out_index" label="出市指标">
          <template slot-scope="scope">
            <el-tooltip placement="right" :enterable="false">
              <div class="show-overflow">{{scope.row.out_index.join('')}}</div>
              <div slot="content">
                <ul>
                  <li class="text-center" v-for="outMarket in scope.row.out_index">
                    {{outMarket}}
                  </li>
                </ul>
              </div>
            </el-tooltip>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="second_index"
                         label="二次筛选指标">
          <template slot-scope="scope">
            <el-tooltip placement="right" :enterable="false">
              <div class="show-overflow">{{scope.row.second_index.join('')}}</div>
              <div slot="content">
                <ul>
                  <li class="text-center" v-for="second in scope.row.second_index">
                    {{second}}
                  </li>
                </ul>
              </div>
            </el-tooltip>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="wind_index" label="风控指标">
          <template slot-scope="scope">
            <el-tooltip placement="right" :enterable="false">
              <div class="show-overflow">{{scope.row.wind_index.join('')}}</div>
              <div slot="content">
                <ul>
                  <li class="text-center" v-for="wind in scope.row.wind_index">
                    {{wind}}
                  </li>
                </ul>
              </div>
            </el-tooltip>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="year_profit" :width="200" label="年化收益率（复利）"
                         v-if="buttonDisabled">
          <template slot-scope="scope">
            <span
              :class="{'above-zero':(scope.row.year_profit>0),'equal-zero':(scope.row.year_profit===0),'bellow-zero':(scope.row.year_profit<0)}">{{scope.row.year_profit+"%"}}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="year_profit" sortable="custom" :width="200" label="年化收益率（复利）"
                         v-else>
          <template slot-scope="scope">
            <span
              :class="{'above-zero':(scope.row.year_profit>0),'equal-zero':(scope.row.year_profit===0),'bellow-zero':(scope.row.year_profit<0)}">{{scope.row.year_profit+"%"}}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="max_back" label="最大回撤" v-if="buttonDisabled">
          <template slot-scope="scope">
            <span
              :class="{'above-zero':(scope.row.max_back>0),'equal-zero':(scope.row.max_back===0),'bellow-zero':(scope.row.max_back<0)}">{{scope.row.max_back+"%"}}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="max_back" sortable="custom" label="最大回撤"
                         v-else>
          <template slot-scope="scope">
            <span
              :class="{'above-zero':(scope.row.max_back>0),'equal-zero':(scope.row.max_back===0),'bellow-zero':(scope.row.max_back<0)}">{{scope.row.max_back+"%"}}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" prop="empty_rate" label="空仓占比" v-if="buttonDisabled">
          <template slot-scope="scope">
            <span
              :class="{'above-zero':(scope.row.empty_rate>0),'equal-zero':(scope.row.empty_rate===0),'bellow-zero':(scope.row.empty_rate<0)}">{{scope.row.empty_rate+"%"}}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="empty_rate" sortable="custom" label="空仓占比"
                         v-else>
          <template slot-scope="scope">
            <span
              :class="{'above-zero':(scope.row.empty_rate>0),'equal-zero':(scope.row.empty_rate===0),'bellow-zero':(scope.row.empty_rate<0)}">{{scope.row.empty_rate+"%"}}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="win_rate" label="胜率" v-if="buttonDisabled">
          <template slot-scope="scope">
            <span
              :class="{'above-zero':(scope.row.win_rate>0),'equal-zero':(scope.row.win_rate===0),'bellow-zero':(scope.row.win_rate<0)}">{{scope.row.win_rate+"%"}}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="win_rate" sortable="custom" label="胜率" v-else>
          <template slot-scope="scope">
            <span
              :class="{'above-zero':(scope.row.win_rate>0),'equal-zero':(scope.row.win_rate===0),'bellow-zero':(scope.row.win_rate<0)}">{{scope.row.win_rate+"%"}}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" label="操作">
          <template slot-scope="scope">
            <router-link
              :to="{path:'/model',query:{temp_sa_build:geneticModels[scope.$index+(currentPage - 1) * pageSize].geneticModelId}}"
              target="_blank" :disabled="buttonDisabled">重建模型
            </router-link>
          </template>
        </el-table-column>
      </el-table>
      <div style="margin: 10px;overflow: hidden">
        <!--<Button type="primary" size="large" @click="exportData" :disabled="!status">-->
        <!--<Icon type="ios-download-outline"></Icon>-->
        <!--导出-->
        <!--</Button>-->
        <a id="hrefToExportTable" style="position: absolute;left: -10px;top: -10px;width: 0px;height: 0px;"></a>
        <Button type="primary" size="large" :loading="exportLoading" @click="exportData" :disabled="!status" >
          导出为csv
        </Button>
        <Button type="primary" size="large" :loading="exportLoading" @click="exportExcelData" :disabled="!status" >
          <!--<Icon type="ios-download-outline"></Icon>-->
          <span v-if="!exportLoading">导出为excel</span>
          <span v-else>正在导出</span>
        </Button>
        <div style="float: right;">
          <el-pagination layout="total,prev, pager, next" :page-size="pageSize" :current-page.sync="currentPage"
                         :total="geneticModels.length" @current-change="changePage"></el-pagination>
        </div>
      </div>
    </i-col>
    <!--设置隐藏的表格  方便导出-->
    <i-col span="24" v-show="false">
      <i-table border :columns="columns" :data="tableTemp" ref="table"></i-table>
      <div style="margin: 10px;overflow: hidden">
        <div style="float: right;">
          <Page :total="temp.length" :current="1" :page-size="pageSize"
                show-total></Page>
        </div>
      </div>
    </i-col>
  </Row>
</template>
<script>
  import {
    CONTROLLER,
    resolveIndicator,
    indicatorToDes,
    classifyIndicator
  } from '../../api/model'
  import {getRemoteReqTodo, postRemoteReqTodo} from '../../api/api'
  import table2excel from '../../api/table2excel'
  import {ascObj, descObj, loginTimeoutPrompt, jumpLogin} from '../../api/tools'

  export default {
    mounted() {
      this.getGenetic();
    },
    data() {
      return {
        loading: true,
        exportLoading: false,
        temp: [],
        tableTemp: [],
        buttonDisabled: true,
        status: false,
        controller: [],
        dataText: '...',
        geneticModels: [],
        geneticSetTime: '',
        modal: false,
        pageSize: 20,
        currentPage: 1,
        nest: [],
        columns: [
          {
            title: '时间',
            key: 'time',
            align: 'center',
            sortable: 'custom'
          },
          {
            title: '入市指标',
            key: 'stock_index',
            align: 'center',
            ellipsis: true
          },
          {
            title: '出市指标',
            key: 'out_index',
            align: 'center',
            ellipsis: true
          },
          {
            title: '二次筛选指标',
            key: 'second_index',
            align: 'center',
            ellipsis: true
          },
          {
            title: '风控指标',
            key: 'wind_index',
            align: 'center',
            ellipsis: true
          },
          {
            title: '年化收益率（复利）（%）',
            key: 'year_profit',
            align: 'center',
            sortable: 'custom',
            width: '250'
          },
          {
            title: '最大回撤（%）',
            key: 'max_back',
            align: 'center',
            sortable: 'custom',
          }, {
            title: '胜率（%）',
            key: 'win_rate',
            align: 'center',
            sortable: 'custom'
          }, {
            title: '空仓占比（%）',
            key: 'empty_rate',
            align: 'center',
            sortable: 'custom'
          },
          {
            title: '持有期',
            key: 'hold',
            align: 'center',
          },
          {
            title: '回测区间',
            key: 'test_range',
            align: 'center',
          },
          {
            title: '买入费率（万分之）',
            key: 'buy_rate',
            align: 'center',
          },
          {
            title: '卖出费率（万分之）',
            key: 'sell_rate',
            align: 'center',
          },
          {
            title: '模型压力费率（万分之）',
            key: 'pressure_rate',
            align: 'center',
          },
          {
            title: '每日最大持股数',
            key: 'max_hold',
            align: 'center',
          }
        ]
      }
    },
    computed: {
      sortAble() {
        return this.buttonDisabled;
      },
//      某一页的遗传算法模型数据
      tableData() {
        return this.geneticModels.slice((this.currentPage - 1) * this.pageSize, this.currentPage * this.pageSize);
      },
//      tableTemp(){
//        return [];
//      }
    },
    methods: {
      getGenetic() {
        const that = this;
        getRemoteReqTodo('/stock/genetic/getgeneticreport', [], []).then(res => {
          //首先清空我的模型  再重新获取
          let selectedIndexsTemp = [];
          let symbol = {
            andOrNotIntoMarketLeft: '',
            andOrNotIntoMarketRight: '',
            andOrNotOutMarketLeft: '',
            andOrNotOutMarketRight: '',
            andOrNotWindCtrlLeft: '',
            andOrNotWindCtrlRight: '',
          };
          let conCtrl = {};

          let data = res.data;
          if (data.status === 'SUCCESS') {
            that.geneticModels.splice(0, that.geneticModels.length);
            that.tableTemp.splice(0, that.tableTemp.length);
            clearTimeout(that.geneticSetTime);
            that.buttonDisabled = false;
//                结束状态
//           获取报告  每隔4s请求一次
            let controller = CONTROLLER(data.geneticModels[0].modelPara);
            that.controller.push(controller[0]);
            that.controller.push(controller[1] + '~' + controller[2]);
            that.controller.push(controller[3]);
            that.controller.push(controller[4]);
            that.controller.push(controller[5]);
            that.controller.push(controller[6]);
            if (data.geneticModels.length !== 0) {
              let statisticalInfo;
              data.geneticModels.forEach(function (geneticModel, index) {
                resolveIndicator(selectedIndexsTemp, geneticModel.modelPara, conCtrl, symbol);
                statisticalInfo = JSON.parse(geneticModel.report.replace(/\'/g, '\"'));
                that.geneticModels.push({
                  geneticModelId: geneticModel.intelligentModelId.replace(/\-/g, '%'),
                  time: new Date(geneticModel.returnTime).format('yyyy-MM-dd hh:mm:ss'),
                  stock_index: indicatorToDes(classifyIndicator(selectedIndexsTemp, 'A', 'intoMarket'), symbol.andOrNotIntoMarketLeft, symbol.andOrNotIntoMarketRight),
                  out_index: geneticModel.modelPara.indexOf('[SELL]') !== -1 ? indicatorToDes(classifyIndicator(selectedIndexsTemp, 'sell', 'outMarket'),
                    symbol.andOrNotOutMarketLeft, symbol.andOrNotOutMarketRight) : ['无'],
                  second_index: indicatorToDes(classifyIndicator(selectedIndexsTemp, 'B', 'second'), '', ''),
                  wind_index: geneticModel.modelPara.indexOf('[DAN_CON]') !== -1 ? indicatorToDes(classifyIndicator(selectedIndexsTemp, 'C', 'windCtrl'),
                    symbol.andOrNotWindCtrlLeft, symbol.andOrNotWindCtrlRight) : ['无'],
                  year_profit: statisticalInfo['score'],
                  max_back: statisticalInfo['drop'],
                  win_rate: statisticalInfo['win'],
                  empty_rate: statisticalInfo['none_fate'],
                  modelPara: geneticModel.modelPara
                });
              });
            }
            that.temp = that.geneticModels;
            that.modal = true;
            that.status = true;
            this.loading = false;
          } else if (data.status === 'RUNNING') {
            clearTimeout(that.geneticSetTime);
            that.buttonDisabled = true;
            that.dataText = '数据正在获取中，请您耐心等待...';
            that.geneticModels.splice(0, that.geneticModels.length);
//           获取报告  每隔4s请求一次
            if (data.geneticModels.length !== 0) {
              let statisticalInfo;
              data.geneticModels.forEach(function (geneticModel) {
                resolveIndicator(selectedIndexsTemp, geneticModel.modelPara, conCtrl, symbol);
                statisticalInfo = JSON.parse(geneticModel.report.replace(/\'/g, '\"'));
                that.geneticModels.push({
                  geneticModelId: geneticModel.intelligentModelId.replace(/\-/g, '%'),
                  time: new Date(geneticModel.returnTime).format('yyyy-MM-dd hh:mm:ss'),
                  stock_index: indicatorToDes(classifyIndicator(selectedIndexsTemp, 'A', 'intoMarket'), symbol.andOrNotIntoMarketLeft, symbol.andOrNotIntoMarketRight),
                  out_index: geneticModel.modelPara.indexOf('[SELL]') !== -1 ? indicatorToDes(classifyIndicator(selectedIndexsTemp, 'sell', 'outMarket'), symbol.andOrNotOutMarketLeft, symbol.andOrNotOutMarketRight) : ['无'],
                  second_index: indicatorToDes(classifyIndicator(selectedIndexsTemp, 'B', 'second'), '', ''),
                  wind_index: geneticModel.modelPara.indexOf('[DAN_CON]') !== -1 ? indicatorToDes(classifyIndicator(selectedIndexsTemp, 'C', 'windCtrl'), symbol.andOrNotWindCtrlLeft, symbol.andOrNotWindCtrlRight) : ['无'],
                  year_profit: statisticalInfo['score'],
                  max_back: statisticalInfo['drop'],
                  win_rate: statisticalInfo['win'],
                  empty_rate: statisticalInfo['none_fate'],
                  modelPara: geneticModel.modelPara
                });
              })
            }
            that.modal = true;
            that.geneticSetTime = setTimeout(
              that.getGenetic, 2000);
            this.loading = false;
          } else if (data.status === '') {
//            that.dataText = '暂无数据';
            that.$message.warning('您还没有进行过智能回测，暂无数据');
            that.geneticModels = [];
          } else if (data.status === 'USER_NOT_FOUND') {
            jumpLogin(that);
          } else {
            clearTimeout(that.geneticSetTime);
            that.$message.error(data.message);
          }
        }).catch((e) => {
          that.$message.error('连接服务器异常，请您稍后重试');
        });
      },
      changePage(current) {
        this.currentPage = current;
      },
      exportData() {
//          导出所有数据
        const that = this;
        let a = [];
        this.geneticModels.forEach((item) => {
          a.push({
            time: item.time,
            stock_index: item.stock_index.join('').replace(/\,/g, '、'),
            second_index: item.second_index.join('').replace(/\,/g, '、'),
            wind_index: item.wind_index.join('').replace(/\,/g, '、'),
            out_index: item.out_index.join('').replace(/\,/g, '、'),
            year_profit: item.year_profit,
            max_back: item.max_back,
            win_rate: item.win_rate,
            empty_rate: item.empty_rate,
            hold: that.controller[0],
            test_range: that.controller[1],
            buy_rate: that.controller[2],
            sell_rate: that.controller[3],
            pressure_rate: that.controller[4],
            max_hold: that.controller[5]
          });
        });
        this.$refs.table.exportCsv({
          filename: '智能回测所有数据',
          columns: this.columns,
          data: a
        })
      },
      exportExcelData() {
//        导出为excel表格数据

        let that = this;
        this.tableTemp.splice(0, this.tableTemp.length);
        this.geneticModels.forEach((item) => {
          that.tableTemp.push({
            time: item.time,
            stock_index: item.stock_index.join(''),
            second_index: item.second_index.join(''),
            wind_index: item.wind_index.join(''),
            out_index: item.out_index.join(''),
            year_profit: item.year_profit,
            max_back: item.max_back,
            win_rate: item.win_rate,
            empty_rate: item.empty_rate,
            hold: that.controller[0],
            test_range: that.controller[1],
            buy_rate: that.controller[2],
            sell_rate: that.controller[3],
            pressure_rate: that.controller[4],
            max_hold: that.controller[5]
          });
        });

        this.$nextTick(() => {
          table2excel.transform(this.$refs.table, 'hrefToExportTable', 'excel表格数据');
        });

      },
      sortChange(param) {
        if (param.order === 'descending') {
          this.geneticModels = this.geneticModels.sort(descObj(param.prop));
        } else if (param.order === 'ascending') {
          this.geneticModels = this.geneticModels.sort(ascObj(param.prop));
        } else {
          this.geneticModels = this.geneticModels.sort(descObj('time'));
        }
      },
    }

  }
</script>
<style rel="stylesheet" lang="scss" scoped>
  .show-overflow {
    /*width: 200px;*/
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }

  .text-center {
    text-align: center;
  }

  .above-zero {
    color: #ff0000;
  }

  .equal-zero {
    color: #000;
  }

  .bellow-zero {
    color: #00c261;
  }
</style>
