<template>
    <div class="year-query-container">
      <h2>核算表系统年度计划</h2>
      <div class="department">
      <ul class="department-list">
        <li>部门:</li>
        <li v-if="threeAreas.length>0">
          <select v-model="Pr0132" @change="getBusinessOfficeRequest(1)" :disabled="selectDisabled[0]">
            <option value=''>请选择</option>
            <option v-for="(item, index) in threeAreas" :key="index" :value="item.F_Id">{{item.F_FullName}}</option>
          </select>
        </li>
        <li v-if="businessOffice.length>0">
          <select v-model="secondSelected" @change="getUnitDataRequest(1)" :disabled="selectDisabled[1]">
            <option value=''>全部</option>
            <option v-for="(item, index) in businessOffice" :key="index" :value="item.F_Id">{{item.F_FullName}}</option>
          </select>
        </li>
        <li v-if="unitData.length>0">
          <select v-model="thirdSelected" @change="getFourthSelectStoreRequest(1)" :disabled="selectDisabled[2]">
            <option value=''>全部</option>
            <option v-for="(item, index) in unitData" :key="index" :value="item.F_Id">{{item.F_FullName}}</option>
          </select>
        </li>
        <li v-if="fourthSelectStore.length>0">
          <select v-model="fourthSelectStoreSelected" :disabled="selectDisabled[3]">
            <option value=''>全部</option>
            <option v-for="(item, index) in fourthSelectStore" :key="index" :value="item.F_Id">{{item.F_FullName}}</option>
          </select>
        </li>
      </ul>
        <ul class="year-select">
          <li>年度:</li>
          <li>
          <select v-model="yearSelected">
            <option v-for="n in 30" :key="n">{{2008 + n}}</option>
          </select>
        </li></ul>
        <div class="button-container">
          <el-button type="primary" @click="auditTableQueryRequest">查询</el-button>
          <el-button type="success" @click="linkToIndex">新增</el-button>
        </div>
      </div>
      <div v-if="queryTableAllData.length>0">
        <table class="show-query-table" border="1">
          <thead>
          <tr>
            <th width="90px">序号</th>
            <th width="180px">年度</th>
            <th width="250px">部门</th>
            <th width="150px">主管</th>
            <th width="150px">状态</th>
            <th width="180px">更新时间</th>
            <th>操作</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="(item, index) in queryTableAllData" :key="index">
            <td>{{index + 1}}</td>
            <td>{{copyYearSelected}}</td>
            <td>{{item.F_FullName}}</td>
            <td>{{item.F_RealName}}</td>
            <td>{{item.Status}}</td>
            <td>{{item.Time}}</td>
            <td><el-button type="primary" icon="el-icon-edit" size="small" :disabled="(item.MPID == '0'?true:false) && (item.IsAccountingTable !== 0)" @click="viewEditor(item)">查看编辑</el-button></td>
          </tr>
          </tbody>
        </table>
        <template v-if="totalCount>10">
          <el-pagination
            class="year-pagination"
            background
            :page-size="10"
            layout="prev, pager, next"
            @current-change="doCurrentChange"
            :total="totalCount">
          </el-pagination>
        </template>
      </div>
    </div>
</template>

<script>
import Vue from 'vue';
import { MessageBox, Pagination, Loading } from 'element-ui';
import api from '@/http/index';
import news from '@/assets/js/notification';

Vue.use(Pagination);
Vue.use(Loading);
Vue.component(MessageBox.name, MessageBox);
Vue.use(api);

