<template lang="pug">
    .table-details-container(v-loading="isLoadingTable", element-loading-text="加载表格数据。。。", element-loading-spinner="el-icon-loading", element-loading-background="#fff")
        .has-no-permission-mask(v-if='hasNoPermission')
            span 您没有这张表元数据的查询权限
        // 编辑字段弹出框
        el-dialog.dialog-edit-field(title='字段详情', :visible.sync='dialog_edit_field_visible', width='50%', top='50px', append-to-body='', modal-append-to-body='', lock-scroll='', :show-close='true', :close-on-click-modal='false', :close-on-press-escape='false', close='handleCancelUpdateField')
            el-form(:model='form_edit_field', :disabled='isSubmittingFieldForm', label-width="100px", size='small', class="form-edit-field")
                .non-editable-area
                    el-form-item(label='字段名称: ', prop='fieldName')
                        el-input(v-model='form_edit_field.fieldName', :disabled='true', :style='{"width": "200px"}')

                    el-form-item(label='字段类型:', prop='fieldType')
                        el-input(v-model='form_edit_field.fieldType', :disabled='true', :style='{"width": "200px"}')

                    el-form-item(label='创建时间: ', prop='fieldCreateTime')
                        el-input(v-model='form_edit_field.fieldCreateTime', :disabled='true', :style='{"width": "200px"}')

                    el-form-item(label='更新时间:', prop='fieldUpdateTime')
                        el-input(v-model='form_edit_field.fieldUpdateTime', :disabled='true', :style='{"width": "200px"}')

                    el-form-item(label='是否主键:', prop='isPrimarykey')
                        el-switch(v-model='form_edit_field.isPrimarykey', :disabled='true', active-text='是', :active-value='Number(1)', inactive-text='否', :inactive-value='Number(0)')
                .editable-area
                    el-form-item(label='字段描述: ', prop='descr')
                        el-input(v-model='form_edit_field.descr', type='textarea', :style='{"width": "200px"}')

                    el-form-item(label='统计口径: ', prop='statisticsCaliber')
                        el-input(v-model='form_edit_field.statisticsCaliber', :style='{"width": "200px"}')

                    el-form-item(label='允许空值:', prop='isAllowNull')
                        el-switch(v-model='form_edit_field.isAllowNull', active-text='是', :active-value='"Y"', inactive-text='否', :inactive-value='"N"')

                    el-form-item(label='是否敏感信息:', prop='isSensitiveInfo')
                        el-switch(v-model='form_edit_field.isSensitiveInfo', active-text='是', :active-value='"Y"', inactive-text='否', :inactive-value='"N"')

            div(slot="footer")
                el-button(@click="handleUpdateField", size="mini") 保存
                el-button(@click="handleCancelUpdateField", size="mini") 取消
        // 编辑标签弹出框
        el-dialog.dialog-edit-tags(title='编辑标签', :visible.sync='dialog_edit_tags_visible', width='33%', top='50px', append-to-body='', modal-append-to-body='', lock-scroll='', :show-close='true', :close-on-click-modal='false', :close-on-press-escape='false', close='handleCloseEditTags')
            el-form(:model='form_edit_tags', @submit.native.prevent="", :disabled='isSubmittingTagsForm', label-width="100px", size='small', class="form-edit-tags")
                el-form-item(label='请输入: ', prop='new_tag')
                    el-input(v-model.trim='form_edit_tags.new_tag', placeholder='标签名', @keyup.enter.native="handleNewTagInput", :style='{"width": "200px"}')


        .table-metas
            .entries
                .entry(v-for='entry in tableMetas', :key='entry.name')
                    .display-name {{entry.display_name + " : "}}
                    el-tooltip(v-if='entry.display_name === "表名" || entry.display_name === "描述"', effect="light", :content="entry.value", placement="bottom")
                        .value {{entry.value}}
                    .value(v-else) {{entry.value}}
            .tags
                .tags-title
                    | 标签 :
                el-button.btn-add-tags(@click="openFormAddTags", type="primary" icon="el-icon-plus", size="mini", round='', plain='')
                el-tag(v-for='(tag, idx) in list_of_table_tags', @close="handleDelTag(tag)", :key='tag', :closable='true', :disable-transitions="true", size='medium') {{tag | sevenCharsMax}}

        .table
            el-tabs(v-model='activeTabName', @tab-click='handleTabClick' type='border-card')
                el-tab-pane(label='基本信息查询' name='basic_info')
                    el-table.table-basic-info(:data="tableBasicInfo", ref="table", @row-click="handleRowClick", :row-class-name="highlighted", :fit="true", height="100%", :border='true', :stripe='true', size='mini')
                        el-table-column(prop="fieldName", label="字段名称", :sort-method='sortFieldName', :sortable='true', :show-overflow-tooltip='true', :fixed='true', class-name="mono-font", min-width='150')
                        el-table-column(prop="fieldType", label="字段类型", :sortable="true", align="center", class-name="mono-font", width='100')
                        el-table-column(prop="descr", label="字段描述", min-width='150', :show-overflow-tooltip='true')
                        el-table-column(prop="isSensitiveInfo", label="敏感信息", :sortable="true", align="center", width='100')
                            template(slot-scope="scope")
                                span.check(v-if="checkOrX(scope.row.isSensitiveInfo) === '✓'")
                                    | ✓
                                span.x(v-else-if="checkOrX(scope.row.isSensitiveInfo) === '✕'")
                                    | ✕
                                span(v-else)
                                    | 无
                        |
                        el-table-column(prop="isAllowNull", label="允许空值", :sortable="true", align="center", width='100')
                            template(slot-scope="scope")
                                span.check(v-if="checkOrX(scope.row.isAllowNull) === '✓'")
                                    | ✓
                                span.x(v-else-if="checkOrX(scope.row.isAllowNull) === '✕'")
                                    | ✕
                                span(v-else)
                                    | 无
                        |
                        el-table-column(prop="statisticsCaliber", label="统计口径", width="100", :sortable="true")
                        el-table-column(prop="isPrimarykey", label="是否主键", :sortable="true", align="center", width='100')
                            template(slot-scope="scope")
                                span.check(v-if="checkOrX(scope.row.isPrimarykey) === '✓'")
                                    | ✓
                                span.x(v-else-if="checkOrX(scope.row.isPrimarykey) === '✕'")
                                    | ✕
                                span(v-else)
                                    | 无
                        |
                        el-table-column(prop="displayedFieldCreateTime", label="创建时间", width="220", :sort-method='sortCreateTime', :sortable='true')
                        |
                        el-table-column(prop="displayedFieldUpdateTime", label="更新时间", width="220", :sort-method='sortUpdateTime', :sortable='true')

                |
                el-tab-pane.authed-people-pane(label='人员权限查询' name='authed_people', :disabled="isLoadingTable || hasNoPermission")
                    .authed-people
                        el-popover(v-if="authedPeople.length > 0", v-for='person in authedPeople', :key='person.operatorId', trigger='hover', width="500")
                            el-table(:data="person.permissions")
                                el-table-column(width="200" property="aciton" label="操作名")
                                el-table-column(width="300" property="expired" label="失效日期")
                            .authed-person(slot="reference")
                                img(:src="icon_person")
                                .names
                                    div(v-if='person.chinese_name') {{person.um_name}}
                                    div(v-else) {{person.operatorName}}
                                    div(v-if='person.chinese_name') {{person.chinese_name}}
                    |
                    .pagination
                        el-pagination(:disabled="authedPeople.length === 0" , @size-change="handlePageSizeChange", @current-change="handleCurrentPageChange", :current-page.sync="pageNum", :total="total", :page-size="pageSize", layout="total, prev, pager, next", :background="true", :small="true")
                |
                el-tab-pane(label='变更历史查询' name='modified_log', :disabled="isLoadingTable || hasNoPermission")
                    .modified-log
                        light-timeline(:items="timeline_data")
                |
                el-tab-pane(label='血缘关系查询(暂无)' name='relations', :disabled="true") 血缘关系查询


