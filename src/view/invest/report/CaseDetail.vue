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
                <div style="margin-bottom: 10px; display: flex; justify-content: space-between; align-items: center;">
                    <RadioGroup v-model="personnelType" type="button" @on-change="handlePersonnelTypeChange">
                        <Radio label="suspect">嫌疑人</Radio>
                        <Radio label="victim">受害人</Radio>
                    </RadioGroup>
                    <Button icon="md-add" @click="handleAddPersonnel">新增人员</Button>
                </div>
                <div class="table-scroll-container">
                    <Table :columns="personnelColumns" :data="currentPersonnelData" border ref="personnelTable"
                        :key="personnelTableKey"></Table>
                </div>
            </div>
        </Card>

        <Divider />

        <Card :bordered="false" dis-hover>
            <div slot="title" @click="toggleFinancialFlows"
                style="cursor: pointer; display: flex; align-items: center; justify-content: space-between;">
                <span>资金流信息</span>
                <Icon :type="financialFlowsCollapsed ? 'ios-arrow-down' : 'ios-arrow-up'" style="margin-left: 8px;" />
            </div>
            <div v-show="!financialFlowsCollapsed">
                <div style="text-align: right; margin-bottom: 10px;">
                    <Button icon="md-add" @click="handleAddFinancialFlow">新增资金流</Button>
                </div>
                <div class="table-scroll-container">
                    <Table :columns="financialFlowsColumns" :data="financialFlows" border ref="financialTable"
                        :key="financialTableKey"></Table>
                </div>
            </div>
        </Card>

        <Divider />

        <Card :bordered="false" dis-hover>
            <div slot="title" @click="toggleCommunicationFlows"
                style="cursor: pointer; display: flex; align-items: center; justify-content: space-between;">
                <span>通讯流信息</span>
                <Icon :type="communicationFlowsCollapsed ? 'ios-arrow-down' : 'ios-arrow-up'"
                    style="margin-left: 8px;" />
            </div>
            <div v-show="!communicationFlowsCollapsed">
                <div style="text-align: right; margin-bottom: 10px;">
                    <Button icon="md-add" @click="handleAddCommunicationFlow">新增通讯流</Button>
                </div>
                <div class="table-scroll-container">
                    <Table :columns="communicationFlowsColumns" :data="communicationFlows" border
                        ref="communicationTable" :key="communicationTableKey"></Table>
                </div>
            </div>
        </Card>

        <Divider />

        <Card :bordered="false" dis-hover>
            <div slot="title" @click="toggleNetworkFlows"
                style="cursor: pointer; display: flex; align-items: center; justify-content: space-between;">
                <span>网络流信息</span>
                <Icon :type="networkFlowsCollapsed ? 'ios-arrow-down' : 'ios-arrow-up'" style="margin-left: 8px;" />
            </div>
            <div v-show="!networkFlowsCollapsed">
                <Tabs v-model="networkFlowsActiveTab">
                    <TabPane label="涉诈App" name="scamApps">
                        <div style="text-align: right; margin-bottom: 10px;">
                            <Button icon="md-add" @click="handleAddScamApp">新增涉诈App</Button>
                        </div>
                        <div class="table-scroll-container">
                            <Table :columns="scamAppColumns" :data="scamApps" border ref="scamAppTable"
                                :key="scamAppTableKey"></Table>
                        </div>
                    </TabPane>
                    <TabPane label="涉诈网站" name="scamWebsites">
                        <div style="text-align: right; margin-bottom: 10px;">
                            <Button icon="md-add" @click="handleAddScamWebsite">新增涉诈网站</Button>
                        </div>
                        <div class="table-scroll-container">
                            <Table :columns="scamWebsiteColumns" :data="scamWebsites" border ref="scamWebsiteTable"
                                :key="scamWebsiteTableKey"></Table>
                        </div>
                    </TabPane>
                    <TabPane label="涉诈虚拟身份" name="scamVirtualIdentities">
                        <div style="text-align: right; margin-bottom: 10px;">
                            <Button icon="md-add" @click="handleAddScamVirtualIdentity">新增涉诈虚拟身份</Button>
                        </div>
                        <div class="table-scroll-container">
                            <Table :columns="scamVirtualIdentityColumns" :data="scamVirtualIdentities" border
                                ref="scamVirtualIdentityTable" :key="scamVirtualIdentityTableKey"></Table>
                        </div>
                    </TabPane>
                </Tabs>
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
            personnelFlows: {
                suspect: [],
                victim: []
            },
            financialFlows: [],
            communicationFlows: [],
            scamApps: [],
            scamWebsites: [],
            scamVirtualIdentities: [],
            personnelType: 'suspect',
            networkFlowsActiveTab: 'scamApps',
            fileList: [],
            personnelTableKey: 0,
            financialTableKey: 0,
            communicationTableKey: 0,
            scamAppTableKey: 0,
            scamWebsiteTableKey: 0,
            scamVirtualIdentityTableKey: 0,
            tempPersonnelRow: {},
            tempFinancialRow: {},
            tempCommunicationRow: {},
            tempScamAppRow: {},
            tempScamWebsiteRow: {},
            tempScamVirtualIdentityRow: {},
            editingPersonnelId: null,
            editingFinancialId: null,
            editingCommunicationId: null,
            editingScamAppId: null,
            editingScamWebsiteId: null,
            editingScamVirtualIdentityId: null,
            basicInfoCollapsed: false,
            briefDescriptionCollapsed: false,
            evidenceCollapsed: false,
            personnelCollapsed: false,
            financialFlowsCollapsed: false,
            communicationFlowsCollapsed: false,
            networkFlowsCollapsed: false,
            personnelColumns: [
                // ... 人员流表格列配置，与之前相同 ...
                {
                    title: '姓名',
                    key: 'name',
                    minWidth: 100,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingPersonnelId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.name),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempPersonnelRow.name : row.name,
                                    placeholder: '请输入姓名',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempPersonnelRow, 'name', value);
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
                        const isEditing = row.id === this.editingPersonnelId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.gender),
                            h('Select', {
                                props: {
                                    value: isEditing ? this.tempPersonnelRow.gender : row.gender,
                                    placeholder: '请选择',
                                    size: 'small',
                                    transfer: true
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'on-change': (value) => {
                                        this.$set(this.tempPersonnelRow, 'gender', value);
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
                        const isEditing = row.id === this.editingPersonnelId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.age),
                            h('InputNumber', {
                                props: {
                                    value: isEditing ? this.tempPersonnelRow.age : row.age,
                                    min: 0,
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'on-change': (value) => {
                                        this.$set(this.tempPersonnelRow, 'age', value);
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
                        const isEditing = row.id === this.editingPersonnelId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.idNumber),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempPersonnelRow.idNumber : row.idNumber,
                                    placeholder: '请输入证件号码',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempPersonnelRow, 'idNumber', value);
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
                        const isEditing = row.id === this.editingPersonnelId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.occupation),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempPersonnelRow.occupation : row.occupation,
                                    placeholder: '请输入职业',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempPersonnelRow, 'occupation', value);
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
                        const isEditing = row.id === this.editingPersonnelId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.address),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempPersonnelRow.address : row.address,
                                    placeholder: '请输入住址',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempPersonnelRow, 'address', value);
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
                        const isEditing = row.id === this.editingPersonnelId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.phoneNumber),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempPersonnelRow.phoneNumber : row.phoneNumber,
                                    placeholder: '请输入联系电话',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempPersonnelRow, 'phoneNumber', value);
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
                        const isEditing = row.id === this.editingPersonnelId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.description),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempPersonnelRow.description : row.description,
                                    placeholder: '请输入描述',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempPersonnelRow, 'description', value);
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
                        const isEditing = row.id === this.editingPersonnelId;
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
                                            this.handleSavePersonnelRow(row.id);
                                        } else {
                                            this.handleEditPersonnelRow(row.id);
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
            ],
            financialFlowsColumns: [
                // ... 资金流表格列配置，与之前相同 ...
                {
                    title: '转账类型',
                    key: 'type',
                    width: 100,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingFinancialId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.type),
                            h('Select', {
                                props: {
                                    value: isEditing ? this.tempFinancialRow.type : row.type,
                                    placeholder: '请选择',
                                    size: 'small',
                                    transfer: true
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'on-change': (value) => {
                                        this.$set(this.tempFinancialRow, 'type', value);
                                    }
                                }
                            }, [
                                h('Option', { props: { value: '转入' } }, '转入'),
                                h('Option', { props: { value: '转出' } }, '转出')
                            ])
                        ]);
                    }
                },
                {
                    title: '转账时间',
                    key: 'time',
                    minWidth: 170,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingFinancialId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.time),
                            h('DatePicker', {
                                props: {
                                    value: isEditing ? this.tempFinancialRow.time : row.time,
                                    type: 'datetime',
                                    placeholder: '请选择时间',
                                    size: 'small',
                                    transfer: true
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'on-change': (value) => {
                                        this.$set(this.tempFinancialRow, 'time', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '转账金额',
                    key: 'amount',
                    width: 120,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingFinancialId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.amount),
                            h('InputNumber', {
                                props: {
                                    value: isEditing ? this.tempFinancialRow.amount : row.amount,
                                    min: 0,
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'on-change': (value) => {
                                        this.$set(this.tempFinancialRow, 'amount', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '受害人转账方式',
                    key: 'victimPaymentMethod',
                    minWidth: 150,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingFinancialId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.victimPaymentMethod),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempFinancialRow.victimPaymentMethod : row.victimPaymentMethod,
                                    placeholder: '请输入方式',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempFinancialRow, 'victimPaymentMethod', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '受害人转账卡号/账号',
                    key: 'victimCardNumber',
                    minWidth: 200,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingFinancialId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.victimCardNumber),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempFinancialRow.victimCardNumber : row.victimCardNumber,
                                    placeholder: '请输入卡号/账号',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempFinancialRow, 'victimCardNumber', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '受害人开户名',
                    key: 'victimAccountName',
                    minWidth: 120,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingFinancialId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.victimAccountName),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempFinancialRow.victimAccountName : row.victimAccountName,
                                    placeholder: '请输入开户名',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempFinancialRow, 'victimAccountName', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '受害人开户行/平台',
                    key: 'victimBank',
                    minWidth: 150,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingFinancialId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.victimBank),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempFinancialRow.victimBank : row.victimBank,
                                    placeholder: '请输入银行/平台',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempFinancialRow, 'victimBank', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '骗子收款方式',
                    key: 'suspectCollectionMethod',
                    minWidth: 150,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingFinancialId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.suspectCollectionMethod),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempFinancialRow.suspectCollectionMethod : row.suspectCollectionMethod,
                                    placeholder: '请输入方式',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempFinancialRow, 'suspectCollectionMethod', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '骗子收款卡号/账号',
                    key: 'suspectCardNumber',
                    minWidth: 200,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingFinancialId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.suspectCardNumber),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempFinancialRow.suspectCardNumber : row.suspectCardNumber,
                                    placeholder: '请输入卡号/账号',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempFinancialRow, 'suspectCardNumber', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '骗子开户行/平台',
                    key: 'suspectBank',
                    minWidth: 150,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingFinancialId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.suspectBank),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempFinancialRow.suspectBank : row.suspectBank,
                                    placeholder: '请输入银行/平台',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempFinancialRow, 'suspectBank', value);
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
                        const isEditing = row.id === this.editingFinancialId;
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
                                            this.handleSaveFinancialRow(row.id);
                                        } else {
                                            this.handleEditFinancialRow(row.id);
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
                                        this.handleDeleteFinancialRow(row.id);
                                    }
                                }
                            }, '删除')
                        ]);
                    }
                }
            ],
            communicationFlowsColumns: [
                // ... 通讯流表格列配置，与之前相同 ...
                {
                    title: '通讯号码',
                    key: 'phoneNumber',
                    minWidth: 150,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingCommunicationId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.phoneNumber),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempCommunicationRow.phoneNumber : row.phoneNumber,
                                    placeholder: '请输入号码',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempCommunicationRow, 'phoneNumber', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '归属人',
                    key: 'ownerType',
                    width: 100,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingCommunicationId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.ownerType),
                            h('Select', {
                                props: {
                                    value: isEditing ? this.tempCommunicationRow.ownerType : row.ownerType,
                                    placeholder: '请选择',
                                    size: 'small',
                                    transfer: true
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'on-change': (value) => {
                                        this.$set(this.tempCommunicationRow, 'ownerType', value);
                                    }
                                }
                            }, [
                                h('Option', { props: { value: '受害人' } }, '受害人'),
                                h('Option', { props: { value: '嫌疑人' } }, '嫌疑人'),
                                h('Option', { props: { value: '其他' } }, '其他')
                            ])
                        ]);
                    }
                },
                {
                    title: '号码类型',
                    key: 'numberType',
                    width: 120,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingCommunicationId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.numberType),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempCommunicationRow.numberType : row.numberType,
                                    placeholder: '请输入类型',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempCommunicationRow, 'numberType', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '机主姓名',
                    key: 'ownerName',
                    minWidth: 100,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingCommunicationId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.ownerName),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempCommunicationRow.ownerName : row.ownerName,
                                    placeholder: '请输入姓名',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempCommunicationRow, 'ownerName', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '归属地',
                    key: 'location',
                    minWidth: 150,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingCommunicationId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.location),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempCommunicationRow.location : row.location,
                                    placeholder: '请输入归属地',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempCommunicationRow, 'location', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '涉案环节',
                    key: 'involvementStage',
                    minWidth: 150,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingCommunicationId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.involvementStage),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempCommunicationRow.involvementStage : row.involvementStage,
                                    placeholder: '请输入涉案环节',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempCommunicationRow, 'involvementStage', value);
                                    }
                                }
                            })
                        ]);
                    }
                },
                {
                    title: '当前位置',
                    key: 'currentLocation',
                    minWidth: 150,
                    render: (h, params) => {
                        const row = params.row;
                        const isEditing = row.id === this.editingCommunicationId;
                        return h('div', [
                            h('span', {
                                style: {
                                    display: isEditing ? 'none' : 'inline-block'
                                }
                            }, row.currentLocation),
                            h('Input', {
                                props: {
                                    value: isEditing ? this.tempCommunicationRow.currentLocation : row.currentLocation,
                                    placeholder: '请输入当前位置',
                                    size: 'small'
                                },
                                style: {
                                    display: isEditing ? 'inline-block' : 'none'
                                },
                                on: {
                                    'input': (value) => {
                                        this.$set(this.tempCommunicationRow, 'currentLocation', value);
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
                        const isEditing = row.id === this.editingCommunicationId;
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
                                            this.handleSaveCommunicationRow(row.id);
                                        } else {
                                            this.handleEditCommunicationRow(row.id);
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
                                        this.handleDeleteCommunicationRow(row.id);
                                    }
                                }
                            }, '删除')
                        ]);
                    }
                }
            ],
            scamAppColumns: [
                {
                    title: 'App名称',
                    key: 'appName',
                    minWidth: 120,
                    render: (h, params) => this.renderInput(h, params, 'scamApp', 'appName', '请输入名称')
                },
                {
                    title: '包名',
                    key: 'packageName',
                    minWidth: 150,
                    render: (h, params) => this.renderInput(h, params, 'scamApp', 'packageName', '请输入包名')
                },
                {
                    title: 'MD5签名',
                    key: 'md5Signature',
                    minWidth: 200,
                    render: (h, params) => this.renderInput(h, params, 'scamApp', 'md5Signature', '请输入MD5签名')
                },
                {
                    title: '下载链接',
                    key: 'downloadLink',
                    minWidth: 200,
                    render: (h, params) => this.renderInput(h, params, 'scamApp', 'downloadLink', '请输入下载链接')
                },
                {
                    title: '注册人',
                    key: 'registrant',
                    minWidth: 120,
                    render: (h, params) => this.renderInput(h, params, 'scamApp', 'registrant', '请输入注册人')
                },
                {
                    title: 'IP位置',
                    key: 'ipLocation',
                    minWidth: 150,
                    render: (h, params) => this.renderInput(h, params, 'scamApp', 'ipLocation', '请输入IP位置')
                },
                {
                    title: '操作',
                    key: 'action',
                    width: 140,
                    align: 'center',
                    fixed: 'right',
                    render: (h, params) => this.renderActionButtons(h, params, 'scamApp')
                }
            ],
            scamWebsiteColumns: [
                {
                    title: '网站名称',
                    key: 'websiteName',
                    minWidth: 120,
                    render: (h, params) => this.renderInput(h, params, 'scamWebsite', 'websiteName', '请输入网站名称')
                },
                {
                    title: '网址',
                    key: 'url',
                    minWidth: 200,
                    render: (h, params) => this.renderInput(h, params, 'scamWebsite', 'url', '请输入网址')
                },
                {
                    title: '注册人',
                    key: 'registrant',
                    minWidth: 120,
                    render: (h, params) => this.renderInput(h, params, 'scamWebsite', 'registrant', '请输入注册人')
                },
                {
                    title: 'IP位置',
                    key: 'ipLocation',
                    minWidth: 150,
                    render: (h, params) => this.renderInput(h, params, 'scamWebsite', 'ipLocation', '请输入IP位置')
                },
                {
                    title: '操作',
                    key: 'action',
                    width: 140,
                    align: 'center',
                    fixed: 'right',
                    render: (h, params) => this.renderActionButtons(h, params, 'scamWebsite')
                }
            ],
            scamVirtualIdentityColumns: [
                {
                    title: '虚拟身份号码',
                    key: 'identityNumber',
                    minWidth: 150,
                    render: (h, params) => this.renderInput(h, params, 'scamVirtualIdentity', 'identityNumber', '请输入虚拟身份号码')
                },
                {
                    title: '号码类型',
                    key: 'numberType',
                    minWidth: 120,
                    render: (h, params) => this.renderInput(h, params, 'scamVirtualIdentity', 'numberType', '请输入号码类型')
                },
                {
                    title: '涉案环节',
                    key: 'involvementStage',
                    minWidth: 150,
                    render: (h, params) => this.renderInput(h, params, 'scamVirtualIdentity', 'involvementStage', '请输入涉案环节')
                },
                {
                    title: '操作',
                    key: 'action',
                    width: 140,
                    align: 'center',
                    fixed: 'right',
                    render: (h, params) => this.renderActionButtons(h, params, 'scamVirtualIdentity')
                }
            ]
        };
    },
    computed: {
        currentPersonnelData() {
            return this.personnelFlows[this.personnelType];
        }
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
                personnelFlows: {
                    suspect: [
                        { name: '张三', gender: '男', age: 35, idNumber: '320000199001011234', occupation: '无业', address: '某市某区某路101号', phoneNumber: '13812345678', description: '主要嫌疑人' },
                        { name: '李四', gender: '男', age: 40, idNumber: '320000198502025678', occupation: '公司职员', address: '某市某区某路102号', phoneNumber: '13987654321', description: '同案犯，负责资金转移' }
                    ],
                    victim: [
                        { name: '王小明', gender: '男', age: 28, idNumber: '440000199705054321', occupation: '学生', address: '某市某区某大学宿舍', phoneNumber: '13500001111', description: '受害人，被骗金额10万元' }
                    ]
                },
                financialFlows: [
                    { id: 1, type: '转出', time: '2025-07-30 15:05:00', amount: 10000, victimPaymentMethod: '银行卡', victimCardNumber: '6227xxxxxxxxxxxx1234', victimAccountName: '王小明', victimBank: '招商银行', suspectCollectionMethod: '支付宝', suspectCardNumber: '13812345678', suspectBank: '支付宝' },
                    { id: 2, type: '转出', time: '2025-07-30 16:10:00', amount: 5000, victimPaymentMethod: '支付宝', victimCardNumber: '13500001111', victimAccountName: '王小明', victimBank: '支付宝', suspectCollectionMethod: '银行卡', suspectCardNumber: '6228xxxxxxxxxxxx5678', suspectBank: '建设银行' }
                ],
                communicationFlows: [
                    { id: 1, phoneNumber: '13812345678', ownerType: '嫌疑人', numberType: '移动电话', ownerName: '张三', location: '某市某区', involvementStage: '诈骗实施阶段', currentLocation: '某市某区某地' },
                    { id: 2, phoneNumber: '13500001111', ownerType: '受害人', numberType: '移动电话', ownerName: '王小明', location: '某市某区', involvementStage: '被骗阶段', currentLocation: '某市某区某大学' },
                ],
                scamApps: [
                    { id: 1, appName: '理财宝', packageName: 'com.licai.app', md5Signature: 'd41d8cd98f00b204e9800998ecf8427e', downloadLink: 'http://scam-app.com/download', registrant: '李四', ipLocation: '海外' }
                ],
                scamWebsites: [
                    { id: 1, websiteName: '投资大师', url: 'http://www.touzidashi.com', registrant: '张三', ipLocation: '某省某市' }
                ],
                scamVirtualIdentities: [
                    { id: 1, identityNumber: '1234567890', numberType: 'QQ号', involvementStage: '与受害人沟通' }
                ],
                files: [
                    { name: '调证函-银行.pdf', status: 'finished', url: 'http://example.com/file1.pdf' },
                    { name: '嫌疑人笔录.doc', status: 'finished', url: 'http://example.com/file2.doc' }
                ]
            };

            setTimeout(() => {
                this.caseInfo = mockData;
                this.briefDescription = mockData.briefDescription;
                this.personnelFlows.suspect = mockData.personnelFlows.suspect.map((item, index) => ({ ...item, id: index + 1 }));
                this.personnelFlows.victim = mockData.personnelFlows.victim.map((item, index) => ({ ...item, id: index + 1 }));
                this.financialFlows = mockData.financialFlows;
                this.communicationFlows = mockData.communicationFlows;
                this.scamApps = mockData.scamApps;
                this.scamWebsites = mockData.scamWebsites;
                this.scamVirtualIdentities = mockData.scamVirtualIdentities;
                this.fileList = mockData.files;
                this.personnelTableKey++;
                this.financialTableKey++;
                this.communicationTableKey++;
                this.scamAppTableKey++;
                this.scamWebsiteTableKey++;
                this.scamVirtualIdentityTableKey++;
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
        toggleFinancialFlows() {
            this.financialFlowsCollapsed = !this.financialFlowsCollapsed;
        },
        toggleCommunicationFlows() {
            this.communicationFlowsCollapsed = !this.communicationFlowsCollapsed;
        },
        toggleNetworkFlows() {
            this.networkFlowsCollapsed = !this.networkFlowsCollapsed;
        },
        // 渲染方法，减少重复代码
        renderInput(h, params, type, key, placeholder) {
            const row = params.row;
            const isEditing = row.id === this[`editing${this.capitalize(type)}Id`];
            const tempRow = this[`temp${this.capitalize(type)}Row`];
            return h('div', [
                h('span', {
                    style: {
                        display: isEditing ? 'none' : 'inline-block'
                    }
                }, row[key]),
                h('Input', {
                    props: {
                        value: isEditing ? tempRow[key] : row[key],
                        placeholder: placeholder,
                        size: 'small'
                    },
                    style: {
                        display: isEditing ? 'inline-block' : 'none'
                    },
                    on: {
                        'input': (value) => {
                            this.$set(tempRow, key, value);
                        }
                    }
                })
            ]);
        },
        renderActionButtons(h, params, type) {
            const row = params.row;
            const isEditing = row.id === this[`editing${this.capitalize(type)}Id`];
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
                                this[`handleSave${this.capitalize(type)}Row`](row.id);
                            } else {
                                this[`handleEdit${this.capitalize(type)}Row`](row.id);
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
                            this[`handleDelete${this.capitalize(type)}Row`](row.id);
                        }
                    }
                }, '删除')
            ]);
        },
        capitalize(str) {
            return str.charAt(0).toUpperCase() + str.slice(1);
        },
        // 人员流相关方法
        handlePersonnelTypeChange(type) {
            this.personnelType = type;
            this.editingPersonnelId = null;
            this.tempPersonnelRow = {};
            this.personnelTableKey++;
        },
        handleEditPersonnelRow(id) {
            this.editingPersonnelId = id;
            const index = this.currentPersonnelData.findIndex(p => p.id === id);
            if (index !== -1) {
                this.tempPersonnelRow = JSON.parse(JSON.stringify(this.currentPersonnelData[index]));
            }
        },
        handleSavePersonnelRow(id) {
            const index = this.currentPersonnelData.findIndex(p => p.id === id);
            if (index !== -1) {
                this.$set(this.currentPersonnelData, index, { ...this.tempPersonnelRow, id: id });
                this.tempPersonnelRow = {};
                this.editingPersonnelId = null;
                this.$Message.success('人员信息保存成功');
            }
        },
        handleAddPersonnel() {
            const currentData = this.currentPersonnelData;
            const newId = currentData.length > 0 ? Math.max(...currentData.map(p => p.id)) + 1 : 1;
            const newPersonnel = {
                id: newId, name: '', gender: '', age: null, idNumber: '', occupation: '', address: '', phoneNumber: '', description: ''
            };
            this.personnelFlows[this.personnelType].push(newPersonnel);
            this.tempPersonnelRow = { ...newPersonnel };
            this.editingPersonnelId = newId;
            this.personnelTableKey++;
        },
        handleDeletePersonnel(id) {
            this.$Modal.confirm({
                title: '确认删除',
                content: '确定要删除该人员信息吗？',
                onOk: () => {
                    const currentData = this.currentPersonnelData;
                    const index = currentData.findIndex(p => p.id === id);
                    if (index !== -1) {
                        currentData.splice(index, 1);
                        if (this.editingPersonnelId === id) {
                            this.editingPersonnelId = null;
                            this.tempPersonnelRow = {};
                        }
                        this.$Message.success('人员信息删除成功');
                        this.personnelTableKey++;
                    }
                }
            });
        },
        // 资金流相关方法
        handleAddFinancialFlow() {
            const newId = this.financialFlows.length > 0 ? Math.max(...this.financialFlows.map(f => f.id)) + 1 : 1;
            const newFlow = {
                id: newId, type: '', time: '', amount: null, victimPaymentMethod: '', victimCardNumber: '', victimAccountName: '', victimBank: '', suspectCollectionMethod: '', suspectCardNumber: '', suspectBank: ''
            };
            this.financialFlows.push(newFlow);
            this.tempFinancialRow = { ...newFlow };
            this.editingFinancialId = newId;
            this.financialTableKey++;
        },
        handleEditFinancialRow(id) {
            this.editingFinancialId = id;
            const index = this.financialFlows.findIndex(f => f.id === id);
            if (index !== -1) {
                this.tempFinancialRow = JSON.parse(JSON.stringify(this.financialFlows[index]));
            }
        },
        handleSaveFinancialRow(id) {
            const index = this.financialFlows.findIndex(f => f.id === id);
            if (index !== -1) {
                this.$set(this.financialFlows, index, { ...this.tempFinancialRow, id: id });
                this.tempFinancialRow = {};
                this.editingFinancialId = null;
                this.$Message.success('资金流信息保存成功');
            }
        },
        handleDeleteFinancialRow(id) {
            this.$Modal.confirm({
                title: '确认删除',
                content: '确定要删除该资金流信息吗？',
                onOk: () => {
                    const index = this.financialFlows.findIndex(f => f.id === id);
                    if (index !== -1) {
                        this.financialFlows.splice(index, 1);
                        if (this.editingFinancialId === id) {
                            this.editingFinancialId = null;
                            this.tempFinancialRow = {};
                        }
                        this.$Message.success('资金流信息删除成功');
                        this.financialTableKey++;
                    }
                }
            });
        },
        // 通讯流相关方法
        handleAddCommunicationFlow() {
            const newId = this.communicationFlows.length > 0 ? Math.max(...this.communicationFlows.map(c => c.id)) + 1 : 1;
            const newFlow = {
                id: newId, phoneNumber: '', ownerType: '', numberType: '', ownerName: '', location: '', involvementStage: '', currentLocation: ''
            };
            this.communicationFlows.push(newFlow);
            this.tempCommunicationRow = { ...newFlow };
            this.editingCommunicationId = newId;
            this.communicationTableKey++;
        },
        handleEditCommunicationRow(id) {
            this.editingCommunicationId = id;
            const index = this.communicationFlows.findIndex(c => c.id === id);
            if (index !== -1) {
                this.tempCommunicationRow = JSON.parse(JSON.stringify(this.communicationFlows[index]));
            }
        },
        handleSaveCommunicationRow(id) {
            const index = this.communicationFlows.findIndex(c => c.id === id);
            if (index !== -1) {
                this.$set(this.communicationFlows, index, { ...this.tempCommunicationRow, id: id });
                this.tempCommunicationRow = {};
                this.editingCommunicationId = null;
                this.$Message.success('通讯流信息保存成功');
            }
        },
        handleDeleteCommunicationRow(id) {
            this.$Modal.confirm({
                title: '确认删除',
                content: '确定要删除该通讯流信息吗？',
                onOk: () => {
                    const index = this.communicationFlows.findIndex(c => c.id === id);
                    if (index !== -1) {
                        this.communicationFlows.splice(index, 1);
                        if (this.editingCommunicationId === id) {
                            this.editingCommunicationId = null;
                            this.tempCommunicationRow = {};
                        }
                        this.$Message.success('通讯流信息删除成功');
                        this.communicationTableKey++;
                    }
                }
            });
        },
        // 网络流 - 涉诈App相关方法
        handleAddScamApp() {
            const newId = this.scamApps.length > 0 ? Math.max(...this.scamApps.map(a => a.id)) + 1 : 1;
            const newApp = {
                id: newId, appName: '', packageName: '', md5Signature: '', downloadLink: '', registrant: '', ipLocation: ''
            };
            this.scamApps.push(newApp);
            this.tempScamAppRow = { ...newApp };
            this.editingScamAppId = newId;
            this.scamAppTableKey++;
        },
        handleEditScamAppRow(id) {
            this.editingScamAppId = id;
            const index = this.scamApps.findIndex(a => a.id === id);
            if (index !== -1) {
                this.tempScamAppRow = JSON.parse(JSON.stringify(this.scamApps[index]));
            }
        },
        handleSaveScamAppRow(id) {
            const index = this.scamApps.findIndex(a => a.id === id);
            if (index !== -1) {
                this.$set(this.scamApps, index, { ...this.tempScamAppRow, id: id });
                this.tempScamAppRow = {};
                this.editingScamAppId = null;
                this.$Message.success('涉诈App信息保存成功');
            }
        },
        handleDeleteScamAppRow(id) {
            this.$Modal.confirm({
                title: '确认删除',
                content: '确定要删除该涉诈App信息吗？',
                onOk: () => {
                    const index = this.scamApps.findIndex(a => a.id === id);
                    if (index !== -1) {
                        this.scamApps.splice(index, 1);
                        if (this.editingScamAppId === id) {
                            this.editingScamAppId = null;
                            this.tempScamAppRow = {};
                        }
                        this.$Message.success('涉诈App信息删除成功');
                        this.scamAppTableKey++;
                    }
                }
            });
        },
        // 网络流 - 涉诈网站相关方法
        handleAddScamWebsite() {
            const newId = this.scamWebsites.length > 0 ? Math.max(...this.scamWebsites.map(w => w.id)) + 1 : 1;
            const newWebsite = {
                id: newId, websiteName: '', url: '', registrant: '', ipLocation: ''
            };
            this.scamWebsites.push(newWebsite);
            this.tempScamWebsiteRow = { ...newWebsite };
            this.editingScamWebsiteId = newId;
            this.scamWebsiteTableKey++;
        },
        handleEditScamWebsiteRow(id) {
            this.editingScamWebsiteId = id;
            const index = this.scamWebsites.findIndex(w => w.id === id);
            if (index !== -1) {
                this.tempScamWebsiteRow = JSON.parse(JSON.stringify(this.scamWebsites[index]));
            }
        },
        handleSaveScamWebsiteRow(id) {
            const index = this.scamWebsites.findIndex(w => w.id === id);
            if (index !== -1) {
                this.$set(this.scamWebsites, index, { ...this.tempScamWebsiteRow, id: id });
                this.tempScamWebsiteRow = {};
                this.editingScamWebsiteId = null;
                this.$Message.success('涉诈网站信息保存成功');
            }
        },
        handleDeleteScamWebsiteRow(id) {
            this.$Modal.confirm({
                title: '确认删除',
                content: '确定要删除该涉诈网站信息吗？',
                onOk: () => {
                    const index = this.scamWebsites.findIndex(w => w.id === id);
                    if (index !== -1) {
                        this.scamWebsites.splice(index, 1);
                        if (this.editingScamWebsiteId === id) {
                            this.editingScamWebsiteId = null;
                            this.tempScamWebsiteRow = {};
                        }
                        this.$Message.success('涉诈网站信息删除成功');
                        this.scamWebsiteTableKey++;
                    }
                }
            });
        },
        // 网络流 - 涉诈虚拟身份相关方法
        handleAddScamVirtualIdentity() {
            const newId = this.scamVirtualIdentities.length > 0 ? Math.max(...this.scamVirtualIdentities.map(v => v.id)) + 1 : 1;
            const newIdentity = {
                id: newId, identityNumber: '', numberType: '', involvementStage: ''
            };
            this.scamVirtualIdentities.push(newIdentity);
            this.tempScamVirtualIdentityRow = { ...newIdentity };
            this.editingScamVirtualIdentityId = newId;
            this.scamVirtualIdentityTableKey++;
        },
        handleEditScamVirtualIdentityRow(id) {
            this.editingScamVirtualIdentityId = id;
            const index = this.scamVirtualIdentities.findIndex(v => v.id === id);
            if (index !== -1) {
                this.tempScamVirtualIdentityRow = JSON.parse(JSON.stringify(this.scamVirtualIdentities[index]));
            }
        },
        handleSaveScamVirtualIdentityRow(id) {
            const index = this.scamVirtualIdentities.findIndex(v => v.id === id);
            if (index !== -1) {
                this.$set(this.scamVirtualIdentities, index, { ...this.tempScamVirtualIdentityRow, id: id });
                this.tempScamVirtualIdentityRow = {};
                this.editingScamVirtualIdentityId = null;
                this.$Message.success('涉诈虚拟身份信息保存成功');
            }
        },
        handleDeleteScamVirtualIdentityRow(id) {
            this.$Modal.confirm({
                title: '确认删除',
                content: '确定要删除该涉诈虚拟身份信息吗？',
                onOk: () => {
                    const index = this.scamVirtualIdentities.findIndex(v => v.id === id);
                    if (index !== -1) {
                        this.scamVirtualIdentities.splice(index, 1);
                        if (this.editingScamVirtualIdentityId === id) {
                            this.editingScamVirtualIdentityId = null;
                            this.tempScamVirtualIdentityRow = {};
                        }
                        this.$Message.success('涉诈虚拟身份信息删除成功');
                        this.scamVirtualIdentityTableKey++;
                    }
                }
            });
        },
        // 调证和案件操作
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