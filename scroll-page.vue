<template>
  <div class="store-list-container">
    <!-- <div style="margin-bottom: 50px;">
      <select-scroll
        :list="storeList"
        @handleScroll="handleScroll"
      />
    </div> -->
    <div class="title">店铺列表</div>
    <div class="main-wrapper">
      <div
        class="search d-flex"
      >
        <div
          :span="6"
          class="d-flex align-center search__item"
        >
          <label class="label opacity--85">时间：</label>
          <el-select
            v-model="searchFields.selectDateType"
            placeholder="请选择"
          >
            <el-option
              v-for="item in dateTypeOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            />
          </el-select>
        </div>
        <div
          :span="6"
          class="d-flex align-center search__item"
        >
          <label class="label opacity--85">省份/城市：</label>
          <el-cascader
            v-model="selectArea"
            :options="areaList"
            :props="{ checkStrictly: true, }"
            clearable
            filterable
          />
        </div>
        <div
          :span="6"
          class="d-flex align-center search__item"
        >
          <label class="label opacity--85">店铺：</label>
          <!-- <el-select
            v-model="searchFields.selectedStore"
            placeholder="请选择"
          >
            <el-option
              v-for="item in storeList"
              filterable
              :key="item.value"
              :label="item.label"
              :value="item.value"
            />
          </el-select> -->
          <select-scroll
            :list="list"
            :value.sync="searchFields.selectedStore"
            placeholder="请选择"
            @handleScroll="handleScroll"
          />
        </div>
        <div :span="6">
          <el-button
            type="primary"
            @click="search"
          >查询</el-button>
          <el-button
            type="default"
            @click="reset"
          >重置</el-button>
        </div>
      </div>

      <div class="time-wrapper">
        <label class="label">数据时间：</label>
        <span>{{ `（${dataTime.type}）${dataTime.time}` }}</span>
      </div>
      <div class="checkbox-wrapper d-flex">
        <label class="label">可选表格字段：</label>
        <el-checkbox-group v-model="checkColumns">
          <el-checkbox
            :label="item.prop"
            v-for="(item , index) in columnsOption"
            :key="index"
          >
            {{ item.label }}
          </el-checkbox>
        </el-checkbox-group>
      </div>
      <div class="table-wrapper">
        <el-table
          :data="tableData"
          ref='table'
        >
          <el-table-column
            prop="marketName"
            label="店铺名称"
            width="180"
          />
          <template
            v-for="(colum) in columnsOption"
          >
            <el-table-column
              :key="colum.prop"
              :prop="colum.prop"
              :label="colum.label"
              :width="colum.width || 120"
              v-if="checkColumns.find(c => c === colum.prop)"
            />
          </template>
          <el-table-column
            label="操作"
            :min-width="100"
          >
            <template slot-scope="scope">
              <el-button
                type="text"
                @click="goStore(scope.row)"
              >查看店铺</el-button>
            </template>
          </el-table-column>
        </el-table>
        <div class="pagination-wrapper">
          <el-pagination
            layout="prev, pager, next"
            :total="total"
            :current-page.sync="currentPage"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex';
import SelectScroll from '@/components/select-scroll';