export default {
  name: 'QueryContainer',
  data() {
    return {
      // userID: '{8F5FF78A-E0C0-40EE-91ED-88B32A247DE9}',
      // userID: '{85811A95-BB15-4914-8926-82E88F5E6E78}',
      userID: '',
      organizationListData: [],
      threeAreas: [
        { F_Id: '{19BDD92C-55A5-4284-A917-5BCC99FAC4E1}', F_FullName: '上海信义' },
        { F_Id: '{A138EF0C-7CA1-4288-8DCF-24C9583C4EB6}', F_FullName: '苏州信义' },
        { F_Id: '{C278D9F0-20A7-4A0D-8AFE-1E6FF35DF4EE}', F_FullName: '浙江信义' },
      ],
      businessOffice: [],
      unitData: [],
      fourthSelectStore: [],
      Pr0132: '',
      ParentId: '',
      District: '',
      OrganizeId: '',
      FourthParentId: '',
      yearSelected: (new Date().getFullYear() + 1).toString(),
      copyYearSelected: '',
      // Permission: 'S',
      Permission: '',
      selectDisabled: ['', '', '', ''],
      querySelectorArr: [],
      page: 1,
      rows: 10,
      queryTableAllData: '',
      otherParentId: '',
      secondSelected: '',
      thirdSelected: '',
      fourthSelectStoreSelected: '',
      loading: '',
      totalCount: '',
    };
  },
  methods: {
    doCurrentChange(index) {
      this.page = index;
      this.auditTableQueryRequest();
    },
    getUserRequest(permission) {
      console.log({ userID: this.userID });
      this.$api.queryAndAddedUserInfo({ userID: this.userID })
        .then((res) => {
          console.log(JSON.parse(res.data));
          console.log(JSON.parse(res.data).Company);
          this.Pr0132 = JSON.parse(res.data).Company;
          this.District = JSON.parse(res.data).District;
          this.ParentId = JSON.parse(res.data).ParentId;
          this.OrganizeId = JSON.parse(res.data).OrganizeId;
          this.Permission = JSON.parse(res.data).Permission;
          this.getBusinessOfficeRequest(permission);
        })
        .catch((err) => {
          console.log(err);
          this.loading.close();
          news.ElErrorMessage(err);
        });
    },

    getQueryVariable(variable) {
      const query = window.location.search.substring(1);
      const vars = query.split('&');
      for (let i = 0; i < vars.length; i += 1) {
        const pair = vars[i].split('=');
        if (pair[0] === variable) { return pair[1]; }
      }
      return false;
    },

    linkToIndex() {
      window.location = 'yearIndex.html?fromWhichBtn=0';
    },

    getBusinessOfficeRequest(permission) {
      this.businessOffice = [];
      this.unitData = [];
      this.fourthSelectStore = [];
      this.$api.queryAndAddedUserInfo({ Pr0132: this.Pr0132 })
        .then((res) => {
          console.log(JSON.parse(res.data));
          this.businessOffice = JSON.parse(res.data);
          if (permission !== 1) {
            if (this.District !== '' && this.Permission !== 'C') {
              this.secondSelected = this.District;
            } else if (this.District !== '' && this.Permission === 'C') {
              this.secondSelected = this.ParentId;
            } else if (this.District === '' && this.Permission === 'B') {
              this.secondSelected = this.OrganizeId;
            } else {
              this.secondSelected = this.ParentId;
            }
            this.getUnitDataRequest(permission);
          }
          if (permission === 1) {
            this.secondSelected = '';
          }
        })
        .catch((err) => {
          console.log(err);
          this.loading.close();
          news.ElErrorMessage(err);
        });
    },

    getUnitDataRequest(permission) {
      this.unitData = [];
      this.fourthSelectStore = [];
      const unitDataParams = {};
      if (permission !== 1) {
        if (this.District !== '' && this.Permission !== 'C') {
          unitDataParams.ParentId = this.District;
        } else if (this.District !== '' && this.Permission === 'C') {
          unitDataParams.ParentId = this.ParentId;
        } else if (this.District === '' && this.Permission === 'B') {
          unitDataParams.ParentId = this.OrganizeId;
        } else {
          unitDataParams.ParentId = this.ParentId;
        }
      } else {
        unitDataParams.ParentId = this.secondSelected;
      }
      console.log(unitDataParams);
      this.$api.queryAndAddedUserInfo(unitDataParams)
        .then((res) => {
          console.log(JSON.parse(res.data));
          this.unitData = JSON.parse(res.data);
          if (permission !== 1) {
            if (this.District !== '' && this.Permission !== 'C') {
              this.thirdSelected = this.ParentId;
              this.getFourthSelectStoreRequest(permission);
            } else if (this.District !== '' && this.Permission === 'C') {
              this.thirdSelected = this.OrganizeId;
              this.getFourthSelectStoreRequest(permission);
            } else if (this.District === '' && this.Permission === 'B') {
              this.thirdSelected = '';
              this.authorityJudgment();
              this.auditTableQueryRequest();
            } else {
              this.thirdSelected = this.OrganizeId;
              this.getFourthSelectStoreRequest(permission);
            }
          }
          if (permission === 1) {
            this.thirdSelected = '';
          }
        })
        .catch((err) => {
          console.log(err);
          this.loading.close();
          news.ElErrorMessage(err);
        });
    },

    getFourthSelectStoreRequest(permission) {
      this.fourthSelectStore = [];
      const unitDataParams = {};
      if (permission !== 1) {
        if (this.District !== '' && this.Permission !== 'C') {
          unitDataParams.ParentId = this.ParentId;
        } else if (this.District !== '' && this.Permission === 'C') {
          unitDataParams.ParentId = this.OrganizeId;
        } else {
          unitDataParams.ParentId = this.OrganizeId;
        }
      } else {
        unitDataParams.ParentId = this.thirdSelected;
      }
      this.$api.queryAndAddedUserInfo(unitDataParams)
        .then((res) => {
          this.fourthSelectStore = JSON.parse(res.data);
          if (permission !== 1) {
            if (this.District !== '' && this.Permission !== 'C') {
              this.fourthSelectStoreSelected = this.OrganizeId;
            } else if (this.District !== '' && this.Permission === 'C') {
              this.fourthSelectStoreSelected = '';
            } else {
              this.fourthSelectStoreSelected = '';
            }
            this.authorityJudgment();
            this.auditTableQueryRequest();
          }
          if (permission === 1) {
            this.fourthSelectStoreSelected = '';
          }
          console.log(JSON.parse(res.data));
        })
        .catch((err) => {
          console.log(err);
          this.loading.close();
          news.ElErrorMessage(err);
        });
    },

    auditTableQueryRequest() {
      this.querySelectorArr = [];
      this.querySelectorArr.push(this.Pr0132);
      this.querySelectorArr.push(this.secondSelected);
      this.querySelectorArr.push(this.thirdSelected);
      this.querySelectorArr.push(this.fourthSelectStoreSelected);
      const queryArguments = {
        page: this.page,
        rows: this.rows,
        years: this.yearSelected,
      };
      for (let i = 3; i >= 0; i -= 1) {
        if (this.querySelectorArr[i] !== '') {
          switch (true) {
            case i === 3:
              queryArguments.Store = this.fourthSelectStoreSelected;
              break;
            case i === 2:
              queryArguments.department = this.thirdSelected;
              break;
            case i === 1:
              queryArguments.District = this.secondSelected;
              break;
            case i === 0:
              queryArguments.company = this.Pr0132;
              break;
            default:
          }
          break;
        }
      }
      console.log(queryArguments);
      this.$api.queryAndAddedQuery(queryArguments)
        .then((res) => {
          this.copyYearSelected = this.yearSelected;
          console.log(res.data);
          this.queryTableAllData = res.data;
          this.totalCount = res.data[0].TotalCount;
          this.loading.close();
        })
        .catch((err) => {
          console.log(err);
          this.loading.close();
          news.ElErrorMessage(err);
        });
    },

    authorityJudgment() {
      for (let i = 0; i < 4; i += 1) {
        if (this.Permission === 'D') {
          this.selectDisabled[i] = true;
        } else if (this.Permission === 'C') {
          this.selectDisabled[i] = (i !== 3);
        } else if (this.Permission === 'B') {
          this.selectDisabled[i] = i < 2;
        } else if (this.Permission === 'A') {
          this.selectDisabled[i] = i < 1;
        } else if (this.Permission === 'S') {
          this.selectDisabled[i] = false;
        }
      }
    },

    viewEditor(item) {
      if (item.IsAccountingTable === 0) {
        console.log(item.F_FullName);
        sessionStorage.setItem('yearStoreName', item.F_FullName);
        window.location = 'annualDataSummary.html?OrganizeId=' + this.OrganizeId + '&years=' + this.copyYearSelected + '&CreateByUser=' + encodeURI(item.CreateByUser);
      } else if (item.IsAccountingTable === 1) {
        window.location = 'yearIndex.html?CreateByUser=' + encodeURI(item.CreateByUser) + '&fromWhichBtn=1&viewEditorYear=' + this.copyYearSelected;
      }
    },
    setLoading() {
      this.loading = this.$loading({
        lock: false,
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)',
      });
    },
  },
  created() {
    console.log(process.env);
    this.setLoading();
    if (process.env.NODE_ENV === 'development') {
      this.userID = sessionStorage.getItem('userID');
    } else if (process.env.NODE_ENV === 'production') {
      this.userID = decodeURI(this.getQueryVariable('UserID'));
      sessionStorage.setItem('userID', this.userID);
    }
    this.getUserRequest(0);
  },
};
</script>

