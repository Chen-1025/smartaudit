<template>
  <div class="page-container">
    <div class="main-content">
      <div class="filter-area">
        <Form ref="filterForm" :model="filterForm" :label-width="80">
          <Row :gutter="16">
            <Col span="6">
              <FormItem label="案件编号" prop="caseNumber">
                <Input v-model="filterForm.caseNumber" placeholder="请输入" />
              </FormItem>
            </Col>
            <Col span="6">
              <FormItem label="管辖单位" prop="jurisdictionUnit">
                <Input v-model="filterForm.jurisdictionUnit" placeholder="请输入" />
              </FormItem>
            </Col>
            <Col span="6">
              <FormItem label="登记时间" prop="registrationTime">
                <DatePicker v-model="filterForm.registrationTime" type="daterange" placeholder="选择日期范围"></DatePicker>
              </FormItem>
            </Col>
            <Col span="6">
              <FormItem label="笔录状态" prop="transcriptStatus">
                <Select v-model="filterForm.transcriptStatus" placeholder="请选择">
                  <Option value="uploaded">已上传</Option>
                  <Option value="not_uploaded">未上传</Option>
                </Select>
              </FormItem>
            </Col>

            <template v-if="showMoreFilters">
              <Col span="6">
                <FormItem label="警情编号" prop="alarmNumber">
                  <Input v-model="filterForm.alarmNumber" placeholder="请输入" />
                </FormItem>
              </Col>
              <Col span="6">
                <FormItem label="受理状态" prop="isAccepted">
                  <Select v-model="filterForm.isAccepted" placeholder="请选择">
                    <Option value="yes">已受理</Option>
                    <Option value="no">未受理</Option>
                  </Select>
                </FormItem>
              </Col>
              <Col span="6">
                <FormItem label="案件名称" prop="caseName">
                  <Input v-model="filterForm.caseName" placeholder="请输入" />
                </FormItem>
              </Col>
              <Col span="6">
                <FormItem label="报案金额" prop="reportAmount">
                  <InputNumber v-model="filterForm.reportAmount.min" placeholder="最小金额" :min="0" style="width: 48%;" />
                  <span style="display: inline-block; width: 4%; text-align: center;">-</span>
                  <InputNumber v-model="filterForm.reportAmount.max" placeholder="最大金额" :min="0" style="width: 48%;" />
                </FormItem>
              </Col>
              <Col span="6">
                <FormItem label="登记人员" prop="registrant">
                  <Input v-model="filterForm.registrant" placeholder="请输入" />
                </FormItem>
              </Col>
              <Col span="6">
                <FormItem label="警情类型" prop="alarmType">
                  <Select v-model="filterForm.alarmType" placeholder="请选择">
                    <Option v-for="type in alarmTypes" :value="type" :key="type">{{ type }}</Option>
                  </Select>
                </FormItem>
              </Col>
            </template>
          </Row>

          <Row>
            <Col span="24" style="text-align: right; margin-top: 10px;">
              <Tooltip content="查询" placement="top">
                <Button type="primary" icon="ios-search" style="margin-right: 8px;" @click="handleSearch"></Button>
              </Tooltip>
              <Tooltip content="重置" placement="top">
                <Button type="default" icon="md-refresh" style="margin-right: 8px;" @click="handleReset"></Button>
              </Tooltip>
              <a @click="toggleFilters" style="margin-right: 8px;">
                {{ showMoreFilters ? '收起' : '展开' }}
                <Icon :type="showMoreFilters ? 'ios-arrow-up' : 'ios-arrow-down'" />
              </a>
            </Col>
          </Row>
        </Form>
      </div>

      <div class="table-area">
        <Table :columns="columns" :data="tableData"></Table>
      </div>

      <div class="pagination-area">
        <Page 
          :total="totalCount" 
          :page-size="pageSize" 
          :current="currentPage" 
          :page-size-opts="[5, 10, 20, 50, 100]" 
          @on-change="handlePageChange" 
          @on-page-size-change="handlePageSizeChange"
          show-sizer 
          show-elevator 
          show-total 
        />
      </div>
    </div>

    <Modal v-model="uploadModalVisible" title="文件上传" footer-hide>
      <Form :label-width="100">
        <FormItem label="案件编号">
          <Input v-model="caseInfo.caseNumber" disabled />
        </FormItem>
        <FormItem label="案件名称">
          <Input v-model="caseInfo.caseName" disabled />
        </FormItem>
      </Form>
      
      <div class="upload-list-container">
        <div v-for="(item, index) in uploadFiles" :key="index" class="upload-item">
          <Upload action="//jsonplaceholder.typicode.com/posts/" :on-success="handleUploadSuccess(index)">
            <Button icon="ios-cloud-upload-outline">上传文件</Button>
          </Upload>
          <span class="file-name">{{ item.name || '未选择文件' }}</span>
          <Select v-model="item.type" class="file-type-select" placeholder="请选择文件类型">
            <Option value="victim">受害人笔录</Option>
            <Option value="suspect">嫌疑人笔录</Option>
            <Option value="warning">预警信息</Option>
            <Option value="synergy">协同数据</Option>
            <Option value="bank">警银数据</Option>
          </Select>
          <Button @click="removeFile(index)" type="error" icon="md-close-circle" style="margin-left: 8px;"></Button>
        </div>
        <Button @click="addFile" icon="md-add" type="primary" ghost style="width: 100%; margin-top: 10px;">添加文件</Button>
      </div>
      
      <div class="modal-footer">
        <Button type="primary" @click="submitUpload">提交</Button>
        <Button @click="cancelUpload">取消</Button>
      </div>
    </Modal>

    <Modal v-model="remarkModalVisible" title="添加备注">
      <Form ref="remarkForm" :model="remarkForm" :rules="remarkRules">
        <FormItem prop="remark">
          <Input v-model="remarkForm.remark" type="textarea" :rows="4" placeholder="请输入备注信息..." />
        </FormItem>
      </Form>
      <div slot="footer">
        <Button type="primary" @click="submitRemark">提交</Button>
        <Button @click="cancelRemark">取消</Button>
      </div>
    </Modal>
  </div>
</template>

<script>
import dayjs from 'dayjs';

export default {
  data () {
    return {
      showMoreFilters: false,
      filterForm: {
        alarmNumber: '',
        caseNumber: '',
        caseName: '',
        jurisdictionUnit: '',
        registrant: '',
        isAccepted: '',
        registrationTime: [], // 保持为数组
        reportAmount: {
          min: null,
          max: null
        },
        transcriptStatus: '',
        alarmType: ''
      },
      currentPage: 1,
      pageSize: 10,
      totalCount: 50,
      
      uploadModalVisible: false,
      remarkModalVisible: false,
      
      remarkForm: {
        remark: ''
      },
      remarkRules: {
        remark: [
          { required: true, message: '备注信息不能为空', trigger: 'blur' }
        ]
      },
      
      caseInfo: {
        caseNumber: '',
        caseName: ''
      },
      uploadFiles: [
        { name: '', type: 'victim' } 
      ],
      columns: [
        { title: '警情编号', key: 'alarmNumber' },
        { title: '是否受理', key: 'isAccepted', render: (h, params) => {
            const isAccepted = params.row.isAccepted;
            const statusText = isAccepted ? '已受理' : '未受理';
            const statusColor = isAccepted ? '#19be6b' : '#ed4014';
            return h('div', {
              style: {
                display: 'flex',
                alignItems: 'center'
              }
            }, [
              h('span', {
                style: {
                  display: 'inline-block',
                  width: '8px',
                  height: '8px',
                  borderRadius: '50%',
                  backgroundColor: statusColor,
                  marginRight: '8px'
                }
              }),
              h('span', statusText)
            ]);
          } 
        },
        { title: '案件编号', key: 'caseNumber' },
        { title: '案件名称', key: 'caseName' },
        { title: '管辖单位', key: 'jurisdictionUnit' },
        { title: '登记人员', key: 'registrant' },
        { title: '登记时间', key: 'registrationTime' },
        { title: '报案金额', key: 'reportAmount' },
        { title: '警情类型', key: 'alarmType' },
        { title: '笔录状态', key: 'transcriptStatus', render: (h, params) => {
            return h('span', params.row.transcriptStatus ? '已上传' : '未上传');
          }
        },
        {
          title: '操作',
          key: 'action',
          width: 120,
          render: (h, params) => {
            const isUploaded = params.row.transcriptStatus;
            return h('div', [
              h('Tooltip', {
                props: {
                  content: isUploaded ? '已上传，无法再次上传' : '上传文件',
                  placement: 'top'
                }
              }, [
                h('Button', {
                  props: {
                    type: isUploaded ? 'text' : 'primary',
                    size: 'small',
                    ghost: true,
                    icon: 'md-cloud-upload',
                    disabled: isUploaded
                  },
                  on: {
                    click: () => {
                      if (!isUploaded) {
                        this.showUploadModal(params.row);
                      }
                    }
                  }
                })
              ]),
              h('Tooltip', {
                props: {
                  content: '添加备注',
                  placement: 'top'
                }
              }, [
                h('Button', {
                  props: {
                    type: 'info',
                    size: 'small',
                    ghost: true,
                    icon: 'md-create'
                  },
                  style: {
                    marginLeft: '8px'
                  },
                  on: {
                    click: () => {
                      this.showRemarkModal(params.row);
                    }
                  }
                })
              ])
            ]);
          }
        }
      ],
      tableData: this.generateTableData(1, 10),
      alarmTypes: []
    };
  },
  mounted() {
    this.getAlarmTypes();
    this.handleSearch();
  },
  methods: {
    getAlarmTypes() {
      setTimeout(() => {
        const mockData = ['电信诈骗', '非法集资', '盗窃', '抢劫', '故意伤害'];
        this.alarmTypes = mockData;
        console.log('--- 警情类型数据已从后台接口获取：---', this.alarmTypes);
      }, 500);
    },
    generateTableData(page, size) {
      const data = [];
      const start = (page - 1) * size + 1;
      const end = page * size;
      const alarmTypes = ['电信诈骗', '非法集资', '盗窃', '抢劫', '故意伤害'];
      for (let i = start; i <= end; i++) {
        data.push({
          alarmNumber: `JQ2025080${Math.floor(Math.random() * 30) + 1}0${i}`,
          isAccepted: i % 2 === 0,
          caseNumber: `AJ2025080${Math.floor(Math.random() * 30) + 1}0${i}`,
          caseName: `模拟案件${i}`,
          jurisdictionUnit: i % 2 === 0 ? '刑警大队' : '经侦大队',
          registrant: `人员${i}`,
          registrationTime: `2025-08-0${Math.floor(Math.random() * 9) + 1} 09:00`,
          reportAmount: `${Math.floor(Math.random() * 1000000) + 10000}`,
          alarmType: alarmTypes[i % alarmTypes.length],
          transcriptStatus: i % 2 === 1
        });
      }
      return data;
    },
    toggleFilters () {
      this.showMoreFilters = !this.showMoreFilters;
    },
    handleSearch () {
      // 在这里处理日期范围，将时间戳加上
      let startTime = '';
      let endTime = '';
      if (this.filterForm.registrationTime && this.filterForm.registrationTime.length === 2 && this.filterForm.registrationTime[0]) {
        // 使用 dayjs 库格式化日期
        startTime = dayjs(this.filterForm.registrationTime[0]).startOf('day').format('YYYY-MM-DD HH:mm:ss');
        endTime = dayjs(this.filterForm.registrationTime[1]).endOf('day').format('YYYY-MM-DD HH:mm:ss');
      }

      const params = {
        ...this.filterForm,
        // 使用格式化后的日期时间
        registrationTime: [startTime, endTime],
        page: this.currentPage,
        size: this.pageSize
      };
      console.log('--- 触发查询接口，查询参数：---', params);
      this.tableData = this.generateTableData(this.currentPage, this.pageSize);
    },
    handleReset () {
      this.$refs.filterForm.resetFields();
      this.currentPage = 1;
      this.handleSearch();
      console.log('表单已重置，并查询第一页数据');
    },
    handlePageChange(page) {
      this.currentPage = page;
      console.log('--- 触发分页接口，跳转到第' + page + '页 ---');
      this.handleSearch();
    },
    handlePageSizeChange(size) {
      this.pageSize = size;
      this.currentPage = 1;
      console.log('--- 触发分页接口，每页显示条数改变为：' + size + '条 ---');
      this.handleSearch();
    },
    showUploadModal (row) {
      this.caseInfo.caseNumber = row.caseNumber;
      this.caseInfo.caseName = row.caseName;
      this.uploadFiles = [{ name: '', type: 'victim' }];
      this.uploadModalVisible = true;
    },
    addFile () {
      this.uploadFiles.push({ name: '', type: 'victim' });
    },
    removeFile (index) {
      if (this.uploadFiles.length > 1) {
        this.uploadFiles.splice(index, 1);
      }
    },
    handleUploadSuccess (index) {
      return (response, file) => {
        this.uploadFiles[index].name = file.name;
        this.$Message.success('文件上传成功');
      };
    },
    submitUpload () {
      console.log('提交上传文件:', this.uploadFiles);
      this.uploadModalVisible = false;
      this.$Message.success('文件提交成功');
    },
    cancelUpload () {
      this.uploadModalVisible = false;
    },
    showRemarkModal (row) {
      this.remarkForm.remark = '';
      this.remarkModalVisible = true;
      console.log('正在为案件：' + row.caseNumber + ' 添加备注');
    },
    submitRemark () {
      this.$refs.remarkForm.validate((valid) => {
        if (valid) {
          console.log('提交备注:', this.remarkForm.remark);
          this.remarkModalVisible = false;
          this.$Message.success('备注提交成功');
        } else {
          this.$Message.error('请填写备注信息');
        }
      });
    },
    cancelRemark () {
      this.remarkModalVisible = false;
    }
  }
};
</script>

<style scoped>
/* 样式部分保持不变 */
.page-container {
  background-color: #f5f7f9;
  min-height: 100vh;
}

.main-content {
  padding: 20px;
}

.filter-area {
  margin-bottom: 20px;
  padding: 16px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0,0,0,.05);
}

.table-area {
  margin-bottom: 20px;
}

.pagination-area {
  text-align: right;
}

.upload-list-container {
  border: 1px dashed #dcdee2;
  padding: 15px;
  border-radius: 4px;
  margin-bottom: 20px;
}

.upload-item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.upload-item .file-name {
  flex: 1;
  margin-left: 10px;
  color: #515a6e;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.file-type-select {
  width: 180px;
  margin-left: 10px;
}

.modal-footer {
  text-align: right;
  padding-top: 10px;
  border-top: 1px solid #e8eaec;
  margin-top: 10px;
}
.modal-footer Button {
  margin-left: 8px;
}
</style>