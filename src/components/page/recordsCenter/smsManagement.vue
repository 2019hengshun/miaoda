<template>
    <div class="container">
        <el-row>
            <el-alert
              title="短信列表展示"
              :closable="false"
              type="info">
            </el-alert>           
        </el-row>        
        <el-row class="m20" >
            <el-col   class="col-flex-end">
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
            v-loading="loading"
          >
                      <el-table-column
                 type="index"
                  align="center"
                  fixed
                 width="50">
            </el-table-column>
            <el-table-column prop="realName" label="用户姓名" align="center"  width="120"></el-table-column>
            <el-table-column prop="allOverdueMoney" label="短信类型" align="center"  width="120">
                <template slot-scope="scope">
                            <el-tag
                                :type="scope.row.type==1?'':scope.row.type==2?'danger':scope.row.type==3?'success':scope.row.type==4?'info':''"
                            >{{scope.row.type==1?'注册':scope.row.type==2?"忘记密码":scope.row.type==3?"验证码登录":scope.row.type==4?"通过审核":'电销短信'}}</el-tag>                  
                </template>                
            </el-table-column>
            <el-table-column prop="createTime" label="发送时间" align="center" width="180" sortable>
                <template slot-scope="scope">
                    {{scope.row.createTime|dateServer}}
                </template>
            </el-table-column>
            <!-- <el-table-column prop="borrowDay" label="催回总金额" align="center" sortable >
                <template slot-scope="scope">
                    {{scope.row.collect_money_m1+scope.row.collect_money_m2+scope.row.collect_money_m3+scope.row.collect_money_m4}}
                </template>
            </el-table-column> -->
            <el-table-column prop="phoneNumber" label="电话号码" align="center" sortable width="140"></el-table-column>
            <el-table-column prop="content" label="内容" align="center"  min-width="400">
                        <template slot-scope="scope">
                          <el-tooltip class="item" effect="dark" :content="scope.row.content" placement="top">
                              <span>{{scope.row.content}}</span>
                          </el-tooltip>
                    </template>               
            </el-table-column>
            <el-table-column prop="status" label="状态" align="center"   width="120" >
                <template slot-scope="scope">
                            <el-tag
                                :type="scope.row.status?'':'danger'"
                            >{{scope.row.status?'成功':'失败'}}</el-tag>                  
                </template>              
            </el-table-column>

            <!-- <el-table-column prop="overdue_day" label="逾期天数" align="center" sortable ></el-table-column>
            <el-table-column prop="late_fee" label="逾期金" align="center" sortable ></el-table-column>
            <el-table-column prop="ned_return_money" label="需要还钱数" align="center" sortable ></el-table-column>
                <el-table-column prop="cz"  align="center" label="操作"   >
                    <template slot-scope="scope">
                    <el-button
                        size="mini"
                        type="success"
                        @click="handleAllocation(scope.$index, scope.row)"
                       >还款</el-button>
                    </template> 
                </el-table-column>             -->
        </el-table>  
        <el-row class="m20" v-if="total>0">
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
        <el-dialog
          title="线下还款"
          :visible.sync="dialogVisible"
          center
          width="40%"
          >
        </el-dialog>                  
    </div>
</template>

<script>
import { mapGetters } from "vuex";
import axios from "axios";
import { getSmsrecordlist } from "../../../service/http";
import Timer from "../../../config/timer";
import { timeFormat } from "../../../config/time";
export default {
  name: "credit",
  data() {
    return {
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
      editForm: {
        loanApplyId: 1,
        interestOverdue: 2,
        discountAmt: 3,
        mustPayBackAmt: 4,
        actualPayBackAmt: 5
      },
      fileList2: [],
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
      begainTimeString,
      endTimeString,
      npage,
      pagesize,
      phonenumber
    ) {
      let _this = this;
      this.loading = true;
      getSmsrecordlist(
        loginId,
        begainTimeString,
        endTimeString,
        npage,
        pagesize,
        phonenumber
      )
        .then(res => {
          let data = res.data;
          let tableData = data.list;
          _this.tableData = tableData;
          _this.total = data.allsize;
          _this.loading = false;
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
      if (this.search.time && this.search.time.length) {
        this.getData(
          this.loginId,
          this.search.time[0] + " 00:00:00",
          timeFormat(this.search.time[1], 1) + " 00:00:00",
          this.npage,
          this.pagesize,
          this.search.phoneNumber
        );
      } else {
        this.getData(
          this.loginId,
          "",
          "",
          this.npage,
          this.pagesize,
          this.search.phoneNumber
        );
      }
    },

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
      this.handleSearch();
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    handleAllocation(index, row) {},

    filterStauts(value, row) {
      return row.status === value;
    }
  },
  mounted() {
    this.getData(this.loginId, "", "", this.npage, this.pagesize);
  }
};
</script>


<style scoped>
</style>