<style lang="less" scoped>
.year-query-container{
  h2{
    text-align: center;
  }
  .department-list li,.year-select li{
    list-style: none;
    float: left;
    height:30px;
    line-height: 30px;
    select{
      height:30px;
      margin-left:30px;
    }
  }
  .department-list,.year-select{
    height:30px;
  }
  .year-select{
    select{
      width: 78px;
    }
  }
  .addBtn{
    width: 70px;
    height: 40px;
    display: inline-block;
    line-height: 1;
    white-space: nowrap;
    cursor: pointer;
    background-color: #67c23a;
    border-color: #67c23a;
    color: white;
    -webkit-appearance: none;
    text-align: center;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    outline: 0;
    margin: 0;
    -webkit-transition: .1s;
    transition: .1s;
    font-weight: 500;
    -moz-user-select: none;
    -webkit-user-select: none;
    -ms-user-select: none;
    padding: 12px 20px;
    font-size: 14px;
    border-radius: 4px;
    text-decoration: none;
  }
  .department{
    position: relative;
  }
  .button-container{
    position: absolute;
    top:18px;
    left:800px;
    a,button{
      margin-left:30px;
    }
  }
  .show-query-table{
    thead>tr{
      height:40px;
      th{
        box-sizing: border-box;
      }
    }
    width: 100%;
    text-align: center;
    border-collapse: collapse;
  }
  .show-query-table td{
    padding:5px 0;
  }
  .year-pagination{
    text-align: center;
    margin-top: 20px;
  }
}
</style>