</template>

<script>
  import API from "@/service/api";
  import icon_db from "@/assets/images/icon_db.png";
  import icon_table from "@/assets/images/icon_table.png";
  import icon_person from "@/assets/images/ic_person_outline_24px.svg";
  import {
    sortBy,
    extend,
    filter,
    map,
    assign,
    pick,
    isEmpty
  } from "lodash";
  import format from "date-fns/format";

  const zh_cn = require("date-fns/locale/zh-CN");

  export default {
    name: "TableDetails",
    props: ["table_id", "high_light_field_id"],
    data() {
      return {
        icon_db,
        icon_table,
        icon_person,
        mapping: {
          dbName: "所属库名",
          tableName: "表名",
          descr: "描述",
          amount: "数据量",
          tableCreateTime: "创建时间",
          effectiveTime: "生效时间",
          tableUpdateTime: "更新时间",
          dataOwner: "数据归属",
          devOwner: "开发归属",
          businessOwner: "口径归属"
        },
        isLoadingTable: false,
        hasNoPermission: false,
        dialog_edit_field_visible: false,
        dialog_edit_tags_visible: false,
        isSubmittingFieldForm: false,
        isSubmittingTagsForm: false,
        form_edit_field: {}, // 选中编辑的字段副本
        form_edit_tags: {
          new_tag: ""
        },
        table_basic_info: [],
        table_metas: {},
        list_of_table_tags: [],
        authed_people: [],
        pageNum: 1,// authed_people当前页
        pageSize: 20,// authed_people请求分页数
        total: 0,// authed_people总数
        timeline_data: [],
        activeTabName: "basic_info"
      };
    },
    computed: {
      tableBasicInfo() {
        if (isEmpty(this.table_basic_info)) {
          return [];
        }
        // return this.table_basic_info;
        return sortBy(map(this.table_basic_info, row => {
          return assign(row, {
            displayedFieldCreateTime: row.fieldCreateTime ? format(
              new Date(row.fieldCreateTime),
              "YYYY[年]MMMD[日]Ah[点]mm[分]ss[秒]",
              { locale: zh_cn }
            ) : "无",
            displayedFieldUpdateTime: row.fieldUpdateTime ? format(
              new Date(row.fieldUpdateTime),
              "YYYY[年]MMMD[日]Ah[点]mm[分]ss[秒]",
              { locale: zh_cn }
            ) : "无"
          });
        }), ["fieldName", "isPrimarykey"]);
      },
      tableMetas() {
        if (isEmpty(this.table_metas)) {
          return [];
        }
        return map([
          "dbName",
          "tableName",
          "descr",
          "amount",
          "tableCreateTime",
          "effectiveTime",
          "tableUpdateTime",
          "dataOwner",
          "devOwner"
          // "businessOwner"
        ], (v) => {
          if (v in this.table_metas) {
            if (v.toUpperCase().includes("TIME")) {
              return {
                name: v,
                display_name: this.mapping[v],
                value: this.table_metas[v] ? format(
                  new Date(v),
                  "YYYY[/]MM[/]D hh[:]mm[:]ss",
                  { locale: zh_cn }
                ) : "无"
              };
            }
            return {
              name: v,
              display_name: this.mapping[v],
              value: this.table_metas[v]
            };
          } else {
            return {
              name: v,
              display_name: this.mapping[v],
              value: "暂无"
            };
          }
        });
      },
      authedPeople() {
        let reg = /\((.*?)\)/g; // 匹配圆括号中的人名
        return map(
          this.authed_people,
          p => {
            if (p.operatorName.match(reg)) {
              return extend(p, {
                um_name: p.operatorName.replace(reg, ""),
                chinese_name: (p.operatorName.match(reg))[0].replace(/^\(/, "").replace(/\)$/, "")
              });
            } else {
              return p;
            }
          }
        );
      }
    },
    mounted() {
      this.setUpUI();
      console.log(`<TableDetails/> mounted(): this.table_id: ${this.table_id}, this.high_light_field_id: ${this.high_light_field_id}`);
    },
    activated() {
      console.log(`<TableDetails/> activated(): this.table_id: ${this.table_id}, this.high_light_field_id: ${this.high_light_field_id}`);
      if (this.isLoadingTable) {
        return false;
      } else {
        this.setUpUI();
      }
    },
    deactivated() {
      console.log(`<TableDetails/> deactivated`);
      // this.setUpUI();
      console.log(`<TableDetails/> deactivated(): this.table_id: ${this.table_id}, this.high_light_field_id: ${this.high_light_field_id}`);
      this.clearUI();
    },
    watch: {
      table_id(new_id, old_id) {
        console.log(`table_id changed: new_id : `, new_id, `old_id:`, old_id);
        this.clearUI();
        return this.setUpUI();
      },
      high_light_field_id(new_id, old_id) {
        console.log(`high_light_field_id changed: new_id : `, new_id, `old_id:`, old_id);
        return this.setUpUI(false);
      },
      activeTabName(tab_name) {
        console.log(`activeTabName changed: tab_name : `, tab_name);
        if (tab_name === "modified_log") {
          const loading = this.$loading({
            lock: true,
            target: ".modified-log",
            text: "加载中。。。",
            spinner: "el-icon-loading",
            background: "#fff"
          });
          return API.getTableHistory({
            tableId: this.table_id
          }).then(res => {
            loading.close();
            console.log(res);
            this.timeline_data = map(res, d => {
              return {
                tag: d.CREATETIME,
                content: d.DESCR
              };
            });
            let h = document.querySelector(".modified-log").scrollHeight;
            console.log(`h: ${h}`);
          });
        } else if (tab_name === "authed_people") {
          return this.fetchAuthPeople();
        } else {
          return console.error(`tab_name: ${tab_name}`);
        }
      }
    },
    methods: {
      setUpUI(should_update_data = true) {
        if (!this.table_id) {
          return false;
        }
        should_update_data && this.fetchTable(this.table_id);
      },
      clearUI() {
        console.log(`clearUI()`);
        this.table_metas = {};
        this.table_basic_info = [];
        this.authed_people = [];
        this.dialog_edit_field_visible = false;
        this.activeTabName = "basic_info";
        this.pageNum = 1;
        this.total = 0;
      },
      highlighted({ row, rowIndex }) {
        if (this.high_light_field_id) {
          if (row.id === this.high_light_field_id) {
            return "high-lighted-field";
          } else {
            return "";
          }
        } else {
          return "";
        }
      },
      fetchTable(table_id) {
        console.log(`fetchTable(${table_id})`);
        if (`${table_id}`.length === 0) {
          return false;
        }
        this.isLoadingTable = true;
        this.hasNoPermission = false;
        API.getTableById(table_id).then(
          res => {
            this.isLoadingTable = false;
            this.dialog_edit_field_visible = false;
            this.table_metas = extend({}, res.tableInfo);
            this.list_of_table_tags = this.table_metas.tags ? filter(this.table_metas.tags.split(","), length) : [];
            this.table_basic_info = res.fieldList.list;
            this.authed_people = res.peopleList;
            console.log(`this.high_light_field_id: ${this.high_light_field_id}`);
            if (this.high_light_field_id) {
              let $table = document.querySelector(".table-basic-info");
              console.log(`$table.scrollHeight: ${$table.scrollHeight} , $table.clientHeight: ${$table.clientHeight}`);
              this.$nextTick(() => {
                $table.scrollTop = 0;
              });
            }
          },
          err => {
            console.error(`err: `, err.errmsg);
            this.isLoadingTable = false;
            if (Number(err.errcode) === 1000) {
              return this.hasNoPermission = true;
            }
            this.$notify({
              message: `${err.errmsg}`,
              type: "error",
              duration: 0
            });
          }
        );
      },
      fetchAuthPeople() {
        const loading = this.$loading({
          lock: true,
          target: ".authed-people-pane",
          text: "加载中。。。",
          spinner: "el-icon-loading",
          background: "#fff"
        });
        return API.getTablePermissionList({
          tableName: this.table_metas.tableName,
          dbName: this.table_metas.dbName,
          pageNum: Number(this.pageNum),
          pageSize: Number(this.pageSize)
        }).then(
          res => {
            loading.close();
            console.log(res);
            this.authed_people = res.data;
            this.total = Number(res.total);
          });
      },
      handleTabClick(tab, event) {
        console.log(tab, event);
      },
      handleRowClick(row) {
        console.log("点击单行() : ", JSON.stringify(row));
        this.form_edit_field = extend({}, row);
        return (this.dialog_edit_field_visible = true);
      },
      handleCancelUpdateField() {
        this.dialog_edit_field_visible = false;
        this.form_edit_field = {};
      },
      handleUpdateField() {
        console.log(this.form_edit_field);
        this.isSubmittingFieldForm = true;
        return API.updateField(pick(this.form_edit_field, [
          "id",
          "descr",
          "statisticsCaliber",
          "isAllowNull",
          "isSensitiveInfo"
        ])).then(_ => {
          this.isSubmittingFieldForm = false;
          this.handleCancelUpdateField();
          return this.setUpUI();
        }, err => {
          this.isSubmittingFieldForm = false;
          console.error(`err: `, err.errmsg);
          this.$notify({
            message: `${err.errmsg}`,
            type: "error",
            duration: 0
          });
        });
      },
      handleDelTag(tag) {
        this.list_of_table_tags.splice(this.list_of_table_tags.indexOf(tag), 1);
        console.log(`handleDelTag(${tag}) => `, this.list_of_table_tags);
        return API.updateTableTags({
          id: this.table_id,
          tags: this.list_of_table_tags.join(",")
        });
      },
      handleNewTagInput() {
        if (this.form_edit_tags.new_tag.trim() === "") {
          this.form_edit_tags.new_tag = "";
          return (this.dialog_edit_tags_visible = false);
        }
        this.list_of_table_tags.push(this.form_edit_tags.new_tag);
        this.isSubmittingTagsForm = true;
        return API.updateTableTags({
          id: this.table_id,
          tags: this.list_of_table_tags.join(",")
        }).then(_ => {
          this.isSubmittingTagsForm = false;
          this.form_edit_tags.new_tag = "";
          return (this.dialog_edit_tags_visible = false);
        }, err => {
          this.isSubmittingTagsForm = false;
          console.error(`err: `, err.errmsg);
          this.$notify({
            message: `${err.errmsg}`,
            type: "error",
            duration: 0
          });
        });
      },
      openFormAddTags() {
        if (this.dialog_edit_tags_visible) {
          return false;
        }
        this.form_edit_tags.new_tag = "";
        return (this.dialog_edit_tags_visible = true);
      },
      checkOrX(v) {
        if (v) {
          if (v === "N") {
            return "✕";
          } else if (v === "Y") {
            return "✓";
          } else if (String(v).trim().length === 0) {
            return "";
          } else {
            console.error(`v: ${v}`);
          }
        } else {
          return "";
        }
      },
      sortFieldName(a, b) {
        // console.log(`sortFieldName( ${a.fieldName}, ${b.fieldName} => ${a.fieldName.trim().localeCompare(b.fieldName.trim())}`);
        return `${a.fieldName}`.trim().localeCompare(`${b.fieldName}`.trim());
      },
      sortCreateTime(a: number, b: number): number {
        return Number(a.fieldCreateTime) - Number(b.fieldCreateTime);
      },
      sortUpdateTime(a: number, b: number): number {
        return Number(a.fieldUpdateTime) - Number(b.fieldUpdateTime);
      },
      handleCurrentPageChange(val) {
        this.pageNum = Number(val);
        return this.fetchAuthPeople();
      },
      handlePageSizeChange(val) {
        this.pageSize = Number(val);
        return this.fetchAuthPeople();
      }
    }
  };
