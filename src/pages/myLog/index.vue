<template>
    <div id="my-log">
        <Row :gutter="10">
            <Col :lg="14" :md="14">
                <Card style="margin-bottom: 10px;">
                    <Row type="flex" justify="center" align="middle" style="margin-bottom: 10px">
                        <span style="font-size: 18px;cursor: pointer;" >
                            <Button type="primary"
                                    shape="circle"
                                    @click.stop="_preMonth"
                                    :disabled="btnDisabled"
                                    icon="chevron-left"></Button>
                        </span>
                        <DatePicker
                                :open="datePickerFlag"
                                :value="dateData"
                                size="large"
                                @on-change="_dateChange"
                                type="month">
                            <span style="padding:0 16px;font-size: 18px;cursor: pointer;" @click="datePickerFlag = !datePickerFlag">{{dateData}}</span>
                        </DatePicker>
                        <span style="font-size: 18px;cursor: pointer;">
                             <Button type="primary"
                                     @click.stop="_nextMonth"
                                     shape="circle"
                                     :disabled="btnDisabled"
                                     icon="chevron-right"></Button>
                        </span>
                    </Row>
                    <Table :columns="columnsData"
                           :data="tableData"
                           :loading="loading"
                           :row-class-name="_rowClassName"
                           :disabled-hover="true"></Table>
                    <Modal v-model="modelFlag"
                           width="900"
                           :mask-closable="false">
                        <p slot="header" style="color:#495060;text-align:center;font-size: 18px">
                            <span>{{logDetail.date}} 日志</span>
                        </p>
                        <div class="" style="min-height: 100px;font-size: 16px;" v-html="logDetail.content" v-if="[5,6].indexOf(logDetail.type) > -1"></div>
                        <template v-if="[0,1,2,3].indexOf(logDetail.type) > -1">
                            <span style="display: inline-block;margin-right: 10px;height: 30px;line-height: 30px;vertical-align: top;">日志类型</span>
                            <Select v-model="logDetail.logType"
                                    placeholder="日志类型"
                                    style="margin-bottom: 10px;width:200px">
                                <Option v-for="item in logTypeList" :value="item.value" :key="item.value">{{ item.label }}</Option>
                            </Select>
                            <text-editor
                                    :menubar="editorOpt.menubar"
                                    :plugins="editorOpt.plugins"
                                    :editor-content="logDetail.content"
                                    :toolbar1="editorOpt.toolbar1"
                                    @content-change="_setContent"></text-editor>
                        </template>
                        <div slot="footer">
                            <div class="" v-if="[0,1,2,3].indexOf(logDetail.type) > -1">

                                <Button type="primary" :loading="submitLoading" @click="_submitLog">
                                    <span v-if="!submitLoading">提交日志</span>
                                    <span v-else>提交中...</span>
                                </Button>
                            </div>
                            <div class="" v-if="[5,6].indexOf(logDetail.type) > -1">
                                <span>指导状态:</span>
                                <Tag color="green">已指导</Tag>
                                <span>类型:</span>
                                <Tag color="blue">{{logDetail.logType1 | _returnLogType}}</Tag>
                                <span>评级结果:</span>
                                <Tag color="blue">{{logDetail.commentResult | _returnCommentResult}}</Tag>
                            </div>
                        </div>
                    </Modal>
                </Card>
            </Col>
            <Col :lg="10" :md="10">
                <Card>
                    <p  class="log-title">{{dateData}} 日志概览</p>
                    <div class="each-log-look" v-for="item in logLookList">
                        <p class="time-title">{{item.date}}</p>
                        <div class="" v-html="item.content"></div>
                    </div>
                </Card>
            </Col>
        </Row>
    </div>
</template>
<style lang="less">
    @import "../../styles/fsBase";
    #my-log {
        .log-title {
            color: @fs-title-color;
            font-size: 18px;
            font-weight: 700;
            text-align: center;
        }
        .each-log-look {
            margin-top: 10px;
            padding: 8px 0;
            &:not(:last-child) {
                border-bottom: 1px solid @fs-divider-color;
            }
            .time-title {
                color: @fs-title-color;
                font-weight: 700;
            }
        }
    }
    .ivu-table .mylog-table-row {
        td {
            .ivu-table-cell {
                position: relative;
                padding: 0;
                padding-top: 80%;
                .inner {
                    display: flex;
                    flex-direction: column;
                    justify-content: center;
                    align-items: center;
                    position: absolute;
                    left: 0;
                    top: 0;
                    width: 100%;
                    height: 100%;
                    font-size: 16px;
                    font-weight: 700;
                    transition: all 0.2s ease-in-out;
                    &:hover {
                        cursor: pointer;
                        background-color: rgba(92, 173, 255, 0.6);
                        color: #fff
                    }
                }
            }
            .fs-tag {
                display: inline-block;
                height: 22px;
                line-height: 22px;
                margin: 2px 4px 2px 0;
                padding: 0 8px;
                border-radius: 3px;
                font-size: 12px;
                vertical-align: middle;
                opacity: 1;
                overflow: hidden;
            }
        }
    }
</style>
<script>
    import moment from 'moment';
    import textEditor from '@/baseComponents/text-editor';
    import dateMixin from '@/mixins/dateMixin';
    export default {
        name: 'myLog',
        mixins: [dateMixin],
        data () {
            return {
                datePickerFlag: false,
                submitLoading: false,
                modelFlag: false,
                loading: false,
                open: true,
                btnDisabled: false,
                dateData: null,
                editorOpt: {
                    menubar: '',
                    plugins: [
                        'advlist autolink lists charmap print preview hr anchor pagebreak imagetools',
                        'searchreplace visualblocks visualchars code fullpage',
                        'insertdatetime media nonbreaking save table contextmenu directionality',
                        'emoticons paste textcolor colorpicker textpattern imagetools codesample'
                    ],
                    toolbar1: 'preview | undo redo | styleselect | forecolor backcolor bold italic | alignleft aligncenter alignright alignjustify | bullist numlist outdent indent'
                },
                logLookList: [],
                logDetail: {
                    logType: '0',
                    date: '2018-01-01',
                    type: 0,
                    logType1: '',
                    editorContent: '',
                    commentResult: '0',
                    content: ''
                },
                logTypeList: [
                    {
                        value: '0',
                        label: '出勤'
                    },
                    {
                        value: '1',
                        label: '休息'
                    }
                ],
                columnsData: [
                    {
                        title: '周日',
                        render: this._rowRender(0),
                        align: 'center'
                    },
                    {
                        title: '周一',
                        render: this._rowRender(1),
                        align: 'center'
                    },
                    {
                        title: '周二',
                        render: this._rowRender(2),
                        align: 'center'
                    },
                    {
                        title: '周三',
                        render: this._rowRender(3),
                        align: 'center'
                    },
                    {
                        title: '周四',
                        render: this._rowRender(4),
                        align: 'center'
                    },
                    {
                        title: '周五',
                        render: this._rowRender(5),
                        align: 'center'
                    },
                    {
                        title: '周六',
                        render: this._rowRender(6),
                        align: 'center'
                    }
                ],
                tableData: []
            };
        },
        watch: {
            dateData(val) {
                this._getLogInfo(val);
            }
        },
        created() {
            this.dateData = moment().format('YYYY-MM');
        },
        filters: {
            _returnCommentResult(val) {
                let text = '';
                switch (+val) {
                    case 0:
                        text = '未评价';
                        break;
                    case 1:
                        text = '优秀';
                        break;
                    case 2:
                        text = '合格';
                        break;
                    case 3:
                        text = '不合格';
                        break;
                }
                return text;
            },
            _returnLogType(val) {
                let text = '';
                switch (+val) {
                    case 0:
                        text = '出勤';
                        break;
                    case 1:
                        text = '休息';
                        break;
                }
                return text;
            }
        },
        methods: {
            _getLogInfo(ym) {
                this.loading = true;
                this.btnDisabled = true;
                this.$http.get('/journal/typeList', {params: {time: ym}}).then((res) => {
                    if (res.success) {
                        let storeArr = res.data.slice(0);
                        this._setLogList(storeArr);
                        let year = moment(ym).year();
                        let month = moment(ym).month();
                        this.tableData = this.returnDateDetail(year, month, res.data);
                    }
                }).finally(() => {
                    this.loading = false;
                    this.btnDisabled = false;
                });
            },
            _dateChange(date) {
                this.dateData = date;
                this.datePickerFlag = false;
            },
            _preMonth() {
                this.dateData = moment(this.dateData).subtract(1, 'M').format('YYYY-MM');
            },
            _nextMonth() {
                this.dateData = moment(this.dateData).add(1, 'M').format('YYYY-MM');
            },
            _rowClassName() {
                return 'mylog-table-row';
            },
            _setContent(content) {
                this.logDetail.editorContent = content;
            },
            _setSelectOpt(type) {
                if (type === 0) {
                    this.logTypeList = [ {
                        value: '1',
                        label: '休息'
                    }];
                    this.logDetail.logType = '1';
                    this.logDetail.content = '休息';
                    this.logDetail.editorContent = '休息';
                } else {
                    this.logTypeList = [
                        {
                            value: '0',
                            label: '出勤'
                        },
                        {
                            value: '1',
                            label: '休息'
                        }
                    ];
                    this.logDetail.logType = '0';
                }
            },
            _setLogList(arr) {
                this.logLookList = arr.filter((item) => {
                    return !!item.content;
                });
            },
            _logRowClick(obj) {
                console.log(obj);
                if (obj.type === 4) {
                    this.$Message.error('超过48小时不可再补写日志！');
                    return;
                }
                this.logDetail.date = obj.date;
                this.logDetail.type = obj.type;
                this.logDetail.commentResult = obj.commentResult;
                this.logDetail.content = obj.content || '';
                this.logDetail.editorContent = obj.content || '';
                this.logDetail.logType1 = obj.logType || '';
                this._setSelectOpt(obj.type);
                this.modelFlag = true;
            },
            _submitLog() {
                this.submitLoading = true;
                let data = {
                    type: this.logDetail.logType,
                    time: this.logDetail.date,
                    content: this.logDetail.editorContent
                };
                this.$http.post('/journal/addJournal', data).then((res) => {
                    if (res.success) {
                        this.$Message.success('日志提交成功！');
                        this._getLogInfo(this.dateData);
                    }
                }).finally(() => {
                    this.modelFlag = false;
                    this.submitLoading = false;
                });
            },
            _rowRender(i) {
                return (h, params) => {
                    let typeDom;
                    let type = params.row['day' + i].type;
                    if (type === 0) {
                        typeDom = '';
                    } else if (type === 1 || type === 5) {
                        typeDom = h('span', {
                            style: {
                                backgroundColor: '#19be6b',
                                color: '#fff'
                            },
                            class: ['fs-tag']
                        }, '已写');
                    } else if (type === 2) {
                        typeDom = h('span', {
                            style: {
                                backgroundColor: '#ed3f14',
                                color: '#fff'
                            },
                            class: ['fs-tag']
                        }, '补写');
                    } else if (type === 3) {
                        typeDom = h('span', {
                            style: {
                                backgroundColor: '#f90',
                                color: '#fff'
                            },
                            class: ['fs-tag']
                        }, '待写');
                    } else if (type === 4) {
                        typeDom = h('span', {
                            style: {
                                backgroundColor: '#80848f',
                                color: '#fff'
                            },
                            class: ['fs-tag']
                        }, '未写');
                    } else if (type === 6) {
                        typeDom = h('span', {
                            style: {
                                backgroundColor: '#2b85e4',
                                color: '#fff'
                            },
                            class: ['fs-tag']
                        }, '休息');
                    }
                    return h('div', {
                        class: [params.row['day' + i].day ? 'inner' : ''],
                        on: {
                            click: () => {
                                this._logRowClick(params.row['day' + i]);
                            }
                        }
                    }, [
                        h('span', params.row['day' + i].day),
                        typeDom
                    ]);
                };
            }
        },
        components: {
            textEditor
        }
    };
</script>
