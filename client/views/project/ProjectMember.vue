<template>
  <div class="member-infor">
    <div class="content">
      <div class="search">
        <el-input placeholder="请输入内容" v-model="searchInput">
          <el-select v-model="searchType" slot="prepend" placeholder="请选择">
            <el-option label="用户名" value="username"></el-option>
            <el-option label="联系方式" value="phone"></el-option>
            <el-option label="邮箱" value="email"></el-option>
          </el-select>
          <el-button slot="append" icon="search" @click="searchUser">搜索用户</el-button>
        </el-input>
      </div>

      <div class="member-list">
        <p class="kind">创建者</p>

        <ul class="member">
          <li class="pull-left name-summary-li">{{ project.create_user.username }}</li>
          <li class="pull-left people-li">
            <p class="name">{{ project.create_user.username }}</p>
            <p class="phone">{{ project.create_user.phone }}</p>
          </li>
          <li class="pull-right menu">
            <i class="el-icon-star-on" style="color: #ffb12f"></i>
          </li>
        </ul>

        <p class="kind" v-if="memberList.length > 1">协作成员</p>

        <el-row :gutter="20" style="padding-bottom: 20px;">

          <el-col v-for="member of memberList" :span="12" :xs="12" :sm="8" :md="6" :lg="6" v-if="member.id !== project.create_userId">
            <ul class="member">
              <li class="pull-left name-summary-li">{{ member.username }}</li>
              <li class="pull-left people-li">
                <p class="name">{{ member.username }}</p>
                <p class="phone">{{ member.phone }}</p>
              </li>
              <li class="pull-right menu">
                <i class="el-icon-delete" @click="handleDelete(member)"></i>
              </li>
            </ul>
          </el-col>

        </el-row>

      </div>

      <el-dialog title="搜索结果" :visible.sync="searchResultDialog" size="tiny" class="member-list">
        <ul class="member" v-for="member of searchResult">
          <li class="pull-left name-summary-li">{{ member.username }}</li>
          <li class="pull-left people-li">
            <p class="name">{{ member.username }}</p>
            <p class="phone">{{ member.phone }}</p>
          </li>
          <li class="pull-right menu">
            <el-button type="text" @click="addUserToProject(member)" v-if="!member.exist">加入</el-button>
            <el-button type="text" v-else disabled>已加入</el-button>
          </li>
        </ul>
        <p style="text-align: center" v-show="searchResult.length < 1">抱歉，没找到该用户！</p>
      </el-dialog>

      <el-dialog title="提示" :visible.sync="deleteProjectDialogVisible" size="tiny">
        <p>确认删除协作者<span style="color: #ff4949">【{{ userToDel.username }}】</span>？</p>
        <p>删除后该用户将无法浏览该项目！</p>
        <span slot="footer" class="dialog-footer">
        <el-button @click="deleteProjectDialogVisible = false">取 消</el-button>
        <el-button type="danger" @click="delUserToProject">删 除</el-button>
      </span>
      </el-dialog>

    </div>




  </div>
</template>
<script>
  import ProjectService from '../../service/project.service'
  import Moment from 'moment'
  export default {
    mounted() {
      this.ProjectService = new ProjectService()
      this.reload()
    },
    computed: {
      userInfor() {
        return this.$store.state.userInfor
      },
      project() {
        return this.$store.state.project
      }
    },
    data() {
      return {
        searchInput: '',
        searchType: 'username',
        memberList: [],
        searchResult: [],
        searchResultDialog: false,
        deleteProjectDialogVisible: false,
        userToDel: {}
      }
    },
    methods: {
      searchUser() {
        this.ProjectService.searchUser({
          searchInput: this.searchInput,
          searchType: this.searchType
        }).then(res => {
          if (res.status === 200) {
            this.searchResult = res.data
            const userId = this.memberList.map(s => {
              return s.id
            })
            this.searchResult.forEach(s => {
              s.exist = userId.includes(s.id)
            })
            this.searchResultDialog = true;
          }
        })
      },
      addUserToProject(member) {
        this.ProjectService.addUserToProject({
          userId: member.id,
          projectId: this.project.id
        }).then(res => {
          if (res.status === 200) {
            member.exist = true
            this.reload()
            this.project.userCount += 1
            this.$message({
              type: 'success',
              message: `添加成功！`
            })
          }
        })
      },
      handleDelete(member) {
        this.deleteProjectDialogVisible = true
        this.userToDel = member
        console.log(this.userToDel)
      },
      delUserToProject() {
        this.ProjectService.delUserToProject({
          userId: this.userToDel.id,
          projectId: this.project.id
        }).then(res => {
          if (res.status === 200) {
            this.reload()
            this.project.userCount -= 1
            this.deleteProjectDialogVisible = false
            this.$message({
              type: 'success',
              message: `删除成功！`
            })
          }
        })
      },
      reload() {
        this.ProjectService.findUserByProject({
          projectId: this.project.id
        }).then(res => {
          if (res.status === 200) {
            this.memberList = res.data
          }
        })
      }
    }
  }

</script>
<style>
  .member-infor {
    .el-select .el-input {
      width: 110px;
    }
    .content {
      background: #FFF;
      .search {
        padding: 20px;
        border-bottom: 1px solid #e5e5e5;
      }
      .member-list {
        padding: 0 20px;
        .kind {
          font-size: 14px;
          font-weight: 700;
          height: 36px;
          line-height: 44px;
          text-indent: 0;
        }
        .member {
          width: 200px;
          padding: 10px;
          overflow: hidden;
          margin-right: 10px;
          border: 1px solid #ededed;
          -webkit-border-radius: 3px;
          border-radius: 3px;
          margin-top: 10px;
          height: 43px;
          line-height: 43px;
          .name-summary-li {
            -webkit-border-radius: 23px;
            border-radius: 23px;
            width: 41px;
            height: 41px;
            line-height: 41px;
            background-color: #fafafa;
            text-align: center;
            color: #999;
            border: 1px solid #ddd;
            font-size: 14px;
          }
          .people-li {
            margin-left: 15px;
            p {
              white-space: nowrap;
              overflow: hidden;
              text-overflow: ellipsis;
              height: 21.5px;
              line-height: 21.5px;
              font-size: 14px;
              margin: 0px;
            }
            .phone {
              color: #999;
            }
          }

          .menu i {
            display: block;
            color: #999;
            font-size: 14px;
            cursor: pointer;
          }

          .pull-left {
            float: left;
          }
          .pull-right {
            float: right;
          }
        }
      }
    }
    .search {
      .el-input-group__append {
        color: #20a0ff;
      }
    }
  }

</style>
