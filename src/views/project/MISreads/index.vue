<template>
  <div class="MISreads-container">
    <el-row :gutter="15">
      <el-col :xs="24" :sm="24" :md="8" :lg="6" :xl="4">
        <el-card>
          <el-form
            ref="MISReadsForm"
            :model="MISReadsForm"
            :rules="MISReadsFormRules"
            label-width="100px"
            label-position="top"
            @submit.native.prevent
          >
            <el-form-item>
              <el-row type="flex" class="row-bg" justify="space-around">
                <el-col :span="9">
                  <el-button
                    icon="el-icon-delete-solid"
                    type="danger"
                    @click="clearEmpText()"
                    >清空</el-button
                  >
                </el-col>
                <el-col :span="9">
                  <el-button
                    icon="el-icon-plus"
                    type="primary"
                    @click="submitEmpText()"
                    >查询</el-button
                  >
                </el-col>
              </el-row>
            </el-form-item>
            <el-checkbox-group v-model="readItem" size="mini">
              <el-checkbox label="0" checked>OA权限</el-checkbox>
              <el-checkbox label="1" checked>纸质档案</el-checkbox>
              <el-checkbox label="2" checked>功 能 机</el-checkbox>
              <el-checkbox label="3" checked>电脑档案</el-checkbox>
            </el-checkbox-group>
            <el-form-item label="请填入员工信息" prop="empText">
              <el-input
                id="empText"
                v-model="MISReadsForm.empText"
                type="textarea"
              ></el-input>
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>
      <el-col :xs="24" :sm="24" :md="16" :lg="18" :xl="20">
        <el-card>
          <el-tabs type="border-card">
            <el-tab-pane label="OA权限申请记录">
              <el-table
                v-loading="loading.OAMISData"
                :data="OAMISData.data"
                style="width: 100%;"
                border
                stripe
                height="560"
              >
                <el-table-column type="expand">
                  <template slot-scope="scope">
                    <el-form
                      label-position="left"
                      class="demo-table-expand"
                      label-width="150px"
                    >
                      <el-form-item label="所任职务">
                        <span>{{ scope.row.XmlData.text_zw }}</span>
                      </el-form-item>
                      <el-form-item label="联系电话">
                        <span>{{ scope.row.XmlData.text_lxdh }}</span>
                      </el-form-item>
                      <el-form-item label="姓名全拼">
                        <span>{{ scope.row.XmlData.text_xmqp }}</span>
                      </el-form-item>
                      <el-form-item label="登记的邮箱密码">
                        <span>{{ scope.row.XmlData.text_mm }}</span>
                      </el-form-item>
                      <el-form-item label="OA单号">
                        <span
                          ><a @click="dumpOAdoc(scope.row)">{{
                            scope.row.WF_DocNumber
                          }}</a></span
                        >
                      </el-form-item>
                    </el-form>
                  </template>
                </el-table-column>
                <el-table-column type="index" label="#">
                  <template slot-scope="scope">
                    {{
                      (OAMISData.page - 1) * OAMISData.pageSize +
                      scope.$index +
                      1
                    }}
                  </template>
                </el-table-column>
                <el-table-column
                  prop="XmlData.text_gh"
                  label="工号"
                  width="100"
                >
                </el-table-column>
                <el-table-column
                  prop="XmlData.text_xm"
                  label="姓名"
                  width="100"
                >
                </el-table-column>
                <el-table-column
                  prop="XmlData.text_bm"
                  label="部门"
                  width="150"
                >
                </el-table-column>
                <el-table-column
                  prop="XmlData.text_jsjmc"
                  label="电脑编号"
                  width="150"
                >
                </el-table-column>
                <el-table-column prop="powers" label="权限" width="250">
                  <template slot-scope="scope">
                    <el-tag v-if="scope.row.XmlData.sqlb_nb === '内邮Mail权限'"
                      >内邮</el-tag
                    >
                    <el-tag
                      v-if="scope.row.XmlData.sqlb_i === 'Internet权限'"
                      type="danger"
                      >外网</el-tag
                    >
                    <el-tag
                      v-if="scope.row.XmlData.sqlb_wb === '外部Mail权限'"
                      type="info"
                      >外邮</el-tag
                    >
                    <el-tag
                      v-if="
                        scope.row.XmlData.sqlb_wx === '微信' ||
                        String(scope.row.XmlData.text_qt).indexOf('微信') !== -1
                      "
                      type="success"
                      >微信</el-tag
                    >
                  </template>
                </el-table-column>
                <el-table-column
                  show-overflow-tooltip
                  prop="XmlData.text_qt"
                  label="其他说明"
                >
                </el-table-column>
                <el-table-column
                  prop="WF_DocCreated"
                  label="申请时间"
                  width="150"
                >
                </el-table-column>
              </el-table>
              <el-pagination
                :current-page="OAMISData.page"
                :page-sizes="[10, 20, 50, 100]"
                :page-size="OAMISData.pageSize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total.OAMISData"
                @size-change="OAMISDataSizeChange"
                @current-change="OAMISDataCurrentChange"
              >
              </el-pagination>
            </el-tab-pane>
            <el-tab-pane label="纸质档案记录">
              <el-table
                v-loading="loading.filesData"
                :data="filesData.data"
                style="width: 100%;"
                border
                stripe
                height="560"
              >
                <el-table-column type="index" label="#"></el-table-column>
                <el-table-column prop="path" label="匹配记录">
                  <template slot-scope="scope">
                    <a @click="dumpFile(scope.row)">{{ scope.row.name }}</a>
                  </template>
                </el-table-column>
              </el-table>
            </el-tab-pane>
            <el-tab-pane label="功能机领取记录">
              <el-table
                v-loading="loading.phonesData"
                :data="phonesData.data"
                style="width: 100%;"
                border
                stripe
                height="560"
              >
                <el-table-column type="index" label="#"></el-table-column>
                <el-table-column prop="employeeCode" label="工号" width="100">
                </el-table-column>
                <el-table-column prop="employeeName" label="姓名" width="100">
                </el-table-column>
                <el-table-column prop="area" label="所属厂" width="100">
                </el-table-column>
                <el-table-column prop="Department" label="部门" width="150">
                </el-table-column>
                <el-table-column prop="issuePhoneNum" label="长号" width="150">
                </el-table-column>
                <el-table-column prop="issuePhoneSort" label="短号" width="100">
                </el-table-column>
                <el-table-column prop="isReturn" label="状态" width="100">
                  <template slot-scope="scope">
                    <el-tag
                      v-if="scope.row.isReturn === '已发出'"
                      type="danger"
                      >{{ scope.row.isReturn }}</el-tag
                    >
                    <el-tag
                      v-if="scope.row.isReturn === '已退回'"
                      type="success"
                      >{{ scope.row.isReturn }}</el-tag
                    >

                    <el-tag
                      v-if="scope.row.isReturn === '其他'"
                      type="warning"
                      >{{ scope.row.isReturn }}</el-tag
                    >
                  </template>
                </el-table-column>
                <el-table-column
                  show-overflow-tooltip
                  prop="remark"
                  label="备注"
                >
                </el-table-column>
                <el-table-column label="操作" width="150">
                  <template slot-scope="scope">
                    <el-button
                      icon="el-icon-edit"
                      size="mini"
                      @click="updatePhoneRow(scope.row)"
                    ></el-button>
                    <el-button
                      type="danger"
                      icon="el-icon-delete-solid"
                      size="mini"
                      @click="deletePhoneRow(scope.row)"
                    ></el-button>
                  </template>
                </el-table-column>
              </el-table>
              <el-pagination
                :current-page="phonesData.page"
                :page-sizes="[10, 20, 50, 100]"
                :page-size="phonesData.pageSize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total.phonesData"
                @size-change="phonesDataSizeChange"
                @current-change="phonesDataCurrentChange"
              >
              </el-pagination>
            </el-tab-pane>
            <el-tab-pane label="电脑档案记录">
              <el-table
                v-loading="loading.computersData"
                :data="computersData.data"
                border
                style="width: 100%;"
                stripe
                height="560"
              >
                <el-table-column type="expand">
                  <template slot-scope="scope">
                    <el-form
                      label-position="left"
                      class="demo-table-expand"
                      label-width="150px"
                    >
                      <el-form-item label="所在部门">
                        <span>{{ scope.row.section }}</span>
                      </el-form-item>
                      <el-form-item label="所任职务">
                        <span>{{ scope.row.duties }}</span>
                      </el-form-item>
                      <el-form-item label="电脑类型">
                        <span>{{ scope.row.displayer }}</span>
                      </el-form-item>
                      <el-form-item label="电脑品牌">
                        <span>{{ scope.row.master }}</span>
                      </el-form-item>
                      <el-form-item label="处理器">
                        <span>{{ scope.row.cpu }}</span>
                      </el-form-item>
                      <el-form-item label="内存">
                        <span>{{ scope.row.ram }}</span>
                      </el-form-item>
                      <el-form-item label="显示器">
                        <span>{{ scope.row.display }}</span>
                      </el-form-item>
                      <el-form-item label="硬盘">
                        <span>{{ scope.row.disk }}</span>
                      </el-form-item>
                      <el-form-item label="上一任使用人">
                        <span>{{ scope.row.olduser }}</span>
                      </el-form-item>
                      <el-form-item label="建档信息">
                        <span>{{ scope.row.comment }}</span>
                      </el-form-item>
                      <el-form-item label="最后更新日期">
                        <span>{{ scope.row.time }}</span>
                      </el-form-item>
                    </el-form>
                  </template>
                </el-table-column>
                <el-table-column type="index" label="#">
                  <template slot-scope="scope">
                    {{
                      (computersData.page - 1) * computersData.pageSize +
                      scope.$index +
                      1
                    }}
                  </template>
                </el-table-column>
                <el-table-column prop="number" label="电脑编号" width="110">
                </el-table-column>
                <el-table-column
                  prop="ip"
                  label="ip地址"
                  :show-overflow-tooltip="true"
                  width="510"
                >
                  <template slot-scope="scope">
                    <el-tag v-if="scope.row.ip" effect="dark">
                      {{ scope.row.ip }}
                    </el-tag>
                    <el-tag v-if="scope.row.ip2" effect="dark">
                      {{ scope.row.ip2 }}
                    </el-tag>
                    <el-tag v-if="scope.row.ip3" effect="dark">
                      {{ scope.row.ip3 }}
                    </el-tag>
                    <el-tag v-if="scope.row.ip4" effect="dark">
                      {{ scope.row.ip4 }}
                    </el-tag>
                  </template>
                </el-table-column>
                <el-table-column
                  prop="category"
                  label="电脑类型"
                ></el-table-column>
                <el-table-column prop="user" label="用户"></el-table-column>
                <el-table-column prop="userid" label="工号"></el-table-column>
                <el-table-column prop="found" label="建档"></el-table-column>
                <el-table-column prop="amend" label="修订"></el-table-column>
                <el-table-column
                  prop="factory"
                  label="所属厂"
                ></el-table-column>
              </el-table>
              <el-pagination
                :current-page="computersData.page"
                :page-sizes="[10, 20, 50, 100]"
                :page-size="computersData.pageSize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total.computersData"
                @size-change="computersDataSizeChange"
                @current-change="computersDataCurrentChange"
              >
              </el-pagination>
            </el-tab-pane>
          </el-tabs>
        </el-card>
      </el-col>
    </el-row>
    <!-- 图片弹出框 -->
    <el-dialog
      title="文件预览"
      :visible.sync="fileVisible"
      :before-close="fileClose"
      width="80%"
    >
      <div>
        <img :src="fileUrl" />
      </div>
    </el-dialog>
    <!-- 修改功能机记录弹出框 -->
    <el-dialog
      title="修改功能机记录"
      :visible.sync="updatePhoneVisible"
      :before-close="updatePhoneClose"
      width="30%"
    >
      <div>
        <mobile-form
          ref="mobileForm"
          :mobile-data="updataPhoneData"
          :parent="'MISreads'"
          @closedialog="updatePhoneClose"
          @RefreshTable="refreshTable(phonesData)"
        ></mobile-form>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  getEmpData,
  getFilesData,
  getPhonesData,
  getComputersData,
  openFilesUrl,
} from "@/api/MISreads";
import { deletePhonesData } from "@/api/update";
import _ from "lodash";
import mobileForm from "@/components/MIS/mobileForm";
export default {
  name: "MISreads",
  components: {
    mobileForm,
  },
  data() {
    return {
      MISReadsForm: {
        empText: "",
        page: 1,
        pageSize: 100,
      },
      MISReadsFormRules: {
        empText: [
          { required: true, message: "请输入员工信息", trigger: "blur" },
        ],
      },
      // OA权限申请记录
      OAMISData: {},
      // 纸质档案记录
      filesData: {},
      // 功能机领取记录
      phonesData: {},
      // 电脑档案记录
      computersData: {},
      // 分页总量
      total: {
        OAMISData: 0,
        filesData: 0,
        phonesData: 0,
        computersData: 0,
      },
      // 加载动画
      loading: {
        OAMISData: false,
        filesData: false,
        phonesData: false,
        computersData: false,
      },
      // 图片弹出框
      fileVisible: false,
      fileUrl: "",
      // 修改功能机记录弹出框
      updatePhoneVisible: false,
      updataPhoneData: {},
      // 勾选查询
      readItem: [],
    };
  },
  created() {},
  mounted() {},
  methods: {
    // 修改功能机记录 关闭前
    updatePhoneClose() {
      this.updataPhoneData = {};
      this.updatePhoneVisible = false;
    },
    // 更新 功能机发放记录 行
    updatePhoneRow(row) {
      this.updatePhoneVisible = true;
      const data = _.cloneDeep(row);
      this.updataPhoneData = data;
    },
    // 删除 功能机发放记录 行
    deletePhoneRow(row) {
      this.$confirm("此操作将永久删除该记录, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.deletePhonesData(row.id);
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    // 功能机发放记录 分页数量
    phonesDataSizeChange(val) {
      this.getPhonesData({
        empText: this.phonesData.searchHistroy,
        page: this.phonesData.page,
        pageSize: val,
      });
    },
    // 功能机发放记录 当前页
    phonesDataCurrentChange(val) {
      this.getPhonesData({
        empText: this.phonesData.searchHistroy,
        page: val,
        pageSize: this.phonesData.pageSize,
      });
    },
    // 电脑档案记录 分页数量
    computersDataSizeChange(val) {
      this.getComputersData({
        empText: this.computersData.searchHistroy,
        page: this.computersData.page,
        pageSize: val,
      });
    },
    // 电脑档案记录 当前页
    computersDataCurrentChange(val) {
      this.getComputersData({
        empText: this.computersData.searchHistroy,
        page: val,
        pageSize: this.computersData.pageSize,
      });
    },
    // OA权限申请记录 分页数量
    OAMISDataSizeChange(val) {
      this.getEmpData({
        empText: this.OAMISData.searchHistroy,
        page: this.OAMISData.page,
        pageSize: val,
      });
    },
    // OA权限申请记录 当前页
    OAMISDataCurrentChange(val) {
      this.getEmpData({
        empText: this.OAMISData.searchHistroy,
        page: val,
        pageSize: this.OAMISData.pageSize,
      });
    },
    // 打开OA表单
    dumpOAdoc(row) {
      const href =
        "http://172.18.8.20/bpm/linkey_workflow_engine.nsf/workflow_doc?readform&WF_ProcessUNID=" +
        row.WF_ProcessUNID +
        "&WF_DocUNID=" +
        row.WF_DocUNID +
        "&WF_DocAction=read";
      // 新标签页打开
      window.open(href);
    },
    // 打开纸质档案图片
    dumpFile(row) {
      this.fileVisible = true;
      console.log(row);
      this.openFilesUrl(row);
    },
    // 关闭纸质档案
    fileClose() {
      this.fileVisible = false;
      this.fileUrl = "";
    },
    // 清空输入框
    clearEmpText() {
      this.$refs.MISReadsForm.resetFields();
    },
    // 提交查询
    submitEmpText() {
      this.$refs.MISReadsForm.validate(async (validate) => {
        // 空input return
        if (!validate) return false;
        // 深拷贝数据
        const searchData = this.returnSearchData();
        // 执行查询
        const loading = this.$loading({
          lock: true,
          text: "Loading",
          spinner: "el-icon-loading",
          background: "rgba(0, 0, 0, 0.7)",
        });
        if (this.readItem.indexOf("2") > -1) {
          await this.getPhonesData(searchData);
        }
        if (this.readItem.indexOf("0") > -1) {
          await this.getEmpData(searchData);
        }
        if (this.readItem.indexOf("1") > -1) {
          await this.getFilesData(searchData);
        }
        if (this.readItem.indexOf("3") > -1) {
          await this.getComputersData(searchData);
        }

        loading.close();
      });
    },
    // tag标签页旁边的新数据小星星
    newTips(element, status = "") {
      if (status === "delete") {
        $("#" + element + ' [class="newTip"]').remove();
      } else {
        if ($("#" + element + ' [class="newTip"]').length == 0) {
          var html = document.getElementById(element).innerHTML;
          document.getElementById(element).innerHTML =
            html + "<span class='newTip' style='color:red'>*</span>";
        }
      }
    },
    // 处理待提交的表单数据
    returnSearchData(form) {
      // 深拷贝数据
      const searchData = _.cloneDeep(this.MISReadsForm);
      searchData.empText = searchData.empText.split("\n");
      if (form) {
        searchData.page = form.page;
        searchData.pageSize = form.pageSize;
      }

      return searchData;
    },
    // 刷新单个table
    refreshTable(from) {
      let searchData = this.returnSearchData(from);
      this.getPhonesData(searchData);
    },
    /**
     * 网络请求
     */

    // 查询OA权限申请记录
    async getEmpData(data) {
      // 加载动画
      this.loading.OAMISData = true;
      await getEmpData(data)
        .then((res) => {
          // 记录总行数，不然大于1页时无法返回正确的总数
          if (res.page == 1) {
            this.total.OAMISData = res.totalCount;
          }
          // 新查询结果提示
          this.newTips("tab-0");
          // 呈现数据
          this.OAMISData = res;
          // 关闭加载动画
          this.loading.OAMISData = false;
          this.$notify.close();
          // 查询成功提示
          this.$notify.success({
            title: "查询成功",
            message: "OA权限申请记录已成功返回数据",
            duration: 22000,
          });
        })
        .catch((err) => {
          this.newTips("tab-0", "delete");
          this.loading.OAMISData = false;
          this.OAMISData = "";
        });
    },
    // 查询纸质档案记录
    async getFilesData(data) {
      // 加载动画
      this.loading.filesData = true;
      await getFilesData(data)
        .then((res) => {
          // 新查询结果提示
          this.newTips("tab-1");
          this.filesData = res;
          this.loading.filesData = false;
          // 查询成功提示
          this.$notify.success({
            title: "查询成功",
            message: "纸质档案记录已成功返回数据",
            duration: 22000,
          });
        })
        .catch((err) => {
          this.newTips("tab-1", "delete");
          this.loading.filesData = false;
          this.filesData = "";
        });
    },
    // 打开纸质档案图片
    openFilesUrl(data) {
      openFilesUrl(data).then((res) => {
        this.fileUrl = res.data;
      });
    },
    // 查询功能机发放记录
    async getPhonesData(data) {
      // 加载动画
      this.loading.phonesData = true;
      await getPhonesData(data)
        .then((res) => {
          // 记录总行数，不然大于1页时无法返回正确的总数
          if (res.page == 1) {
            this.total.phonesData = res.totalCount;
          }
          // 新查询结果提示
          this.newTips("tab-2");
          this.phonesData = res;
          this.loading.phonesData = false;
          // 查询成功提示
          this.$notify.success({
            title: "查询成功",
            message: "功能机发放记录已成功返回数据",
            duration: 22000,
          });
        })
        .catch((err) => {
          this.newTips("tab-2", "delete");
          this.loading.phonesData = false;
          this.phonesData = "";
        });
    },
    // 删除功能机一行记录
    deletePhonesData(data) {
      deletePhonesData(data).then((res) => {
        if (res.code === 200) {
          this.$message.success(res.msg);
          this.refreshTable(this.phonesData);
        }
      });
    },
    // 查询电脑档案记录
    async getComputersData(data) {
      // 加载动画
      this.loading.computersData = true;
      await getComputersData(data)
        .then((res) => {
          // 记录总行数，不然大于1页时无法返回正确的总数
          if (res.page == 1) {
            this.total.computersData = res.totalCount;
          }
          // 新查询结果提示
          this.newTips("tab-3");
          // 呈现数据
          this.computersData = res;
          // 关闭加载动画
          this.loading.computersData = false;
          // 查询成功提示
          this.$notify.success({
            title: "查询成功",
            message: "电脑档案记录已成功返回数据",
            duration: 22000,
          });
        })
        .catch((err) => {
          this.newTips("tab-3", "delete");
          this.loading.computersData = false;
          this.computersData = "";
        });
    },
  },
};
</script>

<style lang="scss" scoped>
.MISreads-container {
  ::v-deep {
    .el-card__body {
      width: 100%;
      height: 708px;
    }

    #empText {
      width: 100%;
      height: 530px;
    }

    .el-tabs {
      width: 100%;
      height: 100%;
    }

    .demo-table-expand {
      font-size: 0;
    }

    .demo-table-expand label {
      width: 90px;
      color: #99a9bf;
    }

    .demo-table-expand .el-form-item {
      width: 50%;
      margin-right: 0;
      margin-bottom: 0;
    }
  }
}
</style>
