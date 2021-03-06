<template>
  <div>
    <TopFilter :list="topFilterList" :cols="4" @filterChange="changeHandle" @onCollapse="collapseHandle" />
    <button-area :container-style="{ paddingLeft: '80px' }">
      <el-button size="small" type="primary">到货确认</el-button>
      <el-button size="small">明细</el-button>
      <el-button size="small">发货单</el-button>
      <el-button size="small">销售发票</el-button>
      <el-button size="small" @click="asdasd">欠货单</el-button>
      <multiuse-button size="small" :auth-list="auths" auth-mark="/api/aaa">出库</multiuse-button>
    </button-area>
    <FilterTable
      ref="table"
      columns-ref="myTable"
      :columns="columns"
      :data-source="list"
      :is-memory-pagination="true"
      :on-columns-change="columns => (this.columns = columns)"
      :on-sync-table-data="tableDateChange"
    >
      <template slot="moreActions">
        <span>批量删除</span>
        <span>任务分配</span>
      </template>
      <template slot="controls">
        <el-button size="small" type="primary" icon="el-icon-plus" @click="visible = true">新建</el-button>
        <el-button size="small" icon="el-icon-printer" @click="printHandler">打印</el-button>
      </template>
    </FilterTable>
    <Drawer :visible.sync="visible" title="标题名称" destroy-on-close :container-style="{ height: 'calc(100% - 60px)', overflow: 'auto', paddingBottom: '60px' }">
      <Panel @close="closeHandler" />
    </Drawer>
    <BasePrint ref="print" :data="printList" printerType="stylus" :alwaysPrint="false" :isPreview="false" template="template1" />
  </div>
</template>

<script>
import { authority } from '@/utils/authMixin';
import { dictionary } from '@/utils/dictMixin';
import res from '@/mock/tableData';
import printData from '@/mock/printData';
import Panel from './Panel';

import pinyin, { STYLE_FIRST_LETTER } from '@/components/Pinyin/index';

export default {
  name: 'Demo',
  components: {
    Panel
  },
  mixins: [authority, dictionary],
  data() {
    this.BaseTable = null;
    return {
      visible: false,
      topFilterList: this.createTopFilters(),
      columns: this.createTableColumns(),
      list: [],
      printList: printData.data
    };
  },
  mounted() {
    this.BaseTable = this.$refs.table;
    console.log('页面不具备的权限：', this.auths);
    this.BaseTable.START_LOADING();
    setTimeout(() => {
      this.BaseTable.STOP_LOADING();
      this.list = [...res.data.items];
      this.topFilterList.find(x => x.fieldName === 'startTime|endTime').initialValue = ['2019-10-12', '2019-10-28'];
      this.BaseTable.SET_DISABLE_SELECT([this.list[0], this.list[2]]);
      this.topFilterList[0].hidden = false;
      this.topFilterList[0].initialValue = 'asdf';
      this.topFilterList[0].labelOptions.initialValue = '22';
      this.topFilterList[0].labelOptions.itemList = [
        { text: '搜索1', value: '11' },
        { text: '搜索2', value: '22' }
      ];
    }, 3000);
  },
  methods: {
    asdasd() {
      console.log(this.BaseTable.GET_REQUIRED_ERROR());
    },
    createTopFilters() {
      return [
        {
          type: 'INPUT',
          label: '搜索',
          fieldName: 'title',
          hidden: true,
          placeholder: '请输入标题名称...',
          labelOptions: {
            fieldName: 'qwe',
            itemList: []
          },
          onInput: val => {
            let res = pinyin(val, { style: STYLE_FIRST_LETTER })
              .flat()
              .join('')
              .toUpperCase();
            this.topFilterList.find(x => x.fieldName === 'zxczxc').initialValue = res;
          },
          rules: [
            { required: true, message: '请输入标题名称', trigger: 'blur' },
            { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
          ]
        },
        {
          type: 'INPUT',
          label: '搜索2',
          fieldName: 'zxczxc',
          readonly: true
        },
        {
          type: 'MULTIPLE_SELECT',
          label: '所属分类',
          fieldName: 'cid',
          placeholder: '所属分类',
          filterable: true,
          rules: [{ required: true, message: '请选择所属分类', trigger: 'change' }],
          request: {
            fetchApi: () => {},
            params: {},
            datakey: 'items',
            valueKey: 'id',
            textKey: 'name'
          }
        },
        {
          type: 'DATE',
          label: '日期',
          fieldName: 'date',
          placeholder: '选择日期',
          maxDateTime: '2019-11-15',
          rules: [{ required: true, message: '请选择日期', trigger: 'change' }]
        },
        {
          type: 'RANGE_DATE',
          label: '日期区间',
          style: { minWidth: '200px' },
          fieldName: 'startTime|endTime',
          rules: [{ required: true, message: '请选择日期', trigger: 'change' }]
        },
        {
          type: 'MULTIPLE_SELECT',
          label: '兴趣爱好',
          fieldName: 'hobby',
          placeholder: '兴趣爱好',
          itemList: [
            { text: '篮球', value: '1' },
            { text: '足球', value: '2' },
            { text: '乒乓球', value: '3' }
          ],
          rules: [{ required: true, message: '请选择兴趣爱好', trigger: 'change' }]
        },
        {
          type: 'SEARCH_HELPER_WEB',
          label: '搜索帮助',
          fieldName: 'person',
          placeholder: '请输入员工名称...',
          itemList: [{ text: '中国' }, { text: '美国' }],
          rules: [{ required: true, message: '请输入员工名称', trigger: 'change' }]
        },
        {
          type: 'INPUT_CASCADER',
          label: '联机',
          fieldName: 'hello',
          placeholder: '请输入标题名称...',
          style: { minWidth: '300px' },
          options: {
            titles: ['品牌', '车型', '车系']
          },
          initialValue: '1,1-2,1-2-1',
          itemList: [
            {
              text: '一级分类1',
              value: '1',
              children: [
                {
                  text: '二级分类一级分类1一级分类1一级分类11-1',
                  value: '1-1',
                  children: [
                    {
                      text: '三级分类1-1',
                      value: '1-1-1'
                    },
                    {
                      text: '三级分类1-2',
                      value: '1-1-2'
                    }
                  ]
                },
                {
                  text: '二级分类1-2',
                  value: '1-2',
                  children: [
                    {
                      text: '三级分类2-1',
                      value: '1-2-1'
                    },
                    {
                      text: '三级分类2-2',
                      value: '1-2-2'
                    }
                  ]
                }
              ]
            },
            {
              text: '一级分类2',
              value: '2',
              children: [
                {
                  text: '二级分类2-1',
                  value: '2-1'
                }
              ]
            }
          ]
        }
      ];
    },
    createTableColumns() {
      return [
        {
          title: '操作',
          dataIndex: 'column-action',
          width: 100,
          render: (props, h) => {
            return (
              <div>
                <multiuse-button size="mini" type="text" divider="after" containerStyle={{ color: 'red' }} auth-list={this.auths} auth-mark={'/api/bbb/*'}>
                  编辑
                </multiuse-button>
                <el-button size="mini" type="text">
                  查看
                </el-button>
              </div>
            );
          }
        },
        {
          title: '序号',
          dataIndex: 'index',
          width: 70,
          sorter: true,
          render: props => {
            return <span>{props.row.index + 1}</span>;
          }
        },
        {
          title: '日期',
          dataIndex: 'date',
          width: 150,
          sorter: true,
          filter: true,
          filterType: 'date-range',
          editRequired: true,
          editable: true,
          editType: 'date-picker'
        },
        {
          title: '姓名',
          dataIndex: 'person.name',
          width: 120,
          sorter: true,
          filter: true,
          filterType: 'input',
          editable: true,
          editType: 'text',
          editRequired: true,
          searchHelper: {
            fetchApi: () => {},
            params: {},
            // key -> 数据字段名
            // value -> json 对象，dataIndex 的值就是 column 的 dataIndex
            aliasKey: {
              name: {
                // 注意：当前列（column）的 dataIndex 必须配置在 aliasKey 中，最好放在一项
                dataIndex: 'person.name'
              },
              number: {
                dataIndex: 'num'
              },
              price: {
                dataIndex: 'price'
              }
            }
          }
        },
        {
          title: '性别',
          dataIndex: 'person.sex',
          width: 100,
          sorter: true,
          filter: true,
          filterType: 'radio',
          filterItems: [
            { text: '男', value: 1 },
            { text: '女', value: 0 }
          ],
          render: props => {
            return <span>{props.row.person.sex === 1 ? '男' : '女'}</span>;
          }
        },
        {
          title: '价格',
          dataIndex: 'price',
          width: 120,
          precision: 2,
          numberFormat: true,
          sorter: true,
          filter: true,
          filterType: 'number',
          summation: true,
          summationUnit: '元',
          editable: true,
          editType: 'number',
          editRequired: true,
          // 可编辑的单元格在强制不可编辑状态下的渲染函数，可以绑定事件
          editDisableRender: props => {
            return <div onClick={() => alert('强制不可编辑单元格自定义的单击事件')}>{props.row.price}</div>;
          }
        },
        {
          title: '数量',
          dataIndex: 'num',
          width: 120,
          sorter: true,
          editable: true,
          editType: 'number',
          precision: 0,
          editRequired: true,
          max: 300,
          min: 0
        },
        {
          title: '总价',
          dataIndex: 'total',
          width: 120,
          sorter: true,
          summation: true,
          summationUnit: '元',
          render: props => {
            // 计算规则
            props.row.total = props.row.price * props.row.num;
            const domStyle = props.row.total > 1000 ? {} : null;
            return <div style={domStyle}>{props.row.total}</div>;
          }
        },
        {
          title: '是否选择',
          dataIndex: 'choice',
          align: 'center',
          sorter: true,
          filter: true,
          filterType: 'radio',
          filterItems: [
            { text: '未选择', value: '0' },
            { text: '已选择', value: '1' }
          ],
          editable: true,
          defaultEditable: true,
          editType: 'checkbox',
          editItems: [
            { text: '', falseValue: '0' },
            { text: '', trueValue: '1' }
          ]
        },
        {
          title: '状态',
          dataIndex: 'state',
          width: 150,
          sorter: true,
          filter: true,
          filterType: 'checkbox',
          editRequired: true,
          filterItems: [
            { text: '已完成', value: 1 },
            { text: '进行中', value: 2 },
            { text: '未完成', value: 3 },
            { text: '已完成', value: 11 },
            { text: '进行中', value: 21 },
            { text: '未完成', value: 31 },
            { text: '已完成', value: 111 },
            { text: '进行中', value: 211 },
            { text: '未完成', value: 311 }
          ],
          editable: true,
          editType: 'select',
          editItems: [
            { text: '已完成', value: 1 },
            { text: '进行中', value: 2 },
            { text: '未完成', value: 3 }
          ]
        },
        {
          title: '业余爱好',
          dataIndex: 'hobby',
          width: 150,
          sorter: true,
          filter: true,
          filterType: 'checkbox',
          filterItems: [
            { text: '篮球', value: 1 },
            { text: '足球', value: 2 },
            { text: '乒乓球', value: 3 },
            { text: '游泳', value: 4 }
          ],
          editable: true,
          editType: 'select-multiple',
          editItems: [
            { text: '篮球', value: 1 },
            { text: '足球', value: 2 },
            { text: '乒乓球', value: 3 },
            { text: '游泳', value: 4 }
          ]
        },
        {
          title: '地址',
          dataIndex: 'address',
          showOverflowTooltip: true,
          editable: true,
          editType: 'text',
          editPattern: /^[0-9a-zA-Z ]+$/
        }
      ];
    },
    changeHandle(val) {
      console.log('搜索的参数：', val);
    },
    collapseHandle() {
      this.$nextTick(() => {
        this.BaseTable.EXECUTE_RESET_HEIGHT();
      });
    },
    closeHandler(val) {
      this.visible = val;
    },
    printHandler() {
      this.$refs.print.EXCUTE_PRINT();
    },
    tableDateChange(list) {
      // console.log(list);
    }
  }
};
</script>

<style lang="less"></style>
