<template>
  <div class="body">
    <div class="report_body">
      <div class="dashbord"><label>Summary</label></div>
      <el-row :gutter="12">
        <el-col :span="12">
          <div class="grid-content bg-purple-light">
            <div>
              <div class="title_position" v-if="suiteLog.executeType == 1">
                Sequential
              </div>
              <div class="title_position" v-else-if="suiteLog.executeType == 0">
                Parallel
              </div>
              <div class="title_position" v-else>Unknow</div>
              <div class="graph">
                <ve-bar
                  :data="chartSummaryData"
                  :settings="chartSettings"
                  :extend="chartSummaryExtend"
                  height="160px"
                  width="600px"
                  style="padding-left: 100px;"
                ></ve-bar>
              </div>
              <div class="title_position">
                <span>
                  <span style="font-weight:bold;">
                    {{ suiteLog.suiteName }}</span
                  >
                </span>
              </div>
              <div class="title_position">
                <span>
                  <span style="font-weight:bold;">{{ passRate }}</span> pass/run
                  <span style="font-weight:bold;">{{ runRate }}</span> run/total
                </span>
              </div>
            </div>
          </div>
        </el-col>
        <el-col :span="12">
          <div class="grid-content bg-purple ">
            <div>
              <div class="title_position">Test</div>
              <div class="graph">
                <ve-ring
                  :data="chartTestData"
                  :settings="chartSettings"
                  :extend="chartTestExtend"
                  height="160px"
                ></ve-ring>
              </div>
              <div class="title_position">
                <span>
                  <span style="font-weight:bold;">{{
                    suiteLog.totalRunCase
                  }}</span>
                  run
                </span>
              </div>
              <div class="title_position">
                <span>
                  <span style="font-weight:bold;">{{
                    suiteLog.totalSuccess
                  }}</span>
                  pass
                  <span style="font-weight:bold;">{{
                    suiteLog.totalFailed
                  }}</span>
                  failed
                  <span style="font-weight:bold;">{{
                    suiteLog.totalError
                  }}</span>
                  error
                  <span style="font-weight:bold;">{{
                    suiteLog.totalSkip
                  }}</span>
                  skip
                </span>
              </div>
            </div>
          </div>
        </el-col>
      </el-row>
      <el-row :gutter="12">
        <el-col :span="6">
          <el-card shadow="hover">
            <div>By</div>
            <div class="card_content_position">{{ suiteLog.executor }}</div>
          </el-card>
        </el-col>
        <el-col :span="6">
          <el-card shadow="hover">
            <div>Env</div>
            <div class="card_content_position" v-if="suiteLog.runDev == 0">
              DEV
            </div>
            <div class="card_content_position" v-else-if="suiteLog.runDev == 1">
              TEST
            </div>
            <div class="card_content_position" v-else-if="suiteLog.runDev == 2">
              STG
            </div>
            <div class="card_content_position" v-else-if="suiteLog.runDev == 3">
              PROD
            </div>
            <div class="card_content_position" v-else-if="suiteLog.runDev == 4">
              DEBUG
            </div>
            <div class="card_content_position" v-else>Unknow</div>
          </el-card>
        </el-col>
        <el-col :span="6">
          <el-card shadow="hover">
            <div>Time Taken</div>
            <div class="card_content_position">{{ suiteLog.runTime }}ms</div>
          </el-card>
        </el-col>
        <el-col :span="6">
          <el-card shadow="hover">
            <div>Time</div>
            <div class="card_content_position">
              {{ suiteLog.startTime }}~{{ suiteLog.endTime }}
            </div>
          </el-card>
        </el-col>
      </el-row>

      <el-row :gutter="12">
        <el-col :span="12">
          <el-card shadow="hover">
            <div
              class="card_content_position"
              v-if="suiteLog.totalRetry != null"
            >
              <el-badge :value="suiteLog.totalRetry" class="item" :max="99">
                <el-tag effect="plain" size="small">Retry</el-tag>
              </el-badge>
            </div>
            <div>
              <el-table
                :data="summaryData"
                stripe
                highlight-current-row
                style="width: 100%; font-size: 13px;"
              >
                <el-table-column
                  property="projectName"
                  label="Project"
                  min-width="30%"
                ></el-table-column>
                <el-table-column
                  property="moduleName"
                  label="Module"
                  min-width="30%"
                ></el-table-column>
                <el-table-column
                  property="pass"
                  label="Pass"
                  min-width="13%"
                ></el-table-column>
                <el-table-column
                  property="failure"
                  label="Failed"
                  min-width="13%"
                ></el-table-column>
                <el-table-column
                  property="error"
                  label="Error"
                  min-width="13%"
                ></el-table-column>
              </el-table>
            </div>
          </el-card>
        </el-col>

        <el-col :span="12">
          <div class="grid-content bg-purple ">
            <div>
              <div class="title_position">Assert</div>
              <div class="graph">
                <ve-ring
                  :data="chartAssertData"
                  :settings="chartSettings"
                  :extend="chartTestExtend"
                  height="160px"
                ></ve-ring>
              </div>
              <div class="title_position">
                <span>
                  <span style="font-weight:bold;">{{
                    assertGroup.pass + assertGroup.failed + assertGroup.error
                  }}</span>
                  run
                </span>
              </div>
              <div class="title_position">
                <span>
                  <span style="font-weight:bold;">{{ assertGroup.pass }}</span>
                  pass
                  <span style="font-weight:bold;">{{
                    assertGroup.failed
                  }}</span>
                  failed
                  <span style="font-weight:bold;">{{ assertGroup.error }}</span>
                  error
                </span>
              </div>
            </div>
          </div>
        </el-col>
      </el-row>
    </div>
  </div>
