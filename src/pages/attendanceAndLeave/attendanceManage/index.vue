<template>
    <div>
        <Card>
            <Form inline :label-width="60">
                <FormItem label="姓名">
                    <Input type="text"
                           @on-change="_inputDebounce"
                           v-model="filterOpt.userName"
                           placeholder="筛选姓名"></Input>
                </FormItem>
                <FormItem label="考勤月份">
                    <DatePicker type="month"
                                placeholder="筛选考勤月份"
                                @on-change="_monthDateChange"
                                :value="filterOpt.monthDate"></DatePicker>
                </FormItem>
                <FormItem label="部门">
                    <Input type="text"
                           @on-change="_inputDebounce"
                           v-model="filterOpt.organizeName"
                           placeholder="筛选部门"></Input>
                </FormItem>
                <FormItem label="岗位">
                    <Input type="text"
                           @on-change="_inputDebounce"
                           v-model="filterOpt.postName"
                           placeholder="筛选岗位"></Input>
                </FormItem>
                <FormItem label="审核状态">
                    <Select v-model="filterOpt.kqstates"
                            clearable
                            @on-change="_filterResultHandler"
                            placeholder="筛选状态"
                            style="width: 100px">
                        <Option value="2">审核完毕</Option>
                        <Option value="1">未审核</Option>
                    </Select>
                </FormItem>
                <FormItem>
                    <ButtonGroup>
                        <Button type="ghost" @click="importModalFlag = true">
                            <Icon type="ios-cloud-upload-outline"></Icon>
                            导入
                        </Button>
                        <Button type="ghost" @click="exportModalFlag = true">
                            <a id="hrefToExportTable" style="postion: absolute;left: -10px;top: -10px;width: 0px;height: 0px;"></a>
                            <Icon type="ios-cloud-download-outline"></Icon>
                            导出
                        </Button>
                        <Button type="warning" @click="deleteModalFlag = true" >
                            <Icon type="ios-trash-outline"></Icon>
                            删除
                        </Button>
                    </ButtonGroup>
                </FormItem>
            </Form>
            <Table :columns="postColumns"
                   :loading="tableLoading"
                   :height="tableHeight"
                   ref="attendanceTable"
                   :data="pageData.list"></Table>
            <Page :total="pageData.totalCount"
                  :current="pageData.page"
                  @on-change="_setPage"
                  @on-page-size-change="_setPageSize"
                  :page-size="pageData.pageSize"
                  placement="top"
                  show-sizer
                  show-total
                  show-elevator
                  style="margin-top: 16px;"></Page>
            <Modal v-model="importModalFlag"
                   width="400"
                   :mask-closable="false">
                <p slot="header" style="color:#495060;text-align:center;font-size: 18px">
                    <span>导入考勤表</span>
                </p>
                <Upload
                        type="drag"
                        :on-format-error="_uploadFormatErr"
                        :on-success="_uploadSuccess"
                        :on-error="_uploadFail"
                        :format="uploadOpt.format"
                        action="/oa/kq/add">
                    <div style="padding: 20px 0">
                        <Icon type="ios-cloud-upload" size="52" style="color: #3399ff"></Icon>
                        <p>点击或者拖拽文件到这里上传(后缀为.xls的文件)</p>
                    </div>
                </Upload>
                <div slot="footer"></div>
            </Modal>
            <Modal v-model="deleteModalFlag"
                   width="300"
                   :mask-closable="false">
                <p slot="header" style="color:#495060;text-align:center;font-size: 18px">
                    <span>删除考勤</span>
                </p>
                <Form >
                    <FormItem label="考勤月份">
                        <DatePicker type="month"
                                    placeholder="筛选考勤月份"
                                    @on-change="_deleteMonthChange"
                                    :value="deleteMonth"></DatePicker>
                    </FormItem>
                </Form>
                <div slot="footer">
                    <Button type="primary"
                            :loading="deleteLoading"
                            @click="_confirmDelete">
                        <span v-if="!deleteLoading">确认删除</span>
                        <span v-else>正在删除...</span>
                    </Button>
                    <Button type="ghost" style="margin-left: 8px" @click="deleteModalFlag = false">取消</Button>
                </div>
            </Modal>
            <Modal v-model="exportModalFlag"
                   width="300"
                   :mask-closable="false">
                <p slot="header" style="color:#495060;text-align:center;font-size: 18px">
                    <span>导出考勤</span>
                </p>
                <Form >
                    <FormItem label="考勤月份">
                        <DatePicker type="month"
                                    placeholder="筛选考勤月份"
                                    @on-change="_exportMonthChange"
                                    :value="exportMonth"></DatePicker>
                    </FormItem>
                </Form>
                <div slot="footer">
                    <Button type="primary"
                            :loading="exportLoading"
                            @click="_confirmExport">
                        <span v-if="!exportLoading">确认导出</span>
                        <span v-else>正在导出...</span>
                    </Button>
                    <Button type="ghost" style="margin-left: 8px" @click="exportModalFlag = false">取消</Button>
                </div>
            </Modal>
            <Modal v-model="settingModalFlag"
                   width="1200"
                   :mask-closable="false">
                <p slot="header" style="color:#495060;text-align:center;font-size: 18px">
                    <span>{{attendanceOpt.userName + ' ' + attendanceOpt.monthDate}} 考勤总汇</span>
                </p>
                <div class="">
                    <Table :columns="attendanceAllCol"
                           :loading="tableLoading2"
                           height="500"
                           :data="attendanceOpt.data"></Table>
                </div>
                <div slot="footer">
                    <Button type="primary" @click="_completeThisMonth">完成 {{attendanceOpt.userName}} 该月审核</Button>
                    <Button type="ghost" style="margin-left: 8px" @click="settingModalFlag = false">取消</Button>
                </div>
            </Modal>
            <Modal v-model="strangeModalFlag"
                   width="400"
                   :styles="{top: '160px', zIndex: 100}"
                   :mask-closable="false">
                <p slot="header" style="color:#495060;text-align:center;font-size: 18px">
                    <span>异常设置</span>
                </p>
                <Form :model="strangeSettingForm" :label-width="80">
                    <FormItem label="异常类型">
                        <Select v-model="strangeSettingForm.type" clearable>
                            <Option value="事假">事假</Option>
                            <Option value="病假">病假</Option>
                            <Option value="婚假">婚假</Option>
                            <Option value="产假" >产假</Option>
                            <Option value="年假" >年假</Option>
                            <Option value="法假">法假</Option>
                            <Option value="出差">出差</Option>
                            <Option value="调休">调休</Option>
                            <Option value="休息">休息</Option>
                            <Option value="旷工">旷工</Option>
                            <Option value="正常">正常</Option>
                            <Option value="生日假">生日假</Option>
                            <Option value="未入职">未入职</Option>
                            <Option value="丧假">丧假</Option>
                            <Option value="与排班不一致">与排班不一致</Option>
                            <Option value="无薪假">无薪假</Option>
                            <Option value="带薪假">带薪假</Option>
                            <Option value="陪护假" >陪护假</Option>
                            <Option value="取消设置">取消设置</Option>
                        </Select>
                    </FormItem>
                    <FormItem label="请假天数">
                        <Select v-model="strangeSettingForm.days" clearable>
                            <Option value="1">1天</Option>
                            <Option value="0.5">0.5天</Option>
                        </Select>
                    </FormItem>
                    <FormItem label="异常描述">
                        <Input v-model="strangeSettingForm.desc" type="textarea"
                               :autosize="{minRows: 2,maxRows: 5}"
                               placeholder=""></Input>
                    </FormItem>
                </Form>
                <div slot="footer">
                    <Button type="primary" @click="_confirmStrangeSetting">确认设置</Button>
                    <Button type="ghost" style="margin-left: 8px" @click="strangeModalFlag = false">取消</Button>
                </div>
            </Modal>
        </Card>
    </div>
