<template>
  <el-column>
    <el-row>
      <center>
        <h3>{{title}}</h3>
        <br>
        <h4>{{classes}}</h4>
      </center>
    </el-row>
    <el-row>

      <el-table :data="TableData"
                style="width: 100%">
        <el-table-column prop="studentUserName"
                         label="用户名"
                         width="150"></el-table-column>
        <el-table-column prop="studentRealName"
                         label="真实姓名"
                         width="150"></el-table-column>
        <el-table-column prop="studentNumber"
                         label="学号"
                         width="150"></el-table-column>
        <el-table-column prop="acCount"
                         label="已完成题目数量"></el-table-column>
        <el-table-column prop="proCount"
                         label="题目数量"></el-table-column>
        <el-table-column prop="ChoicefinishCount"
                         label="已完成选择题数量"></el-table-column>
        <el-table-column prop="ChoiceCount"
                         label="选择题数量"></el-table-column>
        <el-table-column prop="ChoiceScore"
                         label="选择题分数"></el-table-column>
      </el-table>
    </el-row>
  </el-column>
</template>

<script>
export default {
  data () {
    return {
      classes: "",
      title: "",
      contestid: "",
      searchstudent: "",
      classPeopleCount: "",
      className: "",
      totstudent: "",
      student: [],
      studentRealName: [],
      studentNumber: [],
      studentscore: [],
      studentAnswer: [],
      totAcCount: [],
      TableData: [],
      tableData: [],
    }
  },
  methods: {
  },
  created () {
    this.title = this.$route.query.title;
    this.contestid = this.$route.query.contestid;
    this.classes = this.$route.query.class;
    this.$axios.get(`/classStudent/?className=${this.classes}`)
      .then(response => {
        this.tableData = response.data;
        this.totstudent = response.data.length;
        var cnt = 0;
        for (var data of response.data) {
          this.student.push(data.studentUserName);
          this.studentRealName.push(data.studentRealName);
          this.studentNumber.push(data.studentNumber);
        }
      }).then(() => {
        for (var i = 0; i < this.totstudent; i++) {//给表格添加属性
          this.$set(this.tableData[i], "studentUserName", this.student[i]);
          this.$set(this.tableData[i], "studentRealName", this.studentRealName[i]);
          this.$set(this.tableData[i], "studentNumber", this.studentNumber[i]);
        }
      })
      .then(() => {
        this.$axios.get(`/contestproblem/?contestid=${this.contestid}`)
          .then(response2 => {
            for (var i = 0; i < this.totstudent; i++) {
              this.$set(this.tableData[i], "proCount", response2.data.length);
            }
          })
      }).then(() => {
        this.$axios.get(`/contestchoiceproblem/?ContestId=${this.contestid}`)
          .then(response3 => {
            for (var i = 0; i < this.totstudent; i++) {
              this.$set(this.tableData[i], "ChoiceCount", response3.data.length);
            }
          })
      }).then(() => {
        (async () => {
          const dataArray = await Promise.all(
            this.student.map(i => this.$axios.get(`/conteststudentchoiceanswer/?username=${i}&contestid=${this.contestid}`))
          );
          for (var data of dataArray) {
            if (data.data.length > 0) {
              this.studentAnswer.push(data.data[0].answer);
              this.studentscore.push(data.data[0].score);
            }
            else this.studentscore.push('0');
          }
          for (var j = 0; j < this.totstudent; j++) {
            this.$set(this.tableData[j], "ChoiceScore", this.studentscore[j]);
          }
          for (var j = 0; j < this.totstudent; j++) {
            var cnt = 0;
            var stua = this.studentAnswer[j];
            if (stua) {
              for (var i = 0; i < stua.length; i++) {
                if (stua[i] != "X") cnt++;
              }
            }

            this.$set(this.tableData[j], "ChoicefinishCount", cnt);
          }

        })();
      }).then(() => {
        (async () => {
          const dataArray2 = await Promise.all(
            this.student.map(i => this.$axios.get(`/contestboard/?username=${i}&contestid=${this.contestid}&type=1`))
          );
          for (var data2 of dataArray2) {
            var hash = [];
            var cnt2 = 0;
            for (var i = 0; i < data2.data.length; i++) {//统计AC的题目数量
              if (hash.find(val => val.username === data2.data[i].username && val.problemrank === data2.data[i].problemrank)) { continue; };
              cnt2 = cnt2 + 1;
              hash.push(
                {
                  username: data2.data[i].username,
                  problemrank: data2.data[i].problemrank
                });
            }
            this.totAcCount.push(cnt2);
            for (var i = 0; i < this.totstudent; i++) {
              this.$set(this.tableData[i], "acCount", this.totAcCount[i]);
            }
          }
          this.TableData = this.tableData;
        })();
      });
  },
}
</script>

<style scoped>
</style>