</script>

<style lang="stylus" scoped>
    .form-edit-field
        display flex
        > div
            width 50%

    span.check, span.x
        font-size 1.2em

    span.check
        color #67C23A

    span.x
        color #F56C6C

    .table-details-container
        position relative
        display flex
        flex-direction column
        width 100%
        height 100%
        padding 0

        .has-no-permission-mask
            z-index 100
            background white
            position absolute
            display flex
            top 0
            left 0
            width 100%
            height 100%
            color #C0C4CC
            text-align center
            justify-content center
            align-items center

        /deep/ .el-tabs__item
            user-select none

    .table-metas
        display flex
        width 100%
        height 121px
        border 1px solid #dcdfe6
        border-bottom none

        > div {
            display flex
            height 100%
            max-height 100%
            overflow-x hidden
            overflow-y auto
        }

    .entries
        width 75%
        flex-wrap wrap

        .entry
            display flex
            width 33.333%
            height 40px
            line-height 40px
            border-bottom 1px solid #dcdfe6

            .value
                width calc(100% - 75px)
                overflow hidden
                text-overflow ellipsis
                white-space nowrap
                word-break break-all
                vertical-align middle
                font-size 12.5px
                color #606266

            .display-name
                color #1d1d1b
                font-size 13px
                width 75px
                overflow hidden
                padding-left .5em
                font-weight bold

        .entry:nth-child(7),
        .entry:nth-child(8),
        .entry:nth-child(9) {
            border-bottom none
        }

    .tags
        position relative
        flex-shrink 0
        padding-bottom 30px
        width 25%
        min-width 200px
        border-left 1px solid #dcdfe6
        flex-wrap wrap
        justify-content flex-start
        align-content start
        align-items baseline

        .btn-add-tags
            margin-right 5px
            max-height 30px

        .tags-title
            padding-left .5em
            line-height 40px
            width 75px
            color #1d1d1b
            font-size 14px
            font-weight bold

        .el-tag
            margin .5em
            user-select none

    .dbs, .tables .list
        overflow-x hidden
        overflow-y auto
        padding 0

        /deep/ .el-table::before
            height 0

        /deep/ .el-table
            overflow-y auto
            scroll-behavior smooth

            td
                border-bottom 1px solid white

            td:hover
                cursor pointer

            .cell
                line-height 20px

                span
                    color #1D1D1B
                    font-size 15px
                    line-height 20px

                img
                    display inline-block
                    width 20px
                    height auto
                    vertical-align top
                    margin-right .5em

    .el-table /deep/ .mono-font
        font-family mono

    .dbs
        width 40%
        height 100%

    .table
        flex-grow 1
        width 100%
        height calc(100% - 150px)
        overflow hidden

    .table-basic-info
        overflow-y auto

        /deep/ .el-table__row:hover
            cursor pointer

        /deep/ .el-table__body tr.el-table__row.high-lighted-field
            background-color #FF6600

            & > td
                background-color #FF6600
                border-right 1px solid #fff
                .cell
                    color #fff
                    font-weight bold
                    span
                        color #fff

        /deep/ .el-table__row.current-row:hover
            cursor none

    .table-basic-info::before
        height 0

    .authed-people, .modified-log
        display flex
        flex-wrap wrap
        align-content flex-start
        width 100%
        padding 0.5%
        height 100%
        overflow-x hidden
        overflow-y auto

    .authed-people
        height calc(100% - 50px)

    .modified-log .line-container
        width 100%

        & /deep/ .item-tag
            width auto

    .authed-people-pane
        position relative
        flex-direction column

        .pagination
            display flex
            position relative
            justify-content center
            align-items center
            height 50px

            .el-pagination
                display flex
                justify-content center
                align-items center
                padding 5px
                padding-bottom 0

        .authed-person
            display flex
            justify-content space-between
            width 15%
            height 60px
            font-size 11px
            padding .5em
            max-width 250px
            min-width 180px
            margin .5em
            border-radius 4px
            border 1px solid #ebeef5
            overflow hidden
            box-shadow 0 1px 6px 0 rgba(0, 0, 0, .1)

            img
                align-self center
                width 25px
                height 25px

            .names
                display flex
                flex-direction column
                justify-content space-around
                width calc(95% - 25px)
                > div
                    overflow hidden
                    text-overflow ellipsis
                    white-space nowrap
                    word-break break-all
                    vertical-align middle

</style>
