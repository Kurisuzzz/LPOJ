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
      totalcontest: 0,
      tableData: [],
      tableData2: [],
      loading: true,
      contestIDs: [],
      totProCount: [],
      totAcCount: [],
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
          const dataArray = await Promise.all(
            this.contestIDs.map(i => this.$axios.get(`/contestproblem/?contestid=${i}`))
          );
          for (const { data } of dataArray) {
            if (typeof data === "object" && typeof data.length === "number") {
              this.totProCount.push(JSON.stringify(data.length));
            }
          }
          for (var ii = 0; ii < this.totalcontest; ii++) {
            this.$set(this.tableData[ii], "proCount", this.totProCount[ii]);
          }
        })();
      }).then(() => {
        (async () => {
          const dataArray2 = await Promise.all(
            this.contestIDs.map(i => this.$axios.get(`/contestboard/?contestid=${i}&username=${this.username}&type=1`))
          );
          for (const { data2 } of dataArray2) {
            if (typeof data2 === "object" && typeof data2.length === "number") {
              this.totAcCount.push(JSON.stringify(data2.length));
            }
          }
          for (var iii = 0; iii < this.totAcCount; iii++) {
            this.$set(this.tableData[iii], "acCount", this.totAcCount[iii]);
          }
          this.tableData2 = this.tableData;
          console.log(this.tableData2);
        })();
      });


  }

}
</script>


<style scoped>
</style>
