<template>
    <div class="case-detail-container">
        <div class="action-buttons" style="text-align: right; margin-bottom: 20px;">
            <Button type="success" @click="handleEvidenceRequest" style="margin-right: 10px;">调证</Button>
            <Button type="error" @click="handleDeleteCase">删除</Button>
        </div>

        <Card :bordered="false" dis-hover>
            <div slot="title" @click="toggleBasicInfo"
                style="cursor: pointer; display: flex; align-items: center; justify-content: space-between;">
                <span>案件基本信息</span>
                <Icon :type="basicInfoCollapsed ? 'ios-arrow-down' : 'ios-arrow-up'" style="margin-left: 8px;" />
            </div>
            <div class="detail-info-grid" v-show="!basicInfoCollapsed">
                <Row :gutter="16">
                    <Col span="8">
                    <div class="info-item">
                        <span class="label">案件编号：</span>
                        <span class="value">{{ caseInfo.caseNumber }}</span>
                    </div>
                    </Col>
                    <Col span="8">
                    <div class="info-item">
                        <span class="label">案件名称：</span>
                        <span class="value">{{ caseInfo.caseName }}</span>
                    </div>
                    </Col>
                    <Col span="8">
                    <div class="info-item">
                        <span class="label">警情类别：</span>
                        <span class="value">{{ caseInfo.alarmType }}</span>
                    </div>
                    </Col>
                </Row>
                <Row :gutter="16">
                    <Col span="8">
                    <div class="info-item">
                        <span class="label">报警时间：</span>
                        <span class="value">{{ caseInfo.alarmTime }}</span>
                    </div>
                    </Col>
                    <Col span="8">
                    <div class="info-item">
                        <span class="label">金额：</span>
                        <span class="value">{{ caseInfo.amount }} 元</span>
                    </div>
                    </Col>
                    <Col span="8">
                    <div class="info-item">
                        <span class="label">案发时间：</span>
                        <span class="value">{{ caseInfo.occurrenceTime }}</span>
                    </div>
                    </Col>
                </Row>
                <Row :gutter="16">
                    <Col span="8">
                    <div class="info-item">
                        <span class="label">案发地点：</span>
                        <span class="value">{{ caseInfo.occurrenceLocation }}</span>
                    </div>
                    </Col>
                    <Col span="8">
                    <div class="info-item">
                        <span class="label">侦办单位：</span>
                        <span class="value">{{ caseInfo.investigationUnit }}</span>
                    </div>
                    </Col>
                    <Col span="8">
                    <div class="info-item">
                        <span class="label">侦办人员：</span>
                        <span class="value">{{ caseInfo.investigatingPersonnel }}</span>
                    </div>
                    </Col>
                </Row>
                <Row :gutter="16">
                    <Col span="8">
                    <div class="info-item">
                        <span class="label">管辖单位：</span>
                        <span class="value">{{ caseInfo.jurisdictionalUnit }}</span>
                    </div>
                    </Col>
                    <Col span="8">
                    <div class="info-item">
                        <span class="label">受理时间：</span>
                        <span class="value">{{ caseInfo.acceptanceTime }}</span>
                    </div>
                    </Col>
                    <Col span="8">
                    <div class="info-item">
                        <span class="label">立案时间：</span>
                        <span class="value">{{ caseInfo.filingTime }}</span>
                    </div>
                    </Col>
                </Row>
                <Row :gutter="16">
                    <Col span="8">
                    <div class="info-item">
                        <span class="label">立案单位：</span>
                        <span class="value">{{ caseInfo.filingUnit }}</span>
                    </div>
                    </Col>
                </Row>
            </div>
        </Card>

        <Divider />

        <Card :bordered="false" dis-hover>
            <div slot="title" @click="toggleBriefDescription"
                style="cursor: pointer; display: flex; align-items: center; justify-content: space-between;">
                <span>简要案情</span>
                <Icon :type="briefDescriptionCollapsed ? 'ios-arrow-down' : 'ios-arrow-up'" style="margin-left: 8px;" />
            </div>
            <div class="brief-description" v-show="!briefDescriptionCollapsed">
                <p>{{ briefDescription }}</p>
            </div>
        </Card>

        <Divider />

        <Card :bordered="false" dis-hover>
            <div slot="title" @click="toggleEvidence"
                style="cursor: pointer; display: flex; align-items: center; justify-content: space-between;">
                <span>调证资料</span>
                <Icon :type="evidenceCollapsed ? 'ios-arrow-down' : 'ios-arrow-up'" style="margin-left: 8px;" />
            </div>
            <div class="upload-area" v-show="!evidenceCollapsed">
                <Upload action="//jsonplaceholder.typicode.com/posts/" :before-upload="handleBeforeUpload"
                    :on-success="handleUploadSuccess" :on-error="handleUploadError" :on-remove="handleFileRemove"
                    :format="['jpg', 'jpeg', 'png', 'doc', 'docx', 'pdf']" :max-size="2048" multiple type="drag"
                    ref="upload">
                    <div style="padding: 20px 0">
                        <Icon type="ios-cloud-upload" size="52" style="color: #3399ff"></Icon>
                        <p>将文件拖拽到此处，或点击上传</p>
                    </div>
                </Upload>
                <ul class="upload-list">
                    <li v-for="(file, index) in fileList" :key="index">
                        <Icon type="ios-document" /> {{ file.name }}
                        <a @click="handleFileRemove(file, index)" style="float: right;">删除</a>
                    </li>
                </ul>
            </div>
        </Card>

        <Divider />

        <Card :bordered="false" dis-hover>
            <div slot="title" @click="togglePersonnel"
                style="cursor: pointer; display: flex; align-items: center; justify-content: space-between;">
                <span>人员流信息</span>
                <Icon :type="personnelCollapsed ? 'ios-arrow-down' : 'ios-arrow-up'" style="margin-left: 8px;" />
            </div>
            <div v-show="!personnelCollapsed">
                <div style="margin-bottom: 10px;">
                    <Button icon="md-add" @click="handleAddPersonnel">新增人员</Button>
                </div>
                <div class="table-scroll-container">
                    <Table :columns="personnelColumns" :data="personnelFlows" border ref="personnelTable"
                        :key="tableKey"></Table>
                </div>
            </div>
        </Card>
    </div>