</template>
<script>
    import pageMixin from '@/mixins/pageMixin';
    import moment from 'moment';
    import debounce from 'lodash/debounce';
    export default {
        name: 'attendanceManage',
        data () {
            return {
                tableLoading2: false,
                settingModalFlag: false,
                importModalFlag: false,
                exportModalFlag: false,
                deleteModalFlag: false,
                exportLoading: false,
                deleteLoading: false,
                strangeModalFlag: false,
                postFormType: 'update',
                recordId: '',
                exportMonth: moment().format('YYYY-MM'),
                deleteMonth: moment().format('YYYY-MM'),
                strangeSettingForm: {
                    type: '',
                    days: '',
                    desc: '',
                    id: ''
                },
                uploadOpt: {
                    format: ['xls']
                },
                depProps: {
                    value: 'id',
                    label: 'name'
                },
                postSettingForm: {
                    states: '',
                    name: '',
                    number: '',
                    organizename: '',
                    username: ''
                },
                filterOpt: {
                    kqstates: '',
                    userName: '',
                    monthDate: '',
                    organizeName: '',
                    postName: ''
                },
                attendanceDelData: [],
                postColumns: [
                    {
                        type: 'selection',
                        width: 60,
                        align: 'center'
                    },
                    {
                        title: '部门',
                        key: 'organizename',
                        align: 'center'
                    },
                    {
                        title: '岗位',
                        key: 'postname',
                        align: 'center'
                    },
                    {
                        title: '员工姓名',
                        key: 'user_name',
                        align: 'center'
                    },
                    {
                        title: '记录月份',
                        key: 'record_month',
                        align: 'center',
                        render: (h, params) => {
                            return h('span', moment(params.row['record_month']).format('YYYY-MM'));
                        }
                    },
                    {
                        title: '迟到',
                        key: 'late_times',
                        align: 'center',
                        width: 60

                    },
                    {
                        title: '早退',
                        key: 'leave_early',
                        align: 'center',
                        width: 60
                    },
                    {
                        title: '漏打卡',
                        key: 'forget_times',
                        align: 'center',
                        width: 80
                    },
                    {
                        title: '请假(天)',
                        key: 'leave_day',
                        align: 'center',
                        width: 80
                    },
                    {
                        title: '带薪休假(天)',
                        key: 'paid_leave_day',
                        align: 'center',
                        width: 110
                    },
                    {
                        title: '出勤(天)',
                        key: 'regular_day',
                        align: 'center',
                        width: 80
                    },
                    {
                        title: '旷工(天)',
                        key: 'absent_off_day',
                        align: 'center',
                        width: 80
                    },
                    {
                        title: '无薪假(天)',
                        key: 'without_pay_day',
                        align: 'center',
                        width: 100
                    },
                    {
                        title: '状态',
                        key: 'states',
                        align: 'center',
                        width: 120,
                        render: (h, params) => {
                            return h('Tag', {
                                props: {
                                    type: 'border',
                                    color: params.row.status === '未审核' ? 'red' : 'green'
                                }
                            }, params.row.status);
                        }
                    },
                    {
                        title: '操作',
                        width: 100,
                        render: (h, params) => {
                            let vm = this;
                            return h('div', [
                                h('Tooltip', {
                                    props: {
                                        content: '考勤设置',
                                        placement: 'top',
                                        transfer: true
                                    }
                                }, [
                                    h('Button', {
                                        props: {
                                            type: 'primary',
                                            icon: 'ios-gear',
                                            shape: 'circle'
                                        },
                                        on: {
                                            click: function () {
                                                vm._editorSetting(params.row);
                                            }
                                        }
                                    })
                                ])
                            ]);
                        }
                    }
                ],
                attendanceAllCol: [
                    {
                        title: '打卡记录',
                        key: 'kq_re',
                        width: '240',
                        align: 'center',
                        render: (h, params) => {
                            if (params.row.kq_re) {
                                let flag = params.row.exception === null || +params.row.exception === 0 || params.row.offdaytype === '正常';
                                return h('Tag', {
                                    props: {
                                        color: flag ? 'green' : 'red'
                                    },
                                    style: {
                                        fontSize: '14px'
                                    }
                                }, params.row.kq_re);
                            }
                        }
                    },
                    {
                        title: '日期',
                        key: 'k_date',
                        align: 'center',
                        width: '110'
                    },
                    {
                        title: '工种',
                        key: 'truetype',
                        align: 'center',
                        width: 80
                    },
                    {
                        title: '迟到',
                        key: 'c_count',
                        align: 'center',
                        width: 120,
                        render: (h, params) => {
                            if (+params.row.exception !== 2) {
                                let vm = this;
                                return h('InputNumber', {
                                    props: {
                                        value: params.row['c_count'],
                                        size: 'small',
                                        min: 0,
                                        editable: false
                                    },
                                    on: {
                                        'on-change' (val) {
                                            params.row['c_count'] = val;
                                            vm._setEveryStrangeNumber(val, 'late', params.row);
                                        }
                                    }
                                });
                            }
                        }
                    },
                    {
                        title: '早退',
                        key: 'z_count',
                        align: 'center',
                        width: 120,
                        render: (h, params) => {
                            if (+params.row.exception !== 2) {
                                let vm = this;
                                return h('InputNumber', {
                                    props: {
                                        value: params.row['z_count'],
                                        size: 'small',
                                        min: 0,
                                        editable: false
                                    },
                                    on: {
                                        'on-change' (val) {
                                            params.row['z_count'] = val;
                                            vm._setEveryStrangeNumber(val, 'leave_early', params.row);
                                        }
                                    }
                                });
                            }
                        }
                    },
                    {
                        title: '漏打卡',
                        key: 'l_count',
                        align: 'center',
                        width: 120,
                        render: (h, params) => {
                            if (+params.row.exception !== 2) {
                                let vm = this;
                                return h('InputNumber', {
                                    props: {
                                        value: params.row['l_count'],
                                        size: 'small',
                                        min: 0,
                                        editable: false
                                    },
                                    on: {
                                        'on-change' (val) {
                                            params.row['l_count'] = val;
                                            vm._setEveryStrangeNumber(val, 'forget', params.row);
                                        }
                                    }
                                });
                            }
                        }
                    },
                    {
                        title: '审核状态',
                        key: 'offdaytype',
                        align: 'center',
                        width: 120
                    },
                    {
                        title: '备注信息',
                        key: 'describeex',
                        align: 'center'
                    },
                    {
                        title: '操作',
                        width: 120,
                        render: (h, params) => {
                            let vm = this;
                            let flag = params.row.exception === null || +params.row.exception === 0 || params.row.offdaytype === '正常';
                            return h('div', [
                                h('Tooltip', {
                                    props: {
                                        content: '设置异常',
                                        placement: 'top',
                                        transfer: true
                                    }
                                }, [
                                    h('Button', {
                                        props: {
                                            type: 'primary',
                                            icon: 'ios-gear',
                                            shape: 'circle'
                                        },
                                        style: {
                                            marginRight: '8px'
                                        },
                                        on: {
                                            click: function () {
                                                vm._setStrangeDay(params.row);
                                            }
                                        }
                                    })
                                ]),
                                flag ? ''
                                    : h('Tooltip', {
                                        props: {
                                            content: '审核通过',
                                            placement: 'top',
                                            transfer: true
                                        }
                                    }, [
                                        h('Button', {
                                            props: {
                                                type: 'success',
                                                icon: 'checkmark-round',
                                                shape: 'circle'
                                            },
                                            on: {
                                                click: function () {
                                                    vm._setConfirmPass(params.row);
                                                }
                                            }
                                        })
                                    ])
                            ]);
                        }
                    }
                ],
                tableHeight: 500,
                attendanceOpt: {
                    userName: '',
                    monthDate: '',
                    data: []
                }
            };
        },
        mixins: [pageMixin],
        created() {
            this._getPostData();
            this._setTableHeight();
        },
        methods: {
            _confirmDelete() {
                this.deleteLoading = true;
                let data = {};
                data.date = this.deleteMonth;
                this.$http.post('/kq/delete', data).then((res) => {
                    if (res.success) {
                        this.$Message.success(this.deleteMonth + '考勤数据删除成功!');
                        this.deleteModalFlag = false;
                    }
                    console.log(res);
                }).finally(() => {
                    this.deleteLoading = false;
                });
            },
            _exportMonthChange(date) {
                this.exportMonth = date;
            },
            _deleteMonthChange(date) {
                this.deleteMonth = date;
            },
            _confirmExport() {
                this.exportLoading = true;
                let data = {};
                data.date = this.exportMonth;
                this.$http.get('/kq/export', {params: data}).then((res) => {
                    if (res.success) {
                        document.getElementById('hrefToExportTable').href = res.url;
                        document.getElementById('hrefToExportTable').download = this.exportMonth + '考勤统计表.xls';
                        document.getElementById('hrefToExportTable').click();
                        this.exportModalFlag = false;
                    }
                }).finally(() => {
                    this.exportLoading = false;
                });
            },
            _completeThisMonth() {
                let data = {};
                data.user_name = this.attendanceOpt.userName;
                data.record_month = this.attendanceOpt.monthDate;
                this.$http.post('/kq/completeExamine', data).then((res) => {
                    if (res.success) {
                        this.$Message.success('操作成功!');
                        this.settingModalFlag = false;
                    }
                });
            },
            _initAttendanceOpt() {
                this.attendanceOpt.userName = '';
                this.attendanceOpt.monthDate = '';
                this.attendanceOpt.data = [];
            },
            _initStrangeSettingForm() {
                this.strangeSettingForm.type = '';
                this.strangeSettingForm.days = '';
                this.strangeSettingForm.desc = '';
                this.strangeSettingForm.id = '';
            },
            _setConfirmPass(data) {
                let sendData = {};
                sendData.id = data.id;
                this.$http.post('/kq/completeSingle', sendData).then((res) => {
                    if (res.success) {
                        this.$Message.success('审核通过成功!');
                        this._getUserStatistic();
                    }
                });
            },
            _confirmStrangeSetting() {
                let data = {};
                data.id = this.strangeSettingForm.id;
                data.type = this.strangeSettingForm.type;
                data.quality = this.strangeSettingForm.days;
                data.text = this.strangeSettingForm.desc;
                data.date = this.attendanceOpt.monthDate;
                this.$http.post('/kq/updateOffDayType', data).then((res) => {
                    if (res.success) {
                        this.strangeModalFlag = false;
                        this.$Message.success('异常设置成功!');
                        this._getUserStatistic();
                    }
                });
            },
            _setStrangeDay(data) {
                this._initStrangeSettingForm();
                this.strangeSettingForm.id = data.id;
                let sendData = {};
                sendData.id = data.id;
                this.$http.get('/kq/setException', {params: sendData}).then((res) => {
                    this.strangeSettingForm.type = res.exceptionType ? res.exceptionType : '';
                    this.strangeSettingForm.days = res.exceptionDay ? res.exceptionDay + '' : '';
                });
                this.strangeModalFlag = true;
            },
            _setEveryStrangeNumber(val, type, data) {
                let sendData = {};
                sendData.username = data.user_name;
                sendData.id = data.id;
                sendData.number = val;
                sendData.type = type;
                this.$http.post('/kq/updateStatistic', sendData).then((res) => {
                    if (res.success) {
                        this.$Message.success('操作成功!');
                        this._getUserStatistic();
                        this._getPostData();
                    }
                });
            },
            _inputDebounce: debounce(function () {
                this._filterResultHandler();
            }, 600),
            _filterResultHandler() {
                this.initPage();
                this._getPostData();
            },
            _uploadSuccess(response, file, fileList) {
                console.log(response);
            },
            _uploadFail(error, file, fileList) {
                console.log(error);
            },
            _uploadFormatErr() {
                this.$Message.error('上传文件的后缀必须为.xls');
            },
            _setTableHeight() {
                let dm = document.body.clientHeight;
                this.tableHeight = dm - 260;
            },
            _monthDateChange(val) {
                this.filterOpt.monthDate = val;
                this._filterResultHandler();
            },
            _setPage(page) {
                this.pageData.page = page;
                this._getPostData();
            },
            _setPageSize(size) {
                this.pageData.pageSize = size;
                this._getPostData();
            },
            _editorSetting(data) {
                this._initAttendanceOpt();
                this.attendanceOpt.userName = data.user_name;
                this.attendanceOpt.monthDate = moment(data.record_month).format('YYYY-MM');
                this._getUserStatistic();
                this.settingModalFlag = true;
            },
            _getUserStatistic() {
                this.tableLoading2 = true;
                let sendData = {};
                sendData.userName = this.attendanceOpt.userName;
                sendData.recordMonth = this.attendanceOpt.monthDate;
                this.$http.get('/kq/userStatistic', {params: sendData}).then((res) => {
                    if (res.success) {
                        this.attendanceOpt.data = res.data;
                    }
                }).finally(() => {
                    this.tableLoading2 = false;
                });
            },
            _getPostData() {
                let data = {};
                data.userName = this.filterOpt.userName;
                data.monthDate = this.filterOpt.monthDate;
                data.kqstates = this.filterOpt.kqstates;
                data.organizeName = this.filterOpt.organizeName;
                data.postName = this.filterOpt.postName;
                this.getList('/kq/getStatisticList', data);
            }
        },
        components: {}
    };
</script>
