<template lang="pug">
    .db-details-container
        .db-info
            |
            .vertical-middle
                el-card.card(:body-style="{padding:'10px',display: 'flex','justify-content': 'space-between', 'flex-direction': 'column'}")
                    .card-header(slot='header')
                        img(:src='icon_db')
                        span 数据库详情
                    |
                    .place-holder-of-empty-list(v-if='dbInfo.length === 0')
                        span {{text_place_holder}}
                    |
                    .entry(v-for='entry in dbInfo', :key='entry.display_name', v-else)
                        span.display-name {{entry.display_name + " : "}}
                        span.value {{entry.value}}
                |
                .arrow
                    img(:src='icon_arrow_forward')
        |
        .tables
            .text-filter
                el-input(placeholder='过滤表格名', v-model='text_filter_for_tables', :clearable='true', :fit='true', size='mini')
                    img(slot='prefix')
            |
            .list
                el-table(:data='tableList', ref="tableList", @current-change='handleTableListCurrentRowChange', height='100%', :highlight-current-row='true', :show-header='false', size='mini')
                    el-table-column(:show-overflow-tooltip='true')
                        template(slot-scope='scope')
                            img(:src='icon_table')
                            span(style='font-size: 14px;') {{scope.row.tableName}}

</template>

<script>
  import API from "@/service/api";
  import icon_db from "@/assets/images/icon_db.png";
  import icon_table from "@/assets/images/icon_table.png";
  import icon_arrow_forward from "@/assets/images/ic_arrow_forward_black_48dp.png";
  import { isEmpty, map, pick, includes } from "lodash";
  import format from "date-fns/format";

  const zh_cn = require("date-fns/locale/zh-CN");

  export default {
    name: "DbDetails",
    props: ["db_id"],
    data() {
      return {
        icon_db,
        icon_table,
        icon_arrow_forward,
        isLoadingData: false,
        db_info: {},
        table_list: [],
        text_filter_for_tables: "",
        text_place_holder: "暂无数据",
        mapping: {
          "name": "库名",
          "tableCnt": "表数量",
          "createTime": "建库时间",
          "dbAdmin": "Hive管理员",
          "sasPermissionAdmin": "SAS权限管理员",
          "modifyTime": "最近更新时间"
        }
      };
    },
    computed: {
      dbInfo() {
        if (isEmpty(this.db_info)) {
          return [];
        }
        return map(pick(this.db_info, ["name", "tableCnt", "createTime", "dbAdmin", "sasPermissionAdmin", "modifyTime"]),
          (v, k) => {
            return {
              display_name: this.mapping[k],
              value: includes(k.toUpperCase(), "TIME") ? format(
                new Date(v),
                "YYYY[年]MMMD[日]Ah[点]mm[分]ss[秒]",
                { locale: zh_cn }
              ) : v
            };
          });
      },
      tableList() {
        return this.table_list.filter((table) => table.tableName.toLowerCase().includes(String(this.text_filter_for_tables).toLowerCase()));
      }
    },
    watch: {
      db_id(db_id) {
        return this.getHiveById(db_id);
      }
    },
    mounted() {
      console.log(`<DbDetails/> mounted()`);
      this.getHiveById(this.db_id);
    },
    activated: function() {
      console.log(`<DbDetails/> activated()`);
      if (this.isLoadingData) {
        return false;
      } else {
        this.getHiveById(this.db_id);
      }
    },
    methods: {
      handleTableListCurrentRowChange(val) {
        console.log(`select_table() val: => `, val);
        if (isEmpty(val)) {
          return this.$refs.tableList.setCurrentRow();
        }
        return this.$emit("select_table", val.id);
      },
      getHiveById(db_id) {
        console.log(`getHiveById(${db_id})`);
        this.isLoadingData = true;
        this.text_place_holder = "正在获取数据";
        this.db_info = {};
        this.table_list = [];
        return API.getHiveById(db_id).then(res => {
          this.isLoadingData = false;
          this.db_info = res.dbInfo;
          this.table_list = res.tableList.list;
          this.text_place_holder = "暂无数据";
        }, err => {
          console.error(`err: `, err.errmsg);
          this.$notify({
            message: `${err.errmsg}`,
            type: "error",
            duration: 0
          });
          this.isLoadingData = false;
          this.text_place_holder = "暂无数据";
          this.db_info = {};
          this.table_list = [];
        });
      }
    }
  };
</script>

<style lang="stylus" scoped>
    .db-details-container
        display flex
        align-items stretch
        width 100%
        height 100%
        overflow hidden
        background-color #f5f7fa

    .db-info, .tables .list
        overflow-x hidden
        overflow-y auto
        padding 0

        /deep/ .el-card__header
            padding 15px 10px
            font-size 14px

            img
                width 14px
                height auto
                display inline-block
                vertical-align text-top
                margin-right .5em

        /deep/ .entry
            display flex
            height 40px
            line-height 40px
            border-bottom 1px dashed #dcdfe6

            .value
                width 67%
                overflow hidden
                text-overflow ellipsis
                white-space nowrap
                word-break break-all
                vertical-align middle
                font-size 13px
                color #29292d

            .display-name
                color #1d1d1b
                font-size 13px
                width 33%
                font-weight bold

        /deep/ .entry:last-of-type
            border-bottom none

        /deep/ .el-table::before
            height 0

        /deep/ .el-table
            overflow-y auto
            scroll-behavior smooth

            td
                border-bottom 1px solid white

            td:hover
                cursor pointer

            .el-table__body tr.current-row > td
                background-color #f60

                .cell span {
                    color white
                    font-weight bold
                }

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

    .db-info
        width calc(40% + 50px)
        height 100%
        display flex
        align-items start

        .vertical-middle
            display flex
            width 100%
            align-items center

        .card
            width calc(100% - 50px)
            min-height 310px

        .arrow
            display flex
            align-items center
            flex-grow 0

        img
            width 50px
            height auto
            max-height 100%

        .place-holder-of-empty-list
            display flex
            width 100%
            height 100%
            text-align center
            justify-content center
            align-items center
            color #909399
            font-size 14px

    .tables
        flex-grow 1
        width auto
        height 100%

        .text-filter
            height 30px

        .list
            height calc(100% - 30px)


</style>
