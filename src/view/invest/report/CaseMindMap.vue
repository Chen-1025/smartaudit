<template>
    <Card shadow class="case-map-outer">
        <div class="zoom-box">
            <div class="zoom-wrapper">
                <button class="zoom-button" @click="scale('down')">
                    <Icon type="md-remove" :size="14" color="#fff" />
                </button>
                <span class="zoom-number">{{ zoom }}%</span>
                <button class="zoom-button" @click="scale('up')">
                    <Icon type="md-add" :size="14" color="#fff" />
                </button>
            </div>
        </div>

        <div class="button-group">
            <Button @click="expandAllNodes">一键展开</Button>
            <Button @click="collapseAllNodes">一键折叠</Button>
            <Button @click="exportToImage">导出图片</Button>
        </div>

        <div ref="dragWrapper" class="view-box" @mousedown="mousedownView" @contextmenu="handleDocumentContextmenu">
            <div class="org-tree-wrapper" :style="orgTreeStyle">
                <v-org-tree v-if="data" :data="data" :node-render="nodeRender" :expand-all="expandAll" horizontal="true"
                    @on-node-click="handleNodeClick" collapsable></v-org-tree>
            </div>
        </div>
    </Card>
</template>

<script>
import { Card, Icon, Dropdown, DropdownMenu, DropdownItem, Button, Message } from 'iview';
import { on, off } from '@/libs/tools';
import html2canvas from 'html2canvas';

const menuList = [
    { key: 'viewDetail', label: '查看详情' },
    { key: 'addClue', label: '新增线索' },
    { key: 'markImportant', label: '标记重要' },
    { key: 'linkPerson', label: '关联人员' }
];

