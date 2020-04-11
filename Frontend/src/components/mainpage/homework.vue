<template>
  <el-card>
    <el-table :data="tableData2"
              @cell-click="contestclick"
              size="small">
      <el-table-column prop="id"
                       label="ID"
                       :width="100"></el-table-column>
      <el-table-column prop="title"
                       label="作业"
                       :width="300"></el-table-column>
      <el-table-column prop="acCount"
                       label="已完成题目数量"></el-table-column>
      <el-table-column prop="proCount"
                       label="题目数量"></el-table-column>
      <el-table-column prop="ChoicefinishCount"
                       label="已完成选择题数量"></el-table-column>
      <el-table-column prop="ChoiceCount"
                       label="选择题数量"></el-table-column>
      <el-table-column prop="begintime"
                       label="布置时间"></el-table-column>
      <el-table-column prop="endtime"
                       label="截止时间"></el-table-column>
      <el-table-column prop="classes"
                       label="班级"></el-table-column>
    </el-table>
  </el-card>

</template>

<script>
import moment from "moment";
export default {

  data () {
    return {
      username: "",
      className: "",
      totalcontest: 0,//总比赛数
      tableData: [],
      tableData2: [],
      loading: true,
      contestIDs: [],//每场比赛的ID
      totProCount: [],//每场比赛有多少题目
      totAcCount: [],//每场比赛有多少AC题目
      totchoiceCount: [],
      totchoicefiniCount: [],
      searchform: {
        type: "",
        title: ""
      }
    };
  },

  methods: {
    contestclick: function (row, column, cell, event) {
      this.$router.push({
        name: "contestdetail",
        params: { contestID: row.id }
      });
    },
  },
  created () {
    this.username = sessionStorage.username;
    this.className = this.$route.query.className;
    this.$axios
      .get("/contestinfo/?type=Homework&classes=" + this.className)
      .then(response => {
        this.tableData = response.data;
        this.totalcontest = response.data.length;
        for (var k = 0; k <= this.totalcontest; k++) {
          this.totAcCount.push(0);
        }

        for (var k1 = 0; k1 <= this.totalcontest; k1++) {
          this.totchoicefiniCount.push(0);
        }
        for (var i = 0; i < response.data.length; i++) {
          this.contestIDs.push(response.data[i].id);
          var bt = this.tableData[i].begintime;
          bt = bt.split("T")
          bt = bt[0];
          this.$set(this.tableData[i], "begintime", bt);
          bt = bt.split("-");
          var et = new Date(bt[0], bt[1], bt[2]);
          et.setSeconds(et.getDate() + response.data[i].lasttime);
          et = moment(et).format("YYYY-MM-DD");
          this.$set(this.tableData[i], "endtime", et);
        }
        this.loading = false;
      }).then(() => {
        (async () => {
          const dataArray = await Promise.all(//使用Promise执行异步请求获取比赛中的题目数量
            this.contestIDs.map(i => this.$axios.get(`/contestproblem/?contestid=${i}`))
          );
          for (const { data } of dataArray) {//验证返回的数据
            if (typeof data === "object" && typeof data.length === "number") {
              this.totProCount.push(JSON.stringify(data.length));
            }
          }
          for (var ii = 0; ii < this.totalcontest; ii++) {//给表格添加题目数量proCount属性
            this.$set(this.tableData[ii], "proCount", this.totProCount[ii]);
          }
        })();
      }).then(() => {
        (async () => {
          const dataArray2 = await Promise.all(//获取用户做题情况
            this.contestIDs.map(i => this.$axios.get(`/contestboard/?contestid=${i}&username=${this.username}&type=1`))
          );
          var lastrank;
          var lastcontest;
          if (dataArray2.length > 0) {
            for (var data2 of dataArray2) {
              var hash = [];
              for (var i = 0; i < data2.data.length; i++) {//统计AC的题目数量
                if (hash.find(val => val.contestid === data2.data[i].contestid && val.problemrank === data2.data[i].problemrank)) { continue; };
                this.totAcCount[data2.data[i].contestid] = parseInt(this.totAcCount[data2.data[i].contestid]) + parseInt(1);
                hash.push(
                  {
                    contestid: data2.data[i].contestid,
                    problemrank: data2.data[i].problemrank
                  });
              }
            }
            for (var iii = 0; iii < this.totalcontest; iii++) {//给表格添加AC题目数量acCount属性
              this.$set(this.tableData[iii], "acCount", this.totAcCount[this.tableData[iii].id]);
            }
          }
          else {
            for (var iii = 0; iii < this.totalcontest; iii++) {
              this.$set(this.tableData[iii], "acCount", this.totAcCount[this.tableData[iii].id]);
            }
          }

        })();
      }).then(() => {
        (async () => {
          const dataArray3 = await Promise.all(
            this.contestIDs.map(i => this.$axios.get(`/contestchoiceproblem/?ContestId=${i}`))
          );
          for (var data3 of dataArray3) {
            this.totchoiceCount.push(data3.data.length);
          }
          for (var j = 0; j < this.totalcontest; j++) {
            this.$set(this.tableData[j], "ChoiceCount", this.totchoiceCount[j]);
          }
        })();
      }).then(() => {
        (async () => {

          const dataArray4 = await Promise.all(
            this.contestIDs.map(i => this.$axios.get(`/conteststudentchoiceanswer/?contestid=${i}&username=${this.username}`))
          );
          for (var data4 of dataArray4) {
            if (data4.data.length > 0) {
              var cnt = 0;
              var stuanswer = data4.data[0].answer;
              for (var kk = 0; kk < stuanswer.length; kk++) {
                if (stuanswer[kk] != 'X') cnt++;
              }
              this.totchoicefiniCount[data4.data[0].contestid] = cnt;
            }
          }
          for (var jj = 0; jj < this.totalcontest; jj++) {
            this.$set(this.tableData[jj], "ChoicefinishCount", this.totchoicefiniCount[this.tableData[jj].id]);
          }
          this.tableData2 = this.tableData;
        })();
      });
  }
}
</script>


<style scoped>
</style>
