<template>
  <el-tabs type="border-card"
           v-show="canshow">
    <el-row>
      <el-row>
        <el-table :data="tableData"
                  id="out-table"
                  style="width: 100%">
          <el-table-column prop="username"
                           label="用户名"
                           width="180">
          </el-table-column>
          <el-table-column prop="realname"
                           label="真实姓名"
                           width="180">
          </el-table-column>
          <el-table-column prop="number"
                           label="学号"
                           width="180">
          </el-table-column>
          <el-table-column prop="answer"
                           label="学生答案"
                           width="400">
          </el-table-column>
          <el-table-column prop="score"
                           label="分数"
                           width="180">
          </el-table-column>

        </el-table>
      </el-row>

      <el-row>
      </el-row>
      <el-button type="primary"
                 :disabled="!isadmin"
                 @click="exportExcel">导出表格</el-button>
    </el-row>
  </el-tabs>
</template>
<script>


import FileSaver from "file-saver";
import XLSX from "xlsx";
export default {
  name: "givechoiceproblemscore",
  data () {
    return {
      singalscore: "",
      studentcount: "",
      type: 1,
      isadmin: false,
      canshow: false,
      standardanswer: "",
      tableData: [],
      contestid: "",


      scoreform: {
        username: "",
        realname: "",
        number: "",
        contestid: "",
        answer: "",
        score: "",
      }
    }
  },
  methods: {
    exportExcel () {
      /* 从表生成工作簿对象 */
      var wb = XLSX.utils.table_to_book(document.querySelector("#out-table"));
      /* 获取二进制字符串作为输出 */
      var wbout = XLSX.write(wb, {
        bookType: "xlsx",
        bookSST: true,
        type: "array"
      });
      try {
        FileSaver.saveAs(
          //Blob 对象表示一个不可变、原始数据的类文件对象。
          //Blob 表示的不一定是JavaScript原生格式的数据。
          //File 接口基于Blob，继承了 blob 的功能并将其扩展使其支持用户系统上的文件。
          //返回一个新创建的 Blob 对象，其内容由参数中给定的数组串联组成。
          new Blob([wbout], { type: "application/octet-stream" }),
          //设置导出文件名称
          "ChoiceProblemScores.xlsx"
        );
      } catch (e) {
        if (typeof console !== "undefined") console.log(e, wbout);
      }
      return wbout;
    },
  },
  created () {
    this.type = sessionStorage.type;
    console.log(this.type);
    if (this.type != 2 && this.type != 3) {
      this.$message.error("非法访问！");
      this.canshow = false;
      return;
    }
    this.canshow = true;
    if (this.type == 3) {
      this.isadmin = true;
    }
    this.contestid = this.$route.query.contestid;
    console.log(this.contestid);
    this.$axios.get("/conteststudentchoiceanswer/?contestid=" + this.contestid)
      .then(response => {
        this.tableData = response.data;
        this.studentcount = response.data.length;
      })
  },
}
</script>
<style scoped>
</style>