export default {
    name: 'CaseMap',
    components: { Card, Icon, Dropdown, DropdownMenu, DropdownItem, Button },
    data() {
        return {
            data: null,
            zoom: 100,
            step: 20,
            min: 10,
            max: 200,
            orgTreeOffsetLeft: 0,
            orgTreeOffsetTop: 0,
            initPageX: 0,
            initPageY: 0,
            oldMarginLeft: 0,
            oldMarginTop: 0,
            canMove: false,
            currentContextMenuId: '',
            selectedNodeId: '',
            expandAll: true // 新增，控制所有节点的展开/折叠
        };
    },
    computed: {
        zoomHandled() {
            return this.zoom / 100;
        },
        orgTreeStyle() {
            return {
                transform: `translate(-50%, -50%) scale(${this.zoomHandled}, ${this.zoomHandled})`,
                marginLeft: `${this.orgTreeOffsetLeft}px`,
                marginTop: `${this.orgTreeOffsetTop}px`
            };
        }
    },
    methods: {
        scale(type) {
            const newZoom = this.zoom + (type === 'down' ? -this.step : this.step);
            if ((newZoom < this.min && type === 'down') || (newZoom > this.max && type === 'up')) { return; }
            this.zoom = newZoom;
        },
        mousedownView(event) {
            if (event.target.closest('.org-tree-node-label')) {
                return;
            }
            this.canMove = true;
            this.initPageX = event.pageX;
            this.initPageY = event.pageY;
            this.oldMarginLeft = this.orgTreeOffsetLeft;
            this.oldMarginTop = this.orgTreeOffsetTop;
            on(document, 'mousemove', this.mousemoveView);
            on(document, 'mouseup', this.mouseupView);
        },
        mousemoveView(event) {
            if (!this.canMove) return;
            const { pageX, pageY } = event;
            this.orgTreeOffsetLeft = this.oldMarginLeft + pageX - this.initPageX;
            this.orgTreeOffsetTop = this.oldMarginTop + pageY - this.initPageY;
        },
        mouseupView() {
            this.canMove = false;
            off(document, 'mousemove', this.mousemoveView);
            off(document, 'mouseup', this.mouseupView);
        },
        handleDocumentContextmenu() {
            this.canMove = false;
        },
        handleNodeClick(e, data, expand) {
            this.selectedNodeId = data.id;
            expand();
        },
        checkTreeClick(event) {
            if (!this.$refs.dragWrapper) return;
            const isInsideWrapper = this.$refs.dragWrapper.contains(event.target);
            if (!isInsideWrapper) {
                return;
            }
            const isNodeLabel = event.target.closest('.org-tree-node-label');
            const isNodeButton = event.target.closest('.org-tree-node-btn');
            if (!isNodeLabel && !isNodeButton) {
                event.stopPropagation();
            }
        },
        handleContextMenuClick(data, key) {
            this.closeMenu();
            switch (key) {
                case 'viewDetail': this.$Message.info(`正在查看[${data.label}]的详情`); break;
                case 'addClue': this.$Message.success(`为[${data.label}]新增线索`); break;
                case 'markImportant':
                    this.$set(data, 'isImportant', !data.isImportant);
                    this.$Message.success(data.isImportant ? `已标记[${data.label}]为重要` : `已取消[${data.label}]的重要标记`);
                    break;
                case 'linkPerson': this.$Message.info(`准备关联人员到[${data.label}]`); break;
                default: this.$Message.error('未知操作');
            }
        },
        contextmenu(data, $event) {
            let event = $event || window.event;
            event.preventDefault();
            this.currentContextMenuId = data.id;
        },
        closeMenu() { this.currentContextMenuId = ''; },
        handleDropdownClick(event) { event.stopPropagation(); },
        nodeRender(h, data) {
            const customClass = {
                'custom-org-node': true,
                'has-children-label': data.children && data.children.length,
                'long-text-node': data.id === 'description-detail',
                'selected-node': this.selectedNodeId === data.id,
                'important-node': !!data.isImportant
            };
            const nodeContent = data.id === 'description-detail'
                ? <div>{(data.label || '').split('\n').map(line => <p>{line}</p>)}</div>
                : <div><strong>{(data.label || '').split('\n').map(line => <p>{line}</p>)}</strong></div>;
            return (
                <div class={customClass} on-mousedown={event => event.stopPropagation()} on-contextmenu={this.contextmenu.bind(this, data)}>
                    {nodeContent}
                    <Dropdown trigger="custom" class="context-menu" visible={this.currentContextMenuId === data.id} nativeOn-click={this.handleDropdownClick} on-on-click={this.handleContextMenuClick.bind(this, data)} style={{ transform: `scale(${1 / this.zoomHandled}, ${1 / this.zoomHandled})` }} v-click-outside={this.closeMenu}>
                        <Dropdown-Menu slot="list">{menuList.map(item => (<Dropdown-Item name={item.key}>{item.label}</Dropdown-Item>))}</Dropdown-Menu>
                    </Dropdown>
                </div>
            );
        },
        getCaseMapData() {
            return new Promise(resolve => {
                setTimeout(() => {
                    const formattedData = this.generateTreeData();
                    resolve(formattedData);
                }, 500);
            });
        },
        generateTreeData() {
            const caseData = {
                caseNumber: 'C20250801001', caseName: '张三涉嫌电信诈骗案', alarmType: '电信诈骗', alarmTime: '2025-08-01 09:30:00', amount: 500000, occurrenceTime: '2025-07-30 15:00:00', occurrenceLocation: '某市某区某小区', investigationUnit: '某市公安局刑警大队', investigatingPersonnel: '李警官, 王警官', jurisdictionalUnit: '某市公安局', acceptanceTime: '2025-08-01 10:00:00', filingTime: '2025-08-02 14:00:00', filingUnit: '某市公安局',
                briefDescription: '该案件初步侦查结果显示，嫌疑人张三通过网络社交平台发布虚假投资信息，诱骗受害人转账。目前已有多名受害者报案，涉案金额较大。',
                personnelFlows: {
                    suspect: [
                        { name: '张三', gender: '男', age: 35, idNumber: '320000199001011234', occupation: '无业', address: '某市某区某路101号', phoneNumber: '13812345678', description: '主要嫌疑人' },
                        { name: '李四', gender: '男', age: 40, idNumber: '320000198502025678', occupation: '公司职员', address: '某市某区某路102号', phoneNumber: '13987654321', description: '同案犯，负责资金转移' },
                        { name: '赵六', gender: '男', age: 33, idNumber: '320000199203037890', occupation: '技术员', address: '某省某县某村', phoneNumber: '13677778888', description: '负责开发诈骗App' }
                    ],
                    victim: [
                        { name: '王小明', gender: '男', age: 28, idNumber: '440000199705054321', occupation: '学生', address: '某市某区某大学宿舍', phoneNumber: '13500001111', description: '受害人，被骗金额10万元' }
                    ]
                },
                communicationFlows: [
                    { id: 1, phoneNumber: '13812345678', ownerType: '嫌疑人', numberType: '移动电话', ownerName: '张三', location: '某市某区', involvementStage: '诈骗实施阶段', currentLocation: '某市某区某地' },
                    { id: 2, phoneNumber: '13500001111', ownerType: '受害人', numberType: '移动电话', ownerName: '王小明', location: '某市某区', involvementStage: '被骗阶段', currentLocation: '某市某区某大学' },
                    { id: 3, phoneNumber: '13677778888', ownerType: '嫌疑人', numberType: '联通电话', ownerName: '赵六', location: '外省', involvementStage: '技术支持', currentLocation: '外省某市' },
                ],
                scamApps: [
                    { id: 1, appName: '理财宝', packageName: 'com.licai.app', md5Signature: 'd41d8cd98f00b204e9800998ecf8427e', downloadLink: 'http://scam-app.com/download', registrant: '李四', ipLocation: '海外' },
                    { id: 2, appName: '财富+', packageName: 'com.caifu.plus', md5Signature: 'e10adc3949ba59abbe56e057f20f883e', downloadLink: 'http://caifu-plus.com/dl', registrant: '赵六', ipLocation: '本市' }
                ],
                scamWebsites: [{ id: 1, websiteName: '投资大师', url: 'http://www.touzidashi.com', registrant: '张三', ipLocation: '某省某市' }],
                scamVirtualIdentities: [{ id: 1, identityNumber: '1234567890', numberType: 'QQ号', involvementStage: '与受害人沟通' }, { id: 2, identityNumber: '@zhaoliu_scam', numberType: 'Telegram', involvementStage: '指挥洗钱' }],
                financialFlows: [
                    { id: 1, time: '2025-07-30 15:05:00', amount: 10000, victimPaymentMethod: '支付宝', victimCardNumber: 'wangxiaoming@alipay.com', victimAccountName: '王小明', suspectCollectionMethod: '微信', suspectCardNumber: 'wxid_zhangsan', suspectAccountName: '张三' },
                    { id: 2, time: '2025-07-30 16:10:00', amount: 5000, victimPaymentMethod: '银行卡', victimCardNumber: '6222000055556666', victimAccountName: '王小明', victimBank: '农业银行', cardStatus: '正常', suspectCollectionMethod: '银行卡', suspectCardNumber: '6228000011112222', suspectAccountName: '张三', suspectBank: '建设银行', cardStatusSuspect: '正常' },
                    { id: 3, time: '2025-07-31 10:00:00', amount: 20000, victimPaymentMethod: '微信', victimCardNumber: 'wxid_lisi', victimAccountName: '李四', suspectCollectionMethod: '支付宝', suspectCardNumber: 'zhangsan@alipay.com', suspectAccountName: '张三' },
                    { id: 4, time: '2025-07-31 11:30:00', amount: 8000, victimPaymentMethod: '银行卡', victimCardNumber: '6222000055556666', victimAccountName: '王小明', victimBank: '农业银行', cardStatus: '正常', suspectCollectionMethod: '银行卡', suspectCardNumber: '6227000033334444', suspectAccountName: '李四', suspectBank: '工商银行', cardStatusSuspect: '冻结' },
                    { id: 5, time: '2025-08-01 09:00:00', amount: 15000, victimPaymentMethod: '礼品卡', victimCardNumber: 'LPK-12345', victimAccountName: '王小明', suspectCollectionMethod: '平台回收', suspectCardNumber: 'Recycle-67890', suspectAccountName: '平台' },
                    { id: 6, time: '2025-08-01 14:20:00', amount: 3000, victimPaymentMethod: '二维码支付', victimCardNumber: 'QR-PAY-789', victimAccountName: '王小明', suspectCollectionMethod: '个人码', suspectCardNumber: 'wxid_zhaoliu_收款', suspectAccountName: '赵六' },
                    { id: 7, time: '2025-08-01 18:45:00', amount: 25000, victimPaymentMethod: '支付宝', victimCardNumber: 'wangxiaoming@alipay.com', victimAccountName: '王小明', suspectCollectionMethod: '银行卡', suspectCardNumber: '6228000011112222', suspectAccountName: '张三', suspectBank: '建设银行', cardStatusSuspect: '正常' },
                    { id: 8, time: '2025-08-02 10:00:00', amount: 500, victimPaymentMethod: '游戏点卡', victimCardNumber: 'GAME-CARD-XYZ', victimAccountName: '王小明', suspectCollectionMethod: '二手平台', suspectCardNumber: '2nd-hand-platform', suspectAccountName: '匿名' },
                ]
            };
            const basicInfoNode = { id: 'basic-info', label: '基本信息', children: [{ id: 'case-number', label: `案件编号：\n${caseData.caseNumber}` }, { id: 'case-name', label: `案件名称：\n${caseData.caseName}` }, { id: 'alarm-type', label: `警情类别：\n${caseData.alarmType}` }, { id: 'alarm-time', label: `报警时间：\n${caseData.alarmTime}` }, { id: 'amount', label: `涉案金额：\n${caseData.amount.toLocaleString()}` }, { id: 'occurrence-time', label: `案发时间：\n${caseData.occurrenceTime}` }, { id: 'occurrence-location', label: `案发地点：\n${caseData.occurrenceLocation}` }] };
            const briefDescriptionNode = { id: 'brief-description-node', label: '简要案情', children: [{ id: 'description-detail', label: caseData.briefDescription }] };
            const mapPersonnel = p => ({ id: `person-${p.idNumber}`, label: p.name, children: [{ id: `person-${p.idNumber}-gender`, label: `性别: ${p.gender}` }, { id: `person-${p.idNumber}-age`, label: `年龄: ${p.age}` }, { id: `person-${p.idNumber}-phone`, label: `电话: ${p.phoneNumber}` }, { id: `person-${p.idNumber}-occupation`, label: `职业: ${p.occupation}` }, { id: `person-${p.idNumber}-address`, label: `地址: ${p.address}` }, { id: `person-${p.idNumber}-desc`, label: `描述: ${p.description}` },] });
            const suspects = caseData.personnelFlows.suspect.map(mapPersonnel);
            const victims = caseData.personnelFlows.victim.map(mapPersonnel);
            const personnelFlowsNode = { id: 'personnel-flows-node', label: '人员流', children: [{ id: 'suspect-node', label: '嫌疑人', children: suspects }, { id: 'victim-node', label: '受害人', children: victims }] };
            const virtualIdentities = caseData.scamVirtualIdentities.map(v => ({ id: `v-identity-${v.id}`, label: `${v.numberType}:\n${v.identityNumber}`, children: [{ id: `v-identity-${v.id}-stage`, label: `涉案环节：\n${v.involvementStage}` }] }));
            const scamWebsites = caseData.scamWebsites.map(w => ({ id: `website-${w.id}`, label: w.websiteName, children: [{ id: `website-${w.id}-registrant`, label: `注册人: ${w.registrant}` }, { id: `website-${w.id}-url`, label: `URL: ${w.url}` }, { id: `website-${w.id}-ip`, label: `IP位置: ${w.ipLocation}` }] }));
            const scamApps = caseData.scamApps.map(a => ({ id: `app-${a.id}`, label: a.appName, children: [{ id: `app-${a.id}-pkg`, label: `包名: ${a.packageName}` }, { id: `app-${a.id}-md5`, label: `MD5: ${a.md5Signature}` }, { id: `app-${a.id}-reg`, label: `注册人: ${a.registrant}` }] }));
            const networkFlowNode = { id: 'network-flow-node', label: '网络流', children: [{ id: 'scam-identities-node', label: '涉诈虚拟身份', children: virtualIdentities }, { id: 'scam-websites-node', label: '涉诈网站', children: scamWebsites }, { id: 'scam-apps-node', label: '涉诈App', children: scamApps }] };
            const mapCommunication = c => ({ id: `comm-${c.id}`, label: `${c.numberType}:\n${c.phoneNumber}`, children: [{ id: `comm-${c.id}-owner`, label: `机主姓名: ${c.ownerName}` }, { id: `comm-${c.id}-loc`, label: `归属地: ${c.location}` }, { id: `comm-${c.id}-stage`, label: `涉案环节: ${c.involvementStage}` }, { id: `comm-${c.id}-current`, label: `当前位置: ${c.currentLocation}` }] });
            const suspectCommNodes = caseData.communicationFlows.filter(c => c.ownerType === '嫌疑人').map(mapCommunication);
            const victimCommNodes = caseData.communicationFlows.filter(c => c.ownerType === '受害人').map(mapCommunication);
            const communicationFlowsNode = { id: 'communication-flows-node', label: '通讯流', children: [{ id: 'comm-suspects', label: '嫌疑人', children: suspectCommNodes }, { id: 'comm-victims', label: '受害人', children: victimCommNodes }] };
            const flowData = { wechat: {}, alipay: {}, bank: {}, qr: { in: [], out: [] }, other: {} };
            caseData.financialFlows.forEach(tx => {
                const processEvent = (method, account, event, name, bankName, status) => {
                    if (!method || !account) return;
                    const eventType = event.type;
                    if (method === '微信' || method === '支付宝') {
                        const category = method === '微信' ? 'wechat' : 'alipay';
                        if (!flowData[category][account]) flowData[category][account] = { in: [], out: [], total: 0 };
                        flowData[category][account][eventType].push(event);
                        flowData[category][account].total += (eventType === 'in' ? tx.amount : -tx.amount);
                    } else if (method === '银行卡') {
                        if (!flowData.bank[account]) flowData.bank[account] = { in: [], out: [], total: 0, name: name, bankName: bankName, status: status };
                        flowData.bank[account][eventType].push(event);
                        flowData.bank[account].total += (eventType === 'in' ? tx.amount : -tx.amount);
                    } else if (method === '二维码支付' || method === '个人码') {
                        flowData.qr[eventType].push(event);
                    } else {
                        if (!flowData.other[method]) flowData.other[method] = { in: [], out: [] };
                        flowData.other[method][eventType].push(event);
                    }
                };
                processEvent(tx.victimPaymentMethod, tx.victimCardNumber, { type: 'out', time: tx.time, amount: tx.amount }, tx.victimAccountName, tx.victimBank, tx.cardStatus);
                processEvent(tx.suspectCollectionMethod, tx.suspectCardNumber, { type: 'in', time: tx.time, amount: tx.amount }, tx.suspectAccountName, tx.suspectBank, tx.cardStatusSuspect || '未知');
            });
            const txNode = (t, i, prefix) => ({ id: `${prefix}-${i}`, label: `转账时间：${t.time}\n转账金额：${t.amount.toLocaleString()}` });
            const buildSocialPayNodes = (dataObject) => Object.entries(dataObject).map(([account, details]) => ({ id: `flow-social-${account}`, label: `${account}\n总金额: ${details.total.toLocaleString()}`, children: [{ id: `flow-social-${account}-in`, label: '转入', children: details.in.map((t, i) => txNode(t, i, `flow-social-${account}-in`)) }, { id: `flow-social-${account}-out`, label: '转出', children: details.out.map((t, i) => txNode(t, i, `flow-social-${account}-out`)) }] }));
            const buildBankNodes = (dataObject) => Object.entries(dataObject).map(([card, details]) => ({ id: `flow-bank-${card}`, label: `户名: ${details.name}\n开户行: ${details.bankName}\n卡号: ${card}\n笔数: ${details.in.length + details.out.length}\n总金额: ${details.total.toLocaleString()}\n状态: ${details.status}`, children: [{ id: `flow-bank-${card}-in`, label: '转入', children: details.in.map((t, i) => txNode(t, i, `flow-bank-${card}-in`)) }, { id: `flow-bank-${card}-out`, label: '转出', children: details.out.map((t, i) => txNode(t, i, `flow-bank-${card}-out`)) }] }));
            const buildOtherNodes = (dataObject) => Object.entries(dataObject).map(([name, details]) => ({ id: `flow-other-${name}`, label: `方式: ${name}`, children: [{ id: `flow-other-${name}-in`, label: '转入', children: details.in.map((t, i) => txNode(t, i, `flow-other-${name}-in`)) }, { id: `flow-other-${name}-out`, label: '转出', children: details.out.map((t, i) => txNode(t, i, `flow-other-${name}-out`)) }] }));
            const buildQrNodes = (qrData) => {
                const inTotal = qrData.in.reduce((sum, tx) => sum + tx.amount, 0);
                const outTotal = qrData.out.reduce((sum, tx) => sum + tx.amount, 0);
                return [{ id: 'flow-qr-in', label: `转入\n总金额: ${inTotal.toLocaleString()}`, children: qrData.in.map((t, i) => txNode(t, i, 'flow-qr-in')) }, { id: 'flow-qr-out', label: `转出\n总金额: ${outTotal.toLocaleString()}`, children: qrData.out.map((t, i) => txNode(t, i, 'flow-qr-out')) }];
            };
            const financialFlowsNode = { id: 'financial-flows-node', label: '资金流', children: [{ id: 'flow-wechat', label: '微信', children: buildSocialPayNodes(flowData.wechat) }, { id: 'flow-alipay', label: '支付宝', children: buildSocialPayNodes(flowData.alipay) }, { id: 'flow-qrcode', label: '二维码支付', children: buildQrNodes(flowData.qr) }, { id: 'flow-bank', label: '银行转账', children: buildBankNodes(flowData.bank) }, { id: 'flow-other', label: '其他转账', children: buildOtherNodes(flowData.other) }] };

            return {
                id: 'case-root', label: `案件编号：\n${caseData.caseNumber}`, isRoot: true,
                children: [basicInfoNode, briefDescriptionNode, personnelFlowsNode, financialFlowsNode, communicationFlowsNode, networkFlowNode]
            };
        },
        loadData() {
            this.getCaseMapData().then(data => {
                this.data = data;
            }).catch(err => {
                console.error('加载案件数据失败', err);
                this.$Message.error('加载案件数据失败，请稍后重试。');
            });
        },
        // 新增的“一键展开”方法
        expandAllNodes() {
            this.expandAll = true;
            this.$Message.info('已展开所有节点');
        },
        // 新增的“一键折叠”方法
        collapseAllNodes() {
            this.expandAll = false;
            this.$Message.info('已折叠所有节点');
        },
        // 新增的“导出图片”方法
        exportToImage() {
            const el = this.$refs.dragWrapper;

            if (!el) {
                this.$Message.error('无法找到导图容器');
                return;
            }

            // 临时保存当前缩放和平移状态
            const originalTransform = el.style.transform;
            el.style.transform = ''; // 清除 transform，确保截图完整

            // 暂时将导图内容完全展开，以截取全貌
            const originalExpandAll = this.expandAll;
            this.expandAll = true;

            // 使用 nextTick 确保DOM更新后执行截图
            this.$nextTick(() => {
                html2canvas(el.querySelector('.org-tree-wrapper'), {
                    useCORS: true,
                    backgroundColor: null, // 将背景设为透明
                    scale: 2 // 提高截图分辨率
                }).then((canvas) => {
                    const link = document.createElement('a');
                    link.download = `案件导图_${this.data.label}.png`;
                    link.href = canvas.toDataURL('image/png');
                    document.body.appendChild(link);
                    link.click();
                    document.body.removeChild(link);

                    this.$Message.success('案件导图已导出为图片！');
                }).catch((error) => {
                    console.error('导出图片失败', error);
                    this.$Message.error('导出图片失败，请稍后重试。');
                }).finally(() => {
                    // 恢复之前的状态
                    el.style.transform = originalTransform;
                    this.expandAll = originalExpandAll;
                });
            });
        }
    },
    mounted() {
        this.loadData();
        document.addEventListener('click', this.checkTreeClick, true);
        on(document, 'contextmenu', this.handleDocumentContextmenu);
    },
    beforeDestroy() {
        document.removeEventListener('click', this.checkTreeClick, true);
        off(document, 'contextmenu', this.handleDocumentContextmenu);
    }
};
</script>

<style lang="less">
@wrapper: ~'case-map';

.case-map-outer {
    width: 100%;
    height: 100%;
    overflow: auto;
    position: relative;

    .zoom-box {
        position: absolute;
        right: 30px;
        bottom: 30px;
        z-index: 2;
    }

    .button-group {
        position: absolute;
        top: 30px;
        left: 30px;
        z-index: 2;

        .ivu-btn {
            margin-right: 10px;
        }
    }

    .view-box {
        position: absolute;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;
        z-index: 1;
        cursor: move;
        overflow: hidden;

        .org-tree-wrapper {
            display: inline-block;
            position: absolute;
            left: 50%;
            top: 50%;
            transition: transform 0.2s ease-out;

            .org-tree-node-label,
            .org-tree-node-children::before {
                box-shadow: none !important;
                border: none !important;
            }

            .org-tree-node-label-inner {
                padding: 0;

                .custom-org-node {
                    padding: 8px 12px;
                    background: #738699;
                    user-select: none;
                    border-radius: 4px;
                    color: #ffffff;
                    font-size: 12px;
                    font-weight: 500;
                    line-height: 1.4;
                    transition: all 0.2s ease-in-out;
                    cursor: default;
                    width: 220px;
                    min-height: 50px;
                    box-sizing: border-box;
                    display: flex;
                    flex-direction: column;
                    justify-content: center;
                    align-items: flex-start;

                    p {
                        margin: 1px 0 !important;
                        padding: 0 !important;
                        white-space: nowrap;
                        overflow: hidden;
                        text-overflow: ellipsis;
                        width: 100%;
                        text-align: left;
                    }

                    strong>p {
                        font-weight: bold;
                    }

                    strong>p:first-child {
                        white-space: normal;
                    }

                    &:hover {
                        background: #5d6c7b;
                    }

                    &.has-children-label {
                        cursor: pointer;
                    }

                    .context-menu {
                        position: absolute;
                        right: -10px;
                        bottom: 20px;
                        z-index: 10;
                    }

                    &.selected-node {
                        outline: 2px solid #40a9ff;
                        outline-offset: -1px;
                        box-shadow: none !important;
                    }

                    &.important-node {
                        background: #f5222d;

                        &:hover {
                            background: #cf1322;
                        }
                    }
                }

                .long-text-node {
                    width: 350px;
                    height: auto;
                    min-height: 50px;

                    p,
                    strong>p {
                        white-space: normal !important;
                        word-wrap: break-word !important;
                        overflow: visible !important;
                        text-overflow: initial !important;
                    }
                }
            }
        }
    }
}

.zoom-wrapper {
    .zoom-button {
        width: 20px;
        height: 20px;
        line-height: 10px;
        border-radius: 50%;
        background: rgba(157, 162, 172, 1);
        box-shadow: 0px 2px 8px 0px rgba(218, 220, 223, 0.7);
        border: none;
        cursor: pointer;
        outline: none;
        transition: all 0.1s ease-in;

        &:active {
            box-shadow: 0px 0px 2px 2px rgba(218, 220, 223, 0.2) inset;
        }

        &:hover {
            background: #1890ff;
        }
    }

    .zoom-number {
        color: #657180;
        padding: 0 8px;
        display: inline-block;
        width: 46px;
        text-align: center;
    }
}
</style>