<template>
    <div class="container">
        <el-row>
            <el-alert
              title="逾期列表展示"
              :closable="false"
              type="info">
            </el-alert>           
        </el-row>  
        <el-tabs v-model="activeName" @tab-click="handleClick" type="border-card" v-loading="lodings" class="m20">
            <el-tab-pane label="未分配" name="first">
        <el-row class="m20" >
       
            <el-col  :span="24" class="col-flex-end">
                    <el-button-group>
                      <el-button :type="execeedtimeType==0?'info':''" @click="changeExeceedtimeType(0)">重置</el-button>
                
                      <el-button :type="execeedtimeType==1?'primary':''" @click="changeExeceedtimeType(1)">M1</el-button>
                      <el-button :type="execeedtimeType==2?'success':''" @click="changeExeceedtimeType(2)">M2</el-button>
                      <el-button :type="execeedtimeType==3?'warning':''" @click="changeExeceedtimeType(3)">M3</el-button>
                      <el-button :type="execeedtimeType==4?'danger':''" @click="changeExeceedtimeType(4)">M3+</el-button>
                    </el-button-group>    
                    <div class="l20">
                        <el-input
                        style="padding:0px 10px 0px 0px"
                          placeholder="请输入手机号码"
                          v-model="search.phoneNumber"
                          clearable>
                        </el-input> 
                    </div>                                  
                    <el-date-picker
                    style="width:340px"
                    class="l20"
                      v-model="search.time"
                      type="daterange"
                      value-format="yyyy-MM-dd"
                      range-separator="至"
                      start-placeholder="开始日期"
                      end-placeholder="结束日期">
                    </el-date-picker>                
                    <el-button @click="handleSearch" class="l20" style="margin-left:20px" icon="el-icon-search"  type="success" circle></el-button>                                                                  
            </el-col>             
        </el-row> 
   
        <el-table
            :data="tableData"  
            border  
            ref="multipleTable" 
            tooltip-effect="dark"
            style="width: 100%;font-weight:bold;"
            class="m20"
            @selection-change="handleSelectionChange"
          >
            <el-table-column
              type="selection"
              align="center"
              width="55">
            </el-table-column>        
            <el-table-column prop="id" label="订单号" align="center" width="100"  sortable></el-table-column>
            <el-table-column prop="userName" label="姓名" align="center" ></el-table-column>
            <el-table-column prop="mobile" label="手机号" align="center" ></el-table-column>
            <el-table-column prop="idNo" label="身份证号" align="center" width="200"></el-table-column>
            <el-table-column prop="applyTime" label="申请时间" align="center" width="180" sortable>
                <template slot-scope="scope">
                    {{scope.row.applyTime|dateServer}}
                </template>
            </el-table-column>
            <el-table-column prop="approveAmt" label="审核金额" align="center" sortable width="100"></el-table-column>
            <el-table-column prop="collectorUserName" label="催收员" align="center"  >
                <template slot-scope="scope">
                    {{scope.row.collectorUserName?scope.row.collectorUserName:'暂无催收员'}}
                </template>
            </el-table-column>            
            <el-table-column type="expand" label="更多详情" width="80" >
              <template slot-scope="props" >
                <el-alert
                  title="提现情况"
                  type="success"
                  :closable="false"
                  center
                  >
                </el-alert>
                <el-table
                  :data="props.row.detail.withdraws?props.row.detail.withdraws:[]"
                  border 
                  style="width: 100%;font-weight:bold;"
                  >
                      <el-table-column  prop="id" label="等级" align="center" width="100">
                        <template slot-scope="scope">
                            <el-tag
                                :type="(scope.row.overdue===null||scope.row.overdue.overdueDay===0||scope.row.overdue.overdueDay===null)?'success':scope.row.overdue.overdueDay <  11 ?'primary':scope.row.status < 21?'info':scope.row.status < 31?'warning':'danger'"
                            >
                            {{(scope.row.overdue===null||scope.row.overdue.overdueDay===0||scope.row.overdue.overdueDay===null)?'无':scope.row.overdue.overdueDay <  11 ?'M1':scope.row.status < 21?'M2':scope.row.status < 31?'M3':'M3+'}}
                            </el-tag>
                        </template> 
                      </el-table-column>
                      <el-table-column prop="id" label="id" align="center" sortable></el-table-column>
                      <el-table-column prop="withdrawMoney" label="实际到账金额" align="center" sortable></el-table-column>
                    <el-table-column prop="overdue.returnMoney" label="本息金额" align="center" ></el-table-column>   
                      <el-table-column prop="returnMoney" label="总金额" align="center" >
                        <template slot-scope="scope">
                            {{parseInt(scope.row.overdue.returnMoney+scope.row.overdue.penalty_money+scope.row.overdue.lateFee+scope.row.servicesMoney)}}
                        </template>                           
                      </el-table-column>                                      
                      <el-table-column prop="borrowTime" label="提现时间" align="center" sortable>
                        <template slot-scope="scope">
                            {{scope.row.creatime|dateServer}}
                        </template>                          
                      </el-table-column>
                      <el-table-column prop="status" label="是否逾期" align="center" 
                         :filters="[{ value: 0, text: '放款中 ' }, { value: 1, text: '放款成功' }, { value: 2, text: '逾期' }, { value: 3, text: '还款成功' }, { value: 4, text: '放款失败' }, { value: 5, text: '还款中' }, { value: 6, text: '还款失败' }]"
                         :filter-method="filterStauts"                      
                      >
                        <template slot-scope="scope">
                            <el-tag
                                :type="scope.row.status===0?'':scope.row.status===1?'success':scope.row.status===2?'danger':scope.row.status===4?'success':scope.row.status===4?'info':scope.row.status===5?'':'warning'"
                            >{{scope.row.status===0?'放款中':scope.row.status===1?'放款成功':scope.row.status===2?'逾期':scope.row.status===3?'还款成功':scope.row.status===4?'放款失败':scope.row.status===5?'还款中':'还款失败'}}</el-tag>
                        </template>                         
                      </el-table-column>
                                             <el-table-column prop="servicesMoney" label="服务费" align="center" sortable></el-table-column>   
                        <el-table-column prop="overdue.penalty_money" label="逾期服务费" align="center" sortable></el-table-column> 
                      <el-table-column  prop="overdue.overdueDay" label="逾期天数" align="center" sortable></el-table-column>
                      <el-table-column  prop="overdue.lateFee" label="违约金" align="center" sortable></el-table-column>
                      <el-table-column prop="servicesMoneyStatus" label="服务费还款情况" align="center" 
                      >
                        <template slot-scope="scope">
                            <el-tag
                                :type="scope.row.servicesMoneyStatus===0?'danger':'success'"
                            >{{scope.row.servicesMoneyStatus===0?'未还':scope.row.servicesMoneyStatus===1?'已还':''}}</el-tag>
                        </template>                         
                      </el-table-column>                         
                </el-table>
                <el-alert
                  title="催收情况"
                  type="success"
                  :closable="false"
                  center
                  class="m20"
                  >
                </el-alert>                
                <el-table
                  :data="props.row.detail.loanCollectionRecords?props.row.detail.loanCollectionRecords:[]"
                  border 
                      highlight-current-row style="width: 100%;font-weight:bold"
                  >
                      <el-table-column prop="id" fixed  label="id" align="center" ></el-table-column>
                      <el-table-column prop="operatorName" label="催收员姓名" align="center" ></el-table-column>
                      <el-table-column prop="remark" label="描述（备注）" align="center" ></el-table-column>
                      <el-table-column prop="result" label="结果" align="center" >
                        <template slot-scope="scope">
                            <el-tag
                                :type="scope.row.status===1?'':scope.row.status===2?'danger':'success'"
                            >{{scope.row.status===1?'成功':scope.row.status===2?'失败':'进行中 '}}</el-tag>
                        </template>                         
                      </el-table-column>
                      <el-table-column prop="type" label="催收方式" align="center" >
                        <template slot-scope="scope">
                            <el-tag
                                :type="scope.row.status===1?'':scope.row.status===2?'danger':'success'"
                            >{{scope.row.status===1?'电话':scope.row.status===2?'短信':'其他 '}}</el-tag>
                        </template>                         
                      </el-table-column>                      
                </el-table>                
              </template>
            </el-table-column> 
                <el-table-column prop="cz"  align="center" label="操作"   >
                    <template slot-scope="scope">
                    <el-button
                        size="mini"
                        type="success"
                        @click="handleAllocation(scope.$index, scope.row)"
                       >分配</el-button>
                    </template> 
                </el-table-column>            
        </el-table>  
        <el-row class="m20" v-if="total>0">
            <el-button type="primary" style="float:left" @click="handelConfigAll" :disabled="multipleSelection.length==0">批量分配</el-button>
             <div style="float:right">
                 <el-pagination
                   @current-change="handleCurrentChange"
                    @size-change="handleSizeChange"
                   :current-page="npage"
                    :page-sizes="[10, 20, 50, 100, 1000,990000]"
                     :page-size="pagesize"
                   background
                   layout="total,sizes,prev, pager, next,jumper"
                   :total="total">
                 </el-pagination>   
             </div>
        </el-row>
               
            </el-tab-pane>
            <el-tab-pane label="已经分配" name="second">
        <el-row class="m20" >
            <el-col   class="col-flex-end">
                    <el-button-group>
                      <el-button :type="execeedtimeType==0?'info':''" @click="changeExeceedtimeType(0)">重置</el-button>
                             <el-button :type="execeedtimeType==5?'info':''" @click="changeExeceedtimeType(5)">全部</el-button>
                      <el-button :type="execeedtimeType==1?'primary':''" @click="changeExeceedtimeType(1)">M1</el-button>
                      <el-button :type="execeedtimeType==2?'success':''" @click="changeExeceedtimeType(2)">M2</el-button>
                      <el-button :type="execeedtimeType==3?'warning':''" @click="changeExeceedtimeType(3)">M3</el-button>
                      <el-button :type="execeedtimeType==4?'danger':''" @click="changeExeceedtimeType(4)">M3+</el-button>
                    </el-button-group>  
                    <el-select class="l20" v-model="collectorId" placeholder="请选择催收员">
                      <el-option
                        v-for="item in trevewerlist"
                        :key="item.uid"
                        :label="item.username"
                        :value="item.uid">
                      </el-option>
                    </el-select>                        
                    <div class="l20">
                        <el-input
                        style="padding:0px 10px 0px 0px"
                          placeholder="请输入手机号码"
                          v-model="search.phoneNumber"
                          clearable>
                        </el-input> 
                    </div>                                  
                    <el-date-picker
                      style="width:340px"
                      class="l20"
                      v-model="search.time"
                      type="daterange"
                      value-format="yyyy-MM-dd"
                      range-separator="至"
                      start-placeholder="开始日期"
                      end-placeholder="结束日期">
                    </el-date-picker>                
                    <el-button @click="handleSearch" class="l20" style="margin-left:20px" icon="el-icon-search"  type="success" circle></el-button>                                                                  
            </el-col>             
        </el-row> 
         
        <el-table
            :data="tableData"  
            border  
            ref="multipleTable" 
            tooltip-effect="dark"
                highlight-current-row style="width: 100%;font-weight:bold"
            class="m20"
            @selection-change="handleSelectionChange"
          >
      
            <el-table-column
              type="selection"
              width="55">
            </el-table-column>        
       
            <el-table-column prop="id" fixed label="订单号" align="center" width="100" sortable></el-table-column>
            <el-table-column prop="userName" label="姓名" align="center" width="120" ></el-table-column>
            <el-table-column prop="updateTime" label="最后催收更新时间" align="center" width="180" sortable>
                <template slot-scope="scope">
                  <template v-if="scope.row.updateTime">
                      {{scope.row.updateTime|dateServer}}
                  </template>
                   
                </template>
            </el-table-column>  
            <el-table-column prop="applyCollectResultStatus" label="最后催收结果" align="center" min-width="140" >
                        <template slot-scope="scope">
                          <template v-if="scope.row.applyCollectResultStatus">
                            <el-tag
                                :type="scope.row.applyCollectResultStatus===1?'success':scope.row.applyCollectResultStatus===2?'danger':scope.row.applyCollectResultStatus===3?'danger':scope.row.applyCollectResultStatus===4?'danger':scope.row.applyCollectResultStatus===5?'danger':scope.row.applyCollectResultStatus===6?'danger':'info'"
                            >{{scope.row.applyCollectResultStatus===1?'承诺还款':scope.row.applyCollectResultStatus===2?'无人接听':scope.row.applyCollectResultStatus===3?'通话中':scope.row.applyCollectResultStatus===4?'空号':scope.row.applyCollectResultStatus===5?'停机':scope.row.applyCollectResultStatus===6?'来电提醒':scope.row.applyCollectResultStatus===8?'正常接听':scope.row.applyCollectResultStatus===9?'拒绝还款':scope.row.applyCollectResultStatus===10?'短信呼':'其他'}}</el-tag>
                        </template>   
                        </template>   
            </el-table-column>   
            <el-table-column prop="collectorUserName" label="催收员" align="center"  >
                <template slot-scope="scope">
                    {{scope.row.collectorUserName?scope.row.collectorUserName:'暂无催收员'}}
                </template>
            </el-table-column> 
            <el-table-column prop="collectorId" label="催收员ID" align="center" width="100" >
            </el-table-column>                                  
            <el-table-column prop="mobile" label="手机号" align="center"  width="120"></el-table-column>
            <el-table-column prop="idNo" label="身份证号" align="center" width="200"></el-table-column>
            <el-table-column prop="applyTime" label="申请时间" align="center" width="180" sortable>
                <template slot-scope="scope">
                    {{scope.row.applyTime|dateServer}}
                </template>
            </el-table-column>
            <el-table-column prop="approveAmt" label="审核金额" align="center" sortable width="100"></el-table-column>
                       
            <el-table-column type="expand" label="更多详情" width="80" >
              <template slot-scope="props" >
                <el-alert
                  title="提现情况"
                  type="success"
                  :closable="false"
                  center
                  >
                </el-alert>
                <el-table
                  :data="props.row.detail.withdraws?props.row.detail.withdraws:[]"
                  border  
                      highlight-current-row style="width: 100%;font-weight:bold"
                  >
                      <el-table-column fixed   prop="id" label="等级" align="center" width="100">
                        <template slot-scope="scope">
                            <el-tag
                                :type="(scope.row.overdue===null||scope.row.overdue.overdueDay===0||scope.row.overdue.overdueDay===null)?'success':scope.row.overdue.overdueDay <  11 ?'primary':scope.row.status < 21?'info':scope.row.status < 31?'warning':'danger'"
                            >
                            {{(scope.row.overdue===null||scope.row.overdue.overdueDay===0||scope.row.overdue.overdueDay===null)?'无':scope.row.overdue.overdueDay <  11 ?'M1':scope.row.status < 21?'M2':scope.row.status < 31?'M3':'M3+'}}
                            </el-tag>
                        </template> 
                      </el-table-column>                  
                      <el-table-column prop="id" label="id" align="center" sortable></el-table-column>
                      <el-table-column prop="withdrawMoney" label="实际到账金额" align="center" sortable></el-table-column>
                     <el-table-column prop="overdue.returnMoney" label="本息金额" align="center" ></el-table-column>   
                      <el-table-column prop="returnMoney" label="总金额" align="center" >
                        <template slot-scope="scope">
                            {{parseInt(scope.row.overdue.returnMoney+scope.row.overdue.penalty_money+scope.row.overdue.lateFee+scope.row.servicesMoney)}}
                        </template>                           
                      </el-table-column>                   
                      <el-table-column prop="borrowTime" label="提现时间" align="center" sortable>
                        <template slot-scope="scope">
                            {{scope.row.creatime|dateServer}}
                        </template>                          
                      </el-table-column>
                      <el-table-column prop="status" label="是否逾期" align="center" 
                         :filters="[{ value: 0, text: '放款中 ' }, { value: 1, text: '放款成功' }, { value: 2, text: '逾期' }, { value: 3, text: '还款成功' }, { value: 4, text: '放款失败' }, { value: 5, text: '还款中' }, { value: 6, text: '还款失败' }]"
                         :filter-method="filterStauts"                      
                      >
                        <template slot-scope="scope">
                            <el-tag
                                :type="scope.row.status===0?'':scope.row.status===1?'success':scope.row.status===2?'danger':scope.row.status===4?'success':scope.row.status===4?'info':scope.row.status===5?'':'warning'"
                            >{{scope.row.status===0?'放款中':scope.row.status===1?'放款成功':scope.row.status===2?'逾期':scope.row.status===3?'还款成功':scope.row.status===4?'放款失败':scope.row.status===5?'还款中':'还款失败'}}</el-tag>
                        </template>                         
                      </el-table-column>
                                             <el-table-column prop="servicesMoney" label="服务费" align="center" sortable></el-table-column>   
                        <el-table-column prop="overdue.penalty_money" label="逾期服务费" align="center" sortable></el-table-column> 
                      <el-table-column  prop="overdue.overdueDay" label="逾期天数" align="center" sortable></el-table-column>
                      <el-table-column  prop="overdue.lateFee" label="违约金" align="center" sortable></el-table-column>
                </el-table>
                <el-alert
                  title="催收情况"
                  type="success"
                  :closable="false"
                  center
                  class="m20"
                  >
                </el-alert>                
                <el-table
                  :data="props.row.detail.loanCollectionRecords?props.row.detail.loanCollectionRecords:[]"
                  border 
                      highlight-current-row style="width: 100%;font-weight:bold"
                  >
                      <el-table-column fixed prop="id" label="id" align="center" ></el-table-column>
                      <el-table-column prop="operatorName" label="催收员姓名" align="center" ></el-table-column>
                      <el-table-column prop="remark" label="描述（备注）" align="center" ></el-table-column>
                      <el-table-column prop="result" label="结果" align="center" >
                        <template slot-scope="scope">
                            <el-tag
                                :type="scope.row.status===1?'':scope.row.status===2?'danger':'success'"
                            >{{scope.row.status===1?'成功':scope.row.status===2?'失败':'进行中 '}}</el-tag>
                        </template>                         
                      </el-table-column>
                      <el-table-column prop="type" label="催收方式" align="center" >
                        <template slot-scope="scope">
                            <el-tag
                                :type="scope.row.status===1?'':scope.row.status===2?'danger':'success'"
                            >{{scope.row.status===1?'电话':scope.row.status===2?'短信':'其他 '}}</el-tag>
                        </template>                         
                      </el-table-column>                      
                </el-table>                
              </template>
            </el-table-column> 
                <el-table-column prop="cz"  align="center" label="操作"   width="100">
                    <template slot-scope="scope">
                    <el-button
                        size="mini"
                        type="success"
                        @click="handleAllocation(scope.$index, scope.row)"
                       >重新分配</el-button>
                    </template> 
                </el-table-column>            
        </el-table>  
        <el-row class="m20" v-if="total>0">
            <el-button type="primary" style="float:left" @click="handelConfigAll" :disabled="multipleSelection.length==0">重新分配</el-button>
             <div style="float:right">
                 <el-pagination
                   @current-change="handleCurrentChange"
                    @size-change="handleSizeChange"
                   :current-page="npage"
                    :page-sizes="[10, 20, 50, 100, 1000,990000]"
                     :page-size="pagesize"
                   background
                   layout="total,sizes,prev, pager, next,jumper"
                   :total="total">
                 </el-pagination>   
             </div>
        </el-row>
            
            </el-tab-pane>
        </el-tabs>      
        <el-dialog
          title="分配催收员"
          :visible.sync="dialogVisible"
          center
          width="40%"
          >
            <div id="elTag">
              <el-alert
                title="订单号"
                type="info"
                :closable="false">
              </el-alert>               
              <el-tag
              v-for="(tem,i) in dynamicTags " :key="i"
              >
               {{tem}}
              </el-tag>
              <el-input
                class="input-new-tag"
                v-if="inputVisible"
                v-model="inputValue"
                ref="saveTagInput"
                size="small"
                @keyup.enter.native="handleInputConfirm"
                @blur="handleInputConfirm"
              >
              </el-input>
              <!-- <el-button v-else class="button-new-tag" size="small" @click="showInput">+ 正确订单号</el-button>   -->
             
            </div>
            <el-row class="m20">
              <el-alert
                title="催收员"
                type="info"
                :closable="false">
              </el-alert>   
            </el-row>
            <el-row class="m20 col-flex" >
              <el-select v-model="trevewer" placeholder="请选择催收员">
                <el-option
                  v-for="item in trevewerlist"
                  :key="item.uid"
                  :label="item.username"
                  :value="item.uid">
                </el-option>
              </el-select>  
              <el-button type="primary" style="margin-left:30px" @click="handleConfig">确认</el-button>
              <el-button type="primary" style="margin-left:30px" @click="dialogVisible=false">取消</el-button>
            </el-row>
        </el-dialog>   
    </div>
</template>

<script>
import { mapGetters } from "vuex";
import {
  getExeceedtimeapplyList,
  httpExeceedtimeapplydetail,
  httpGetrevewerlist,
  execeedtimeDistribute,
  getExeceedtimeapplyListbycollectorR,
  getExeceedtimeapplyListNo,
  httpGxeceedtime_distributeafreshbatch
} from "../../../service/http";
import Timer from "../../../config/timer";
import { timeFormat } from "../../../config/time";
export default {
  name: "credit",
  data() {
    return {
      collectorId: null,
      distributionStatus: 1,
      lodings: false,
      activeName: "first",
      search: {
        time: [],
        order: null,
        orders: [
          { label: "还款中", value: 5 },
          { label: "逾期", value: 6 },
          { label: "完结", value: 7 }
        ],
        tixian: "",
        tixians: [
          { label: "没有体现", value: 0 },
          { label: "没有完全提现", value: 1 },
          { label: "完全提现", value: 2 }
        ],
        phoneNumber: null
      },
      tableData: [],
      loading: false,
      npage: 1,
      pagesize: 10,
      total: 0,
      execeedtimeType: 0,
      dialogVisible: false,
      ordersList: [],
      dynamicTags: [], //订单号
      inputVisible: false,
      inputValue: "",
      trevewerlist: [],
      trevewer: null,
      multipleSelection: [],
      index: 0
    };
  },
  computed: {
    // 使用对象展开运算符将 getter 混入 computed 对象中
    ...mapGetters([
      "loginId"
      // ...
    ])
  },
  methods: {
    getData(
      loginId,
      npage,
      pagesize,
      begainTimeString,
      endTimeString,
      phonenumber,
      execeedtimeType,
      distributionStatus,
      collectorId
    ) {
      this.loading = true;
      let _this = this;
      let u =
        this.activeName == "first"
          ? getExeceedtimeapplyListNo
          : getExeceedtimeapplyList;
      u(
        loginId,
        npage,
        pagesize,
        begainTimeString,
        endTimeString,
        phonenumber,
        execeedtimeType,
        distributionStatus,
        collectorId
      )
        .then(res => {
          let data = res.data;
          if (this.activeName == "first") {
            let tableData = data.list;
            for (let a = 0; a < data.list.length; a++) {
              httpExeceedtimeapplydetail(data.list[a].id)
                .then(re => {
                  tableData[a].detail = re.data;
                })
                .catch();
            }

            _this.tableData = tableData;
            _this.total = data.allsize;
            _this.loading = false;
          } else {
            if (data.code == 200) {
              let tableData = data.data.list;
              for (let a = 0; a < data.data.list.length; a++) {
                httpExeceedtimeapplydetail(data.data.list[a].id)
                  .then(re => {
                    tableData[a].detail = re.data;
                  })
                  .catch();
              }

              _this.tableData = tableData;
              _this.total = data.data.allsize;
              _this.loading = false;
            }
          }
        })
        .catch(err => {
          _this.tableData = [];
          _this.loading = false;
        });
    },
    handleSearch(type = true) {
      if (type) {
        this.npage = 1;
        this.pagesize = 10;
      }
      console.log(this.collectorId);
      if (this.search.time && this.search.time.length) {
        if (this.execeedtimeType == 0) {
          this.collectorId = null;
          this.getData(
            this.loginId,
            this.npage,
            this.pagesize,
            this.search.time[0] + " 00:00:00",
            timeFormat(this.search.time[1], 1) + " 00:00:00",
            this.search.phoneNumber,
            "",
            this.distributionStatus,
            this.collectorId
          );
        } else {
          this.getData(
            this.loginId,
            this.npage,
            this.pagesize,
            this.search.time[0] + " 00:00:00",
            timeFormat(this.search.time[1], 1) + " 00:00:00",
            this.search.phoneNumber,
            this.execeedtimeType == 5 ? 0 : this.execeedtimeType,
            this.distributionStatus,
            this.collectorId
          );
        }
      } else {
        if (this.execeedtimeType == 0) {
          this.collectorId = null;
          this.getData(
            this.loginId,
            this.npage,
            this.pagesize,
            "",
            "",
            this.search.phoneNumber,
            "",
            this.distributionStatus,
            this.collectorId
          );
        } else {
          this.getData(
            this.loginId,
            this.npage,
            this.pagesize,
            "",
            "",
            this.search.phoneNumber,
            this.execeedtimeType == 5 ? 0 : this.execeedtimeType,
            this.distributionStatus,
            this.collectorId
          );
        }
      }
    },
    _httpGxeceedtime_distributeafreshbatch() {},
    handleCurrentChange(val) {
      this.npage = val;
      this.handleSearch(false);
    },
    handleSizeChange(val) {
      this.pagesize = val;
      this.handleSearch(false);
    },
    changeExeceedtimeType(i) {
      this.execeedtimeType = i;
      this.npage = 1;
      this.pagesize = 10;
      this.getData(
        this.loginId,
        this.npage,
        this.pagesize,
        "",
        "",
        "",
        i,
        this.distributionStatus
      );
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    handleAllocation(index, row) {
      this.dialogVisible = true;
      this.trevewer = null;
      this.dynamicTags.length = 0;
      this.dynamicTags.push(row.id);
    },
    handleClose(tag) {
      this.dynamicTags.splice(this.dynamicTags.indexOf(tag), 1);
    },

    showInput() {
      this.inputVisible = true;
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },

    handleInputConfirm() {
      let inputValue = this.inputValue;
      if (inputValue) {
        this.dynamicTags.push(inputValue);
      }
      this.inputVisible = false;
      this.inputValue = "";
    },
    getrevewerlist() {
      httpGetrevewerlist()
        .then(res => {
          let data = res.data;
          this.trevewerlist = data;
        })
        .catch(err => {});
    },
    handleConfig() {
      let _this = this;
      if (this.trevewer) {
        if (this.distributionStatus == 1) {
          execeedtimeDistribute(
            this.dynamicTags.length == 1
              ? this.dynamicTags[0] + ","
              : this.dynamicTags.join(","),
            this.trevewer
          )
            .then(res => {
              if (res.data.code == 200) {
                this.$message({
                  message: "分配成功",
                  type: "success"
                });
                _this.dialogVisible = false;
                _this.getData(
                  this.loginId,
                  this.npage,
                  this.pagesize,
                  "",
                  "",
                  "",
                  "",
                  this.distributionStatus
                );
              }
            })
            .catch(err => {
              this.$message.error("催收员必须存在");
            });
        } else {
          if (this.dynamicTags.length == 1) {
            getExeceedtimeapplyListbycollectorR(
              this.trevewer,
              this.dynamicTags[0]
            )
              .then(res => {
                if (res.data.code == 200) {
                  this.$message({
                    message: "分配成功",
                    type: "success"
                  });
                  _this.dialogVisible = false;
                  _this.getData(
                    this.loginId,
                    this.npage,
                    this.pagesize,
                    "",
                    "",
                    "",
                    "",
                    this.distributionStatus
                  );
                }
              })
              .catch();
          } else {
            httpGxeceedtime_distributeafreshbatch(
              this.dynamicTags,
              this.trevewer
            )
              .then(res => {
                if (res.data.code == 200) {
                  this.$message({
                    message: "分配成功",
                    type: "success"
                  });
                  _this.dialogVisible = false;
                  _this.getData(
                    this.loginId,
                    this.npage,
                    this.pagesize,
                    "",
                    "",
                    "",
                    "",
                    this.distributionStatus
                  );
                }
              })
              .catch();
          }
        }
      } else {
        this.$message.error("催收员必须存在");
      }
    },
    handelConfigAll() {
      this.dynamicTags.length = 0;
      for (var temp of this.multipleSelection) {
        this.dynamicTags.push(temp.id);
      }
      this.dialogVisible = true;
    },
    filterStauts(value, row) {
      return row.status === value;
    },
    handleClick(tab, event) {
      this.execeedtimeType = 0;
      this.multipleSelection.length = 0;
      this.distributionStatus = 0;
      tab.name === "second"
        ? (this.distributionStatus = 2)
        : (this.distributionStatus = 1);
      this.lodings = true;
      this.getData(
        this.loginId,
        this.npage,
        this.pagesize,
        "",
        "",
        "",
        "",
        this.distributionStatus
      );
      setTimeout(() => {
        this.lodings = false;
      }, 500);
    }
  },
  mounted() {
    this.getData(
      this.loginId,
      this.npage,
      this.pagesize,
      "",
      "",
      "",
      "",
      this.distributionStatus
    );
    this.getrevewerlist();
  }
};
</script>
<style>
#elTag .el-tag {
  margin-top: 20px;
}
#elTag .el-tag + .el-tag {
  margin-left: 10px;
}
#elTag .button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
#elTag .input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>

<style scoped>
</style>
