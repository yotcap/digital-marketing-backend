<style lang="less">
@import "../excel/common.less";
</style>
<template>
	<div>
        <Row>
            <Card class="seach-condition">
                <span>标题名称：</span>
                <Input v-model="page.tagName" placeholder="请输入标题名称" clearable style="width: 150px"></Input>
                <span>标签状态：</span>
                <Select v-model="page.tagStatus" clearable style="width:150px">
                    <Option v-for="item in fTagStatus" :value="item.value" :key="item.value">{{ item.label }}</Option>
                </Select>
                <span>所属行业：</span>
                <Input v-model="page.tagTrade" placeholder="请输入所属行业" clearable style="width: 150px"></Input>
                <Button @click="fetchList" class="margin-left-10" type="primary" icon="ios-search">筛选</Button>
            </Card>
            <Card>
                <Table :loading="loading" stripe border :columns="tableColumns" :data="resultValue"></Table>
				<div style="margin: 10px;overflow: hidden">
					<div style="float: right;">
            <Page show-total :total="total" :current="page.current" @on-change="changePage"></Page>
					</div>
          <Button @click="addModal"   type="warning">添加新标签</Button>
				</div>
            </Card>
        </Row>
        <Modal
            width="900"
            title="编辑标签"
            v-model='modalFlag'
            :closable="false"
            :mask-closable="false"
            footer-hide>
            <Form :model="model" :label-width="80">
                <FormItem label="标题名称">
                    <Input v-model="model.name" placeholder="请输入标题名称"></Input>
                </FormItem>
                <FormItem label="编码">
                    <Input v-model="model.esCode" placeholder="请输入编码"></Input>
                </FormItem>
                <FormItem label="排序">
                    <Input v-model="model.sort" placeholder="请输入排序"></Input>
                </FormItem>
                <FormItem label="附件条件">
                    <can-edit-table :hover-show="hoverShow" :loading="modalLoading" @on-change="handleChangeLabel" @on-delete="handleLabelRemove" v-model="model.condition" :columns-list="labelTableColumns"></can-edit-table>
                    <Button @click="addCondition" class="margin-top-10"  type="primary">添加记录</Button>
                </FormItem>
                <div class="center width-100">
                    <Button @click="cancelModal" type="default">取消</Button>
                    <Button @click="saveModal" type="primary">保存</Button>
                </div>
            </Form>
        </Modal>
    </div>
</template>
<script>
import canEditTable from "./canEditTable";
export default {
  props: ["tabName"],
  components: {
    canEditTable
  },
  data() {
    return {
      resultValue: [],
      loading: true,
      modalFlag: false, // 是否显示标签编辑
      modalLoading: false, // 标签编辑附加条件列表
      hoverShow: true,
      total: null, // 总页数
      page: {
        current: 1, // 当前页数
        size: 10, // 每页显示条数
        tagName: "", // 标题名称
        tagStatus: "", // 标签状态
        tagTrade: "", // 所属行业
        operationId: this.tabName
      },
      loading: true, // 表格加载动画
      model: {
        name: "",
        esCode: "",
        sort: "",
        status: "",
        condition: []
      },
      modelTemp: {
        name: "",
        esCode: "",
        sort: "",
        status: "",
        condition: []
      },
      conditionObj: {
        key: "",
        condition: "",
        value: "",
        remark: ""
      },
      fTagStatus: [
        // 标签状态
        {
          value: "1",
          label: "已上架"
        },
        {
          value: "0",
          label: "未上架"
        }
      ],
      tableColumns: [
        // 表头
        {
          title: "序号",
          type: "index",
          width: 80,
          align: "left"
        },
        {
          title: "标签名称",
          key: "name",
          align: "left"
        },
        {
          title: "编码",
          key: "esCode",
          align: "left"
        },
        {
          title: "附加条件",
          key: "condition",
          align: "left"
        },
        {
          title: "状态",
          key: "status",
          align: "left"
        },
        {
          title: "操作",
          key: "action",
          width: 200,
          align: "left",
          render: (h, params) => {
            return h("div", [
              h(
                "Button",
                {
                  props: {
                    type: "primary",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.editModal(params.row);
                    }
                  }
                },
                "编辑"
              ),
              h(
                "Button",
                {
                  props: {
                    type: "primary",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.editTagStatus(params.row);
                    }
                  }
                },
                "下架"
              ),
              h(
                "Button",
                {
                  props: {
                    type: "error",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.handleRemove(params.row.id);
                    }
                  }
                },
                "删除"
              )
            ]);
          }
        }
      ],

      labelTableColumns: [
        {
          title: "KEY（必填）",
          key: "key",
          width: 150,
          align: "left",
          editable: true
        },
        {
          title: "条件（必填）",
          key: "condition",
          width: 150,
          align: "left",
          editable: true
        },
        {
          title: "value（必填）",
          key: "value",
          width: 150,
          align: "left",
          editable: true
        },
        {
          title: "备注（必填）",
          key: "remark",
          align: "left",
          editable: true
        },
        {
          title: " ",
          key: "action",
          width: 140,
          handle: ["edit", "delete"]
        }
      ]
    };
  },
  beforeMount() {
    // this.loading = false
    this.fetchList();
  },
  methods: {
    // 获取数据
    fetchList() {
      this.loading = true;
      this.getInformationList(this.page)
        .then(res => {
          this.loading = false;
          this.resultValue = res.data.records;
          this.total = res.data.total;
        })
        .catch(err => {
          this.loading = false;
        });
    },
    // 分页
    changePage(pageNum) {
      this.page.current = pageNum;
      this.fetchList();
    },
    // 新增标签 弹框
    addModal() {
      this.modalFlag = true;

    },
    // 编辑标签 弹框
    editModal(row) {
      this.modalFlag = true;
      this.model = Object.assign(row);
    },
    // 上架 下架
    editTagStatus(row) {

    },
    cancelModal() {
      this.modalFlag = false;
      this.model = Object.assign(this.modelTemp);
    },
    // 保存标签
    saveModal() {
      let _self = this;
      let axios = Promise.reject();
      if (_self.model.id !== null) {
        axios = api.editTradeTag(_self.model);
      } else {
        axios = api.addTradeTag(_self.model);
      }
      axios.then(res => {
        if (!res.success) {
          this.Message.error("编辑失败，请重试");
          return;
        }
        this.Message.success("编辑成功");
        this.cancelModal();
      });
    },
    // 删除标签
    handleRemove(id) {
      api.delTradeTag(id).then(res => {
        if (res.success) {
          this.Message.success("删除成功");
          this.fetchList();
          return;
        }
        this.Message.error("删除失败，请重试");
      });
    },
    // 添加附件条件
    addCondition() {
      this.model.condition.push(Object.assign(this.conditionObj));
    },
    // 修改 附件条件
    handleChangeLabel(row) {
      // 无操作
    },
    // 删除 附件条件
    handleLabelRemove(row) {
      this.model.condition.splice(row.index,1);
    }
  }
};
</script>