export default {
  data() {
    return {
      dateTypeOptions: this.$constants.dateTypeOptions,
      checkColumns: ['totalFee', 'totalNum', 'price', 'guestNum', 'userNum', 'customerRatio', 'memberNum', 'memberRatio', 'purchase', 'turnover', 'firstGood', 'firstFee', 'secondGood', 'secondFee', 'thirdGood', 'thirdFee'],
      columnsOption: [{
        prop: 'totalFee',
        label: '销售额',
      }, {
        prop: 'totalNum',
        label: '销售量',
      }, {
        prop: 'price',
        label: '客单价',
      }, {
        prop: 'guestNum',
        label: '客件数',
      }, {
        prop: 'userNum',
        label: '老客数',
      }, {
        prop: 'customerRatio',
        label: '老客率',
      }, {
        prop: 'memberNum',
        label: '会员数',
      }, {
        prop: 'memberRatio',
        label: '会员率',
      }, {
        prop: 'purchase',
        label: '回购周期',
        width: 120,
      }, {
        prop: 'turnover',
        label: '流失周期',
        width: 120,
      }, {
        prop: 'firstGood',
        label: 'TOP1品类',
        width: 120,
      },
      {
        prop: 'firstFee',
        label: 'TOP1品类销售额',
        width: 180,
      },
      {
        prop: 'secondGood',
        label: 'TOP2品类',
        width: 120,
      }, {
        prop: 'secondFee',
        label: 'TOP2品类销售额',
        width: 180,
      }, {
        prop: 'thirdGood',
        label: 'TOP3品类',
        width: 120,
      }, {
        prop: 'thirdFee',
        label: 'TOP3品类销售额',
        width: 180,
      }],
      tableData: [],
      total: 0,
      pageNum: 1,
      pageSize: 10,
      currentPage: 1,
      storeList: [],
      searchFields: {
        selectDateType: 0,
        selectArea: [],
        selectedStore: '',
      },
      storeTotalSize: 0,
      pageLimit: 3,
      pageList: [1],
      pageIndex: 1,
      maxPageNum: 1,
      pageSizeOfList: 20,
      list: [],
      value: '',
      scrollObj: null,
    };
  },
  computed: {
    ...mapState([
      'areaList',
    ]),
    selectArea: {
      get() {
        return this.searchFields.selectArea;
      },
      set(v) {
        this.searchFields.selectArea = v;
      },
    },
    dataTime() {
      const type = this.dateTypeOptions.find(d => d.value === this.searchFields.selectDateType).label;
      const lastWeek = this.$utils.lastWeek(new Date(), true);
      const lastWeekP = this.$utils.lastWeek(new Date());
      const lastMonth = this.$utils.lastMonth();
      const time = {
        0: this.$utils.getYesterday(),
        1: `${lastWeek.begin} ~ ${lastWeek.end}`,
        2: `${lastMonth.begin} ~ ${lastMonth.end}`,
      };
      const paramsTime = {
        0: this.$utils.getYesterday(),
        1: `${lastWeekP.begin}_${lastWeekP.end}`,
        2: `${lastMonth.year}${lastMonth.month}`,
      };
      return {
        type,
        time: time[this.searchFields.selectDateType],
        paramsTime: paramsTime[this.searchFields.selectDateType],
      };
    },
  },
  created() {
    this.initStoreList();
  },
  watch: {
    selectArea() {
      this.searchFields.selectedStore = '';
      this.initStoreList();
    },
    checkColumns() {
      // this.$refs.table.doLayout();
    },
    currentPage() {
      this.getStoreInfoList();
    },
  },
  methods: {
    search() {
      this.getStoreInfoList();
    },
    reset() {
      this.searchFields = {
        selectDateType: 0,
        selectArea: [],
        selectedStore: '',
      };
      this.getStoreInfoList();
    },
    goStore(s) {
      this.$router.push(`/operation-insight?marketId=${s.marketid}&&saasid=${s.saasid}`);
    },
    async handleScroll(v) {
      this.scrollObj = v;
      // console.log(this.pageList);
      // console.log(v, this.maxPageNum, this.pageIndex, '---------');
      if (v.direction === 'bottom' && this.pageIndex < this.maxPageNum) {
        this.pageIndex += 1;
        if (this.pageList.length > this.pageLimit) {
          this.pageList.shift();
          this.list.splice(0, this.pageSize);
        }
        this.pageList.push(this.pageIndex);
        await this.getMarket();
        this.list = [...this.list, ...this.storeList];
        v.el.scrollTop = v.middlePosition;
      } else if (v.direction === 'top' && this.pageList[0] > 1) {
        // console.log(this.pageList[0], '----------');
        // // 向上加载数据  判断是否已经到达第一页
        this.pageIndex = this.pageList[0] - 1;
        this.pageList.pop();
        this.pageList = [this.pageIndex, ...this.pageList];
        // console.log('rrrrr', this.pageList);
        this.list.splice(-this.pageSize, this.pageSize);
        await this.getMarket();
        this.list = [...this.storeList, ...this.list];
        v.el.scrollTop = v.middlePosition;
      }
    },
    handleChnage(v) {
      // console.log(v, 'value-changed');
      this.searchFields.selectedStore = v;
    },
    async initStoreList() {
      this.pageIndex = 1;
      this.pageList = [1];
      await this.getMarket();
      this.list = this.storeList;
    },
    async getMarket(pageSize = this.pageSize) {
      // console.log(this.pageIndex, 'initStoreList');
      const params = {
        pageNum: this.pageIndex,
        pageSize,
        params: {
          ...this.getCityProvince(),
        },
      };
      try {
        const data = await this.$api.storeList.getPageMarket(params);
        this.storeList = ((data && data.list) || []).map(s => ({
          value: s.marketCode,
          label: s.marketName,
        }));
        this.storeTotalSize = data.totalSize;
        this.maxPageNum = Math.ceil(this.storeTotalSize / this.pageSize);
      } catch {
        this.storeList = [];
      }
      return this.storeList;
    },
    async getStoreInfoList() {
      const params = {
        pageNum: this.currentPage,
        pageSize: this.pageSize,
        params: {
          dateType: this.searchFields.selectDateType,
          dt: this.dataTime.paramsTime,
          marketId: this.searchFields.selectedStore || 'all',
          ...this.getCityProvince(),
        },
      };
      try {
        const data = await this.$api.storeList.storeInfoList(params);
        this.tableData = data.list || [];
        this.total = data.totalSize;
      } catch {
        this.tableData = [];
        this.total = 0;
      }
    },
    getCityProvince() {
      return {
        cityId: this.searchFields.selectArea.length === 2 ? this.searchFields.selectArea[1] : 'all', //  默认为all
        provinceId: this.searchFields.selectArea.length > 0 ? this.searchFields.selectArea[0] : 'all',
      };
    },
  },
  components: {
    SelectScroll,
  },
};

</script>

<style lang="scss">
.store-list-container {
  .el-cascader {
    line-height: 1;
  }

  .el-table{
    //  width:102%!important;
  }


  // 解决动态添加列 页面闪烁的问题
  .el-table .el-table__header th > .cell {
    white-space: nowrap;
  }
}

</style>

<style lang="scss" scoped>
.store-list-container {
  height: calc(100% - 70px);
  background: #fff;

  .title {
    padding: 12px 24px;
    font-size:16px;
    font-weight:500;
    color:rgba(0,0,0,0.85);
    border-bottom: 1px solid rgba(0, 0, 0, 0.09);
  }

  .label {
    white-space: nowrap;
  }

  .main-wrapper {
    padding: 24px;
    background-color: #fff;
  }

  .search {
     margin-bottom: 24px;

     &__item {

       margin-right: 32px;

        &__label {

        }
     }
  }

  .time-wrapper {
    margin-bottom: 15px;
    color: #000;
  }

  .checkbox-wrapper {

  }

  .table-wrapper {
    width: 99%;
    margin-top: 20px;
  }

  .pagination-wrapper {
    margin-bottom: 40px;
    background-color:#fff;
  }


}
</style>