</template>
<script>
import {
  findIfSuiteLogByNo,
  findSuiteLogSummary,
  findSuiteReportAssert
} from "@/api/getData";
export default {
  data() {
    return {
      suiteLogNo: this.$route.query.suiteLogNo,
      suiteLog: {},
      assertGroup: {},
      summaryData: [],
      passRate: "",
      runRate: "",

      chartSettings: {
        radius: ["50px", "40px"],
        label: {
          normal: {
            show: false,
            position: "inside"
          }
        }
      },
      chartTestExtend: {
        series: {
          center: ["50%", "60%"]
        },
        color: ["#67C23A", "#E6A23C", "#F56C6C", "#909399"]
      },
      chartTestData: {
        columns: ["type", "count"],
        rows: []
      },
      chartAssertData: {
        columns: ["type", "count"],
        rows: []
      },
      chartSummaryData: {
        // 饼图用
        // columns: ['type', 'count'],
        // 条形图用
        columns: ["type", "total", "run", "skip"],
        rows: []
      },
      chartSummaryExtend: {
        series: {
          center: ["50%", "60%"]
        }
      }
    };
  },
  components: {},
  mounted() {
    this.findIfSuiteLogByNo();
    this.findSuiteLogSummary();
    this.findSuiteReportAssert();
  },
  methods: {
    async findIfSuiteLogByNo() {
      const res = await findIfSuiteLogByNo(this.suiteLogNo);
      if (res.code == 200) {
        this.suiteLog = res.data;
        this.passRate =
          (
            (this.suiteLog.totalSuccess / this.suiteLog.totalRunCase) *
            100
          ).toFixed(2) + "%";
        this.runRate =
          (
            (this.suiteLog.totalRunCase / this.suiteLog.totalCase) *
            100
          ).toFixed(2) + "%";
      }
      this.chartTestData.rows.push({
        type: "pass",
        count: res.data.totalSuccess
      });
      this.chartTestData.rows.push({
        type: "failed",
        count: res.data.totalFailed
      });
      this.chartTestData.rows.push({
        type: "error",
        count: res.data.totalError
      });
      this.chartTestData.rows.push({ type: "skip", count: res.data.totalSkip });

      // 饼图用
      // this.chartSummaryData.rows.push({"type":"run", "count":res.data.totalRunCase})
      // this.chartSummaryData.rows.push({"type":"total", "count":res.data.totalCase})

      // 条形图用
      this.chartSummaryData.rows.push({
        type: "",
        total: res.data.totalRunCase + res.data.totalSkip,
        run: res.data.totalRunCase,
        skip: res.data.totalSkip
      });
    },
    async findSuiteLogSummary() {
      const res = await findSuiteLogSummary(this.suiteLogNo);
      if (res.code == 200) {
        this.summaryData = res.data;
      } else {
        this.$message({
          type: "error",
          center: true,
          message: res.msg
        });
      }
    },
    async findSuiteReportAssert() {
      const res = await findSuiteReportAssert(this.suiteLogNo);
      this.assertGroup = res.data;
      this.chartAssertData.rows.push({ type: "pass", count: res.data.pass });
      this.chartAssertData.rows.push({
        type: "failed",
        count: res.data.failed
      });
      this.chartAssertData.rows.push({ type: "error", count: res.data.error });
    }
  }
};
</script>

<style lang="less">
@import "../style/mixin";
.report_body {
  font-size: 14px;
  background-color: #f7f7f7;
}
.title_position {
  padding-left: 15px;
  padding-top: 15px;
}
.card_content_position {
  font-size: 13px;
  text-align: right;
}
.graph {
  max-height: 300px;
}
.el-row {
  margin-bottom: 20px;
  &:last-child {
    margin-bottom: 0;
  }
}
.el-col {
  border-radius: 0px;
}
.bg-purple {
  background: #ffffff;
}
.bg-purple-light {
  background: #ffffff;
}
.grid-content {
  border-radius: 3px;
  min-height: 36px;
}
.row-bg {
  padding: 0;
  background-color: #f9fafc;
}
.dashbord {
  padding-left: 15px;
  padding-top: 15px;
  height: 40px;
  font-size: 20px;
}
.item {
  margin-top: 10px;
  padding-bottom: 10px;
}
// .left-color{
//   margin: 0 auto;
//   font-weight: 600;
//   font-size: 12px;
//   color: #F5F5F5;
//   background-color: #1565C0
// }
</style>