</template>

<script>
export default {
    name: 'CaseDetail',
    data() {
        return {
            caseNumber: '',
            caseInfo: {},
            briefDescription: '',
            personnelFlows: [],
            fileList: [],
            tableKey: 0,
            tempRow: {},
            editingId: null,
            basicInfoCollapsed: false,
            briefDescriptionCollapsed: false,
            evidenceCollapsed: false,
            personnelCollapsed: false,
            personnelColumns: [
                {
                    title: '姓名',
                    key: 'name',
                    minWidth: 100,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.name),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempRow.name : row.name,
                                    placeholder: '请输入姓名',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempRow, 'name', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '性别',
                    key: 'gender',
                    width: 80,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.gender),
                            h('Select', {
                                props: {
                                    value: isEditing ? this.tempRow.gender : row.gender,
                                    placeholder: '请选择',
                                    size: 'small',
                                    transfer: true
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'on-change': (value) => {
                                        this.$set(this.tempRow, 'gender', value);
                                    }
                                }
                            }, [
                                h('Option', { props: { value: '男' } }, '男'),
                                h('Option', { props: { value: '女' } }, '女')
                            ])
                        ]);
                    }
                },
                {
                    title: '年龄',
                    key: 'age',
                    width: 80,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.age),
                            h('InputNumber', {
                                props: {
                                    value: isEditing ? this.tempRow.age : row.age,
                                    min: 0,
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'on-change': (value) => {
                                        this.$set(this.tempRow, 'age', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '证件号码',
                    key: 'idNumber',
                    minWidth: 150,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.idNumber),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempRow.idNumber : row.idNumber,
                                    placeholder: '请输入证件号码',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempRow, 'idNumber', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '职业',
                    key: 'occupation',
                    minWidth: 100,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.occupation),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempRow.occupation : row.occupation,
                                    placeholder: '请输入职业',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempRow, 'occupation', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '住址',
                    key: 'address',
                    minWidth: 200,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.address),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempRow.address : row.address,
                                    placeholder: '请输入住址',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempRow, 'address', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '联系电话',
                    key: 'phoneNumber',
                    minWidth: 150,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.phoneNumber),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempRow.phoneNumber : row.phoneNumber,
                                    placeholder: '请输入联系电话',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempRow, 'phoneNumber', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '描述',
                    key: 'description',
                    minWidth: 300,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.description),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempRow.description : row.description,
                                    placeholder: '请输入描述',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempRow, 'description', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '操作',
                    key: 'action',
                    width: 140,
                    align: 'center',
                    fixed: 'right',
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingId;
                        return h('div', {
                            style: {
                                display: 'flex',
                                justifyContent: 'center',
                                gap: '5px'
                            }
                        }, [
                            h('Button', {
                                props: {
                                    type: 'primary',
                                    size: 'small',
                                    icon: isEditing ? 'md-checkmark' : 'md-create'
                                },
                                on: {
                                    click: () => {
                                        if (isEditing) {
                                            this.handleSaveRow(row.id);
                                        } else {
                                            this.handleEditRow(row.id);
                                        }
                                    }
                                }
                            }, isEditing ? '保存' : '编辑'),
                            h('Button', {
                                props: {
                                    type: 'error',
                                    size: 'small',
                                    icon: 'md-trash'
                                },
                                on: {
                                    click: () => {
                                        this.handleDeletePersonnel(row.id);
                                    }
                                }
                            }, '删除')
                        ]);
                    }
                }
            ]
        };
    },
    created() {
        this.caseNumber = this.$route.params.caseNumber;
        this.fetchCaseDetail();
    },
    methods: {
        fetchCaseDetail() {
            console.log(`--- 调用接口获取案件详情，案件编号：${this.caseNumber} ---`);

            const mockData = {
                caseNumber: this.caseNumber,
                caseName: '张三涉嫌电信诈骗案',
                alarmType: '电信诈骗',
                alarmTime: '2025-08-01 09:30:00',
                amount: 500000,
                occurrenceTime: '2025-07-30 15:00:00',
                occurrenceLocation: '某市某区某小区',
                investigationUnit: '某市公安局刑警大队',
                investigatingPersonnel: '李警官, 王警官',
                jurisdictionalUnit: '某市公安局',
                acceptanceTime: '2025-08-01 10:00:00',
                filingTime: '2025-08-02 14:00:00',
                filingUnit: '某市公安局',
                briefDescription: '该案件初步侦查结果显示，嫌疑人张三通过网络社交平台发布虚假投资信息，诱骗受害人转账。目前已有多名受害者报案，涉案金额较大。',
                personnelFlows: [
                    { name: '张三', gender: '男', age: 35, idNumber: '320000199001011234', occupation: '无业', address: '某市某区某路101号', phoneNumber: '13812345678', description: '主要嫌疑人' },
                    { name: '李四', gender: '男', age: 40, idNumber: '320000198502025678', occupation: '公司职员', address: '某市某区某路102号', phoneNumber: '13987654321', description: '同案犯，负责资金转移' }
                ],
                files: [
                    { name: '调证函-银行.pdf', status: 'finished', url: 'http://example.com/file1.pdf' },
                    { name: '嫌疑人笔录.doc', status: 'finished', url: 'http://example.com/file2.doc' }
                ]
            };

            setTimeout(() => {
                this.caseInfo = mockData;
                this.briefDescription = mockData.briefDescription;
                this.personnelFlows = mockData.personnelFlows.map((item, index) => ({ ...item, id: index + 1 }));
                this.fileList = mockData.files;
                this.tableKey++;
            }, 500);
        },
        toggleBasicInfo() {
            this.basicInfoCollapsed = !this.basicInfoCollapsed;
        },
        toggleBriefDescription() {
            this.briefDescriptionCollapsed = !this.briefDescriptionCollapsed;
        },
        toggleEvidence() {
            this.evidenceCollapsed = !this.evidenceCollapsed;
        },
        togglePersonnel() {
            this.personnelCollapsed = !this.personnelCollapsed;
        },
        handleEditRow(id) {
            this.editingId = id;
            const index = this.personnelFlows.findIndex(p => p.id === id);
            if (index !== -1) {
                this.tempRow = JSON.parse(JSON.stringify(this.personnelFlows[index]));
            }
        },
        handleSaveRow(id) {
            const index = this.personnelFlows.findIndex(p => p.id === id);
            if (index !== -1) {
                // 使用 this.$set 确保数据更新被 Vue 响应式系统正确捕获
                this.$set(this.personnelFlows, index, { ...this.tempRow, id: id });
                this.tempRow = {};
                this.editingId = null;
                console.log('保存人员信息：', this.personnelFlows[index]);
                this.$Message.success('保存成功');
            }
        },
        handleAddPersonnel() {
            const newId = this.personnelFlows.length > 0 ? Math.max(...this.personnelFlows.map(p => p.id)) + 1 : 1;
            const newPersonnel = {
                id: newId,
                name: '', gender: '', age: null, idNumber: '', occupation: '', address: '', phoneNumber: '', description: ''
            };

            this.personnelFlows = [...this.personnelFlows, newPersonnel];
            this.tempRow = { ...newPersonnel };
            this.editingId = newId;
            this.tableKey++;

            console.log('新增人员信息成功，当前人员列表：', this.personnelFlows);
            console.log('注意：新行的字段默认为空，请在表格中输入内容。');
        },
        handleDeletePersonnel(id) {
            this.$Modal.confirm({
                title: '确认删除',
                content: '确定要删除该人员信息吗？',
                onOk: () => {
                    const index = this.personnelFlows.findIndex(p => p.id === id);
                    if (index !== -1) {
                        this.personnelFlows.splice(index, 1);
                        if (this.editingId === id) {
                            this.editingId = null;
                            this.tempRow = {};
                        }
                        this.$Message.success('删除成功');
                        this.tableKey++;
                    }
                }
            });
        },
        handleBeforeUpload() {
            return true;
        },
        handleUploadSuccess(response, file, fileList) {
            this.$Message.success('文件上传成功');
            this.fileList.push(file);
        },
        handleUploadError() {
            this.$Message.error('文件上传失败');
        },
        handleFileRemove(file, index) {
            this.$Modal.confirm({
                title: '确认删除',
                content: `确定要删除文件“${file.name}”吗？`,
                onOk: () => {
                    this.fileList.splice(index, 1);
                    this.$Message.success('文件删除成功');
                }
            });
        },
        handleEvidenceRequest() {
            console.log('--- 触发调证功能 ---');
            this.$Message.info('触发调证功能，请实现相应业务逻辑');
        },
        handleDeleteCase() {
            this.$Modal.confirm({
                title: '确认删除案件',
                content: `确定要删除此案件（${this.caseInfo.caseName}）吗？`,
                onOk: () => {
                    console.log('--- 触发删除案件功能 ---');
                    this.$Message.success('案件删除成功');
                    this.$router.push({ name: '案件管理' });
                }
            });
        }
    }
};
</script>

<style scoped>
.case-detail-container {
    padding: 20px;
    background-color: #f8f8f9;
    min-height: calc(100vh - 84px);
}

.detail-info-grid .info-item {
    margin-bottom: 15px;
}

.detail-info-grid .label {
    font-weight: bold;
    color: #515a6e;
    display: inline-block;
    width: 100px;
    text-align: right;
    margin-right: 10px;
}

.detail-info-grid .value {
    color: #17233d;
}

.brief-description {
    background-color: #f7f7f7;
    padding: 15px;
    border-radius: 4px;
    min-height: 100px;
    color: #515a6e;
}

.table-scroll-container {
    overflow-x: auto;
}

.upload-area {
    padding: 10px 0;
}

.upload-list {
    list-style: none;
    padding: 0;
    margin-top: 10px;
}

.upload-list li {
    padding: 8px 10px;
    border: 1px solid #dcdee2;
    border-radius: 4px;
    margin-bottom: 5px;
    background-color: #f9f9f9;
}
</style>