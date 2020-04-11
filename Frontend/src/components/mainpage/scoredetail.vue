<template>
  <el-column>
    <el-row>
      <el-table :data="TableData"
                @cell-click="contestclick"
                size="small">
        <el-table-column prop="id"
                         label="ID"
                         :width="100"></el-table-column>
        <el-table-column prop="title"
                         label="作业/比赛"
                         :width="300"></el-table-column>
        <el-table-column prop="begintime"
                         label="布置时间"></el-table-column>
        <el-table-column prop="endtime"
                         label="截止时间"></el-table-column>
        <el-table-column prop="classes"
                         label="班级"></el-table-column>
      </el-table>
    </el-row>
  </el-column>
</template>

<script>
import moment from "moment";
export default {
  data () {
    return {
      className: "",
      tableData: [],
      TableData: [],
      contestIDs:[],
    }
  },
  methods: {
    contestclick(row) {
      window.open(`/classscoredetail?title=${row.title}&contestid=${row.id}&class=${row.classes}`);
    },
  },
  created () {
    this.className = this.$route.query.className;
    this.$axios
      .get("/contestinfo/?classes=" + this.className)
      .then(response => {
        this.tableData = response.data;
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

        this.TableData = this.tableData;
      })
  },

}
</script>

<style scoped>
</style>
