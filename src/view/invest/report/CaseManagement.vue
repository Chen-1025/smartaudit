<template>
    <div class="page-container">
        <div class="main-content">
            <div class="filter-area">
                <Form ref="filterForm" :model="filterForm" :label-width="90">
                    <Row :gutter="16">
                        <Col span="6">
                        <FormItem label="案件编号" prop="caseNumber">
                            <Input v-model="filterForm.caseNumber" placeholder="请输入" />
                        </FormItem>
                        </Col>
                        <Col span="6">
                        <FormItem label="案件名称" prop="caseName">
                            <Input v-model="filterForm.caseName" placeholder="请输入" />
                        </FormItem>
                        </Col>
                        <Col span="6">
                        <FormItem label="警情类别" prop="alarmType">
                            <Select v-model="filterForm.alarmType" placeholder="请选择">
                                <Option v-for="type in alarmTypes" :value="type" :key="type">{{ type }}</Option>
                            </Select>
                        </FormItem>
                        </Col>
                        <Col span="6">
                        <FormItem label="报警时间" prop="alarmTime">
                            <DatePicker type="daterange" placeholder="选择日期范围" :value="filterForm.alarmTime"
                                @on-change="handleDateChange"></DatePicker>
                        </FormItem>
                        </Col>

                        <template v-if="showMoreFilters">
                            <Col span="6">
                            <FormItem label="警情编号" prop="alarmNumber">
                                <Input v-model="filterForm.alarmNumber" placeholder="请输入" />
                            </FormItem>
                            </Col>
                            <Col span="6">
                            <FormItem label="公安机关名称" prop="policeDepartment">
                                <Input v-model="filterForm.policeDepartment" placeholder="请输入" />
                            </FormItem>
                            </Col>
                            <Col span="6">
                            <FormItem label="创建人" prop="creator">
                                <Input v-model="filterForm.creator" placeholder="请输入" />
                            </FormItem>
                            </Col>
                            <Col span="6">
                            <FormItem label="金额" prop="amount">
                                <InputNumber v-model="filterForm.amount.min" placeholder="最小金额" :min="0"
                                    style="width: 48%;" />
                                <span style="display: inline-block; width: 4%; text-align: center;">-</span>
                                <InputNumber v-model="filterForm.amount.max" placeholder="最大金额" :min="0"
                                    style="width: 48%;" />
                            </FormItem>
                            </Col>
                            <Col span="6">
                            <FormItem label="报告状态" prop="reportStatus">
                                <Select v-model="filterForm.reportStatus" placeholder="请选择">
                                    <Option value="submitted">已提交</Option>
                                    <Option value="draft">草稿</Option>
                                    <Option value="reviewing">审核中</Option>
                                </Select>
                            </FormItem>
                            </Col>
                        </template>
                    </Row>

                    <Row>
                        <Col span="24" style="text-align: right; margin-top: 10px;">
                        <Tooltip content="查询" placement="top">
                            <Button type="primary" icon="ios-search" style="margin-right: 8px;"
                                @click="handleSearch">查询</Button>
                        </Tooltip>
                        <Tooltip content="重置" placement="top">
                            <Button type="default" icon="md-refresh" style="margin-right: 8px;"
                                @click="handleReset">重置</Button>
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
                <Table :columns="columns" :data="tableData" border></Table>
            </div>

            <div class="pagination-area">
                <Page :total="totalCount" :page-size="pageSize" :current="currentPage" :page-size-opts="[10, 20, 50]"
                    @on-change="handlePageChange" @on-page-size-change="handlePageSizeChange" show-sizer show-elevator
                    show-total />
            </div>
        </div>

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
    name: 'CaseManagement',
    data() {
        return {
            showMoreFilters: false,
            filterForm: {
                caseNumber: '',
                caseName: '',
                alarmType: '',
                alarmTime: null,
                policeDepartment: '',
                reportStatus: '',
                // 新增筛选条件
                alarmNumber: '',
                creator: '',
                amount: {
                    min: null,
                    max: null
                }
            },
            currentPage: 1,
            pageSize: 10,
            totalCount: 100,

            remarkModalVisible: false,
            remarkForm: {
                remark: ''
            },
            remarkRules: {
                remark: [
                    { required: true, message: '备注信息不能为空', trigger: 'blur' }
                ]
            },

            alarmTypes: [],
            columns: [
                { title: '警情编号', key: 'alarmNumber', minWidth: 150 },
                { title: '案件编号', key: 'caseNumber', minWidth: 150 },
                { title: '案件名称', key: 'caseName', minWidth: 150 },
                { title: '警情类别', key: 'alarmType', minWidth: 120 },
                {
                    title: '报告状态', key: 'reportStatus', minWidth: 120, render: (h, params) => {
                        const status = params.row.reportStatus;
                        let statusText = '';
                        let statusColor = '';
                        switch (status) {
                            case 'submitted':
                                statusText = '已提交';
                                statusColor = '#19be6b';
                                break;
                            case 'draft':
                                statusText = '草稿';
                                statusColor = '#ff9900';
                                break;
                            case 'reviewing':
                                statusText = '审核中';
                                statusColor = '#2d8cf0';
                                break;
                            default:
                                statusText = '未知';
                                statusColor = '#999';
                        }
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
                { title: '报警时间', key: 'alarmTime', minWidth: 180 },
                { title: '金额', key: 'amount', minWidth: 120 },
                { title: '公安机关名称', key: 'policeDepartment', minWidth: 150 },
                { title: '创建人', key: 'creator', minWidth: 100 },
                {
                    title: '操作',
                    key: 'action',
                    width: 200,
                    align: 'center',
                    fixed: 'right',
                    render: (h, params) => {
                        return h('div', [
                            h('Tooltip', { props: { content: '详情', placement: 'top' } }, [
                                h('Button', {
                                    props: { type: 'text', size: 'small', icon: 'md-eye' },
                                    on: {
                                        click: () => { this.showDetail(params.row); }
                                    }
                                })
                            ]),
                            h('Tooltip', { props: { content: '导图', placement: 'top' } }, [
                                h('Button', {
                                    props: { type: 'text', size: 'small', icon: 'md-git-network' },
                                    on: {
                                        click: () => { this.showMindMap(params.row); }
                                    }
                                })
                            ]),
                            h('Tooltip', { props: { content: '备注', placement: 'top' } }, [
                                h('Button', {
                                    props: { type: 'text', size: 'small', icon: 'md-create' },
                                    on: {
                                        click: () => { this.showRemarkModal(params.row); }
                                    }
                                })
                            ]),
                            h('Tooltip', { props: { content: '删除', placement: 'top' } }, [
                                h('Button', {
                                    props: { type: 'text', size: 'small', icon: 'md-trash' },
                                    on: {
                                        click: () => { this.deleteCase(params.row); }
                                    }
                                })
                            ])
                        ]);
                    }
                }
            ],
            tableData: []
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
            }, 500);
        },
        generateTableData(page, size) {
            const data = [];
            const start = (page - 1) * size + 1;
            const end = page * size;
            const alarmTypes = ['电信诈骗', '非法集资', '盗窃', '抢劫', '故意伤害'];
            const statuses = ['submitted', 'draft', 'reviewing'];
            const departments = ['刑警大队', '经侦大队', '网安支队'];
            for (let i = start; i <= end; i++) {
                data.push({
                    alarmNumber: `JQ2025080${Math.floor(Math.random() * 30) + 1}0${i}`,
                    caseNumber: `AJ2025080${Math.floor(Math.random() * 30) + 1}0${i}`,
                    caseName: `案件名称${i}`,
                    alarmType: alarmTypes[i % alarmTypes.length],
                    alarmTime: `2025-08-0${Math.floor(Math.random() * 9) + 1} 10:00:0${Math.floor(Math.random() * 9)}`,
                    amount: `${Math.floor(Math.random() * 1000000) + 10000}`,
                    policeDepartment: departments[i % departments.length],
                    creator: `创建人${i}`,
                    reportStatus: statuses[i % statuses.length],
                });
            }
            return data;
        },
        toggleFilters() {
            this.showMoreFilters = !this.showMoreFilters;
        },
        handleDateChange(value) {
            if (value && value[0] && value[1]) {
                const startDate = dayjs(value[0]).startOf('day').format('YYYY-MM-DD HH:mm:ss');
                const endDate = dayjs(value[1]).endOf('day').format('YYYY-MM-DD HH:mm:ss');
                this.filterForm.alarmTime = [startDate, endDate];
            } else {
                this.filterForm.alarmTime = null;
            }
        },
        handleSearch() {
            const params = {
                ...this.filterForm,
                amountMin: this.filterForm.amount.min,
                amountMax: this.filterForm.amount.max,
                page: this.currentPage,
                size: this.pageSize
            };
            delete params.amount;
            console.log('--- 触发查询接口，查询参数：---', params);
            this.tableData = this.generateTableData(this.currentPage, this.pageSize);
        },
        handleReset() {
            this.$refs.filterForm.resetFields();
            this.filterForm.alarmTime = null;
            this.filterForm.amount.min = null;
            this.filterForm.amount.max = null;
            this.currentPage = 1;
            this.handleSearch();
        },
        handlePageChange(page) {
            this.currentPage = page;
            this.handleSearch();
        },
        handlePageSizeChange(size) {
            this.pageSize = size;
            this.currentPage = 1;
            this.handleSearch();
        },
        // 将 showDetail 方法修改为路由跳转
        showDetail(row) {
            console.log('跳转到案件详情：', row.caseNumber);
            // 假设您的路由配置中有一个名为 'caseDetail' 的路由
            this.$router.push({
                name: '案件详情', // 路由的 name
                params: { caseNumber: row.caseNumber }
            });
            // 或者使用 path
            // this.$router.push(`/case-management/${row.caseNumber}/detail`);
        },
        // 将 showMindMap 方法修改为路由跳转
        showMindMap(row) {
            console.log('跳转到案件导图：', row.caseNumber);
            // 假设您的路由配置中有一个名为 'caseMindMap' 的路由
            this.$router.push({
                name: '案件导图',
                params: { caseNumber: row.caseNumber }
            });
            // 或者使用 path
            // this.$router.push(`/case-management/${row.caseNumber}/mind-map`);
        },
        showRemarkModal(row) {
            this.remarkForm.remark = '';
            this.remarkModalVisible = true;
            console.log('正在为案件：' + row.caseNumber + ' 添加备注');
        },
        submitRemark() {
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
        cancelRemark() {
            this.remarkModalVisible = false;
        },
        deleteCase(row) {
            this.$Modal.confirm({
                title: '确认删除',
                content: `确定要删除案件“${row.caseName}”吗？`,
                onOk: () => {
                    console.log('删除案件：', row.caseNumber);
                    this.$Message.success('案件删除成功');
                }
            });
        }
    }
};
</script>

<style scoped>
/* 样式部分保持不变 */
.page-container {
    background-color: #f5f7f9;
    min-height: 100vh;
    padding: 20px;
}

.main-content {
    background: #fff;
    padding: 20px;
    border-radius: 4px;
    box-shadow: 0 2px 12px rgba(0, 0, 0, .05);
}

.filter-area {
    margin-bottom: 20px;
    padding: 16px;
    border-bottom: 1px solid #e8eaec;
}

.filter-area .ivu-btn {
    font-size: 14px;
}

.filter-area .ivu-btn[type="primary"] {
    background-color: #2d8cf0;
    border-color: #2d8cf0;
    color: #fff;
}

.filter-area .ivu-btn[type="primary"]:hover {
    background-color: #57a3f3;
    border-color: #57a3f3;
}

.filter-area .ivu-btn[type="default"] {
    background-color: #f8f8f9;
    border-color: #dcdee2;
    color: #515a6e;
}

.filter-area .ivu-btn[type="default"]:hover {
    background-color: #f0f2f5;
    border-color: #d1d4d8;
}

.table-area {
    margin-bottom: 20px;
}

.table-area .ivu-table-wrapper {
    overflow-x: auto;
    overflow-y: hidden;
    border: 1px solid #e8eaec;
    border-radius: 4px;
}

.table-area .ivu-table th,
.table-area .ivu-table td {
    white-space: nowrap;
}

.ivu-table .ivu-btn-text {
    font-size: 16px;
    padding: 0;
    height: 24px;
    line-height: 24px;
}

.ivu-table .ivu-btn-text.ivu-btn-primary {
    color: #2d8cf0;
}

.ivu-table .ivu-btn-text.ivu-btn-info {
    color: #2db7f5;
}

.ivu-table .ivu-btn-text.ivu-btn-warning {
    color: #ff9900;
}

.ivu-table .ivu-btn-text.ivu-btn-error {
    color: #ed4014;
}

.pagination-area {
    text-align: right;
}
</style>