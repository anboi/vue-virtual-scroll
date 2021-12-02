<template>
    <div>
      <div :style="styles['table-container']" v-on:scroll="onTableXScroll" id="horizontal-virtual-scroll-table-container">
          <div :style="styles['whole-contents']">
              <div 
                  :style="styles['current-contents']"
              >
                <table :style="styles['table']" v-if="privateState.currentData">
                    <thead>
                        <tr>
                            <th 
                                v-for="(field, index) in privateState.currentData[0]" 
                                :key="index"
                                :style="styles['col']"
                            >
                                {{field}}
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(row, index) in privateState.currentData" :key="index">
                            <td v-for="(col, index) in row" :key="index">
                                {{col}}
                            </td>
                        </tr>
                    </tbody>
                </table>
              </div>
          </div>
      </div>
      <p>document.querySelectorAll('*').length: {{getAllItemsCount()}}</p>
     </div>
</template>

<script>
/* for local testing purpose */
// var randomstring = require("randomstring");

/* npm package for converting csv to json format */
var Papa = require("papaparse");

const TABLE_HEIGHT = 50;
const TABLE_WIDTH = 80;
const COL_WIDTH = 200;

export default {
  name: "HorizontalVirtualScrollTable",
  data() {
    return {
      privateState: {
        // rowNum: 100,
        // colNum: 1000,
        // stringLen: 10,
        tableData: null,
        currentData: null,
        // tableOverflowX: 0
      },
      sharedState: {},
      styles: {
        "table-container": {
          maxHeight: TABLE_HEIGHT + "vh",
          width: TABLE_WIDTH + "vw",
          overflow: "auto"
        },
        "whole-contents": {
          width: "100%",
          overflow: "hidden"
        },
        "current-contents": {},
        table: {
          tableLayout: "fixed",
          width: TABLE_WIDTH + "vw"
        },
        col: {
          width: COL_WIDTH + "px"
        }
      }
    };
  },
  mounted() {
    /**
     * Loading csv file
     */
    var that = this;
    Papa.parse(
      "https://raw.githubusercontent.com/anboi/vue-virtual-scroll/master/src/components/acho_export_virtual_scroll.csv",
      {
        download: true,
        complete: function(results) {
          console.log("Finished:", results.data);
          that.privateState.tableData = results.data;
          that.privateState.currentData = results.data.map(row => {
            return row.slice(
              0,
              Math.ceil(
                document.documentElement.clientWidth *
                  TABLE_WIDTH /
                  100 /
                  COL_WIDTH
              )
            );
          });
          that.styles["whole-contents"].width =
            results.data[0].length * COL_WIDTH + "px";
        }
      }
    );
  },
  computed: {
    /* for local tsting  */
    // getTableData: function() {
    //   let table = {
    //     schema: [],
    //     data: []
    //   };
    //   for (let i = 0; i < this.privateState.rowNum; i++) {
    //     let row = [];
    //     for (let j = 0; j < this.privateState.colNum; j++) {
    //       if (i == 0) table["schema"].push("field_" + j);
    //       row.push(randomstring.generate(this.privateState.stringLen));
    //     }
    //     table["data"].push(row);
    //   }
    //   return table;
    // }
  },
  methods: {
    onTableXScroll() {
      let data = this.privateState.tableData;
      let viewportWidth = document.querySelector(
        "#horizontal-virtual-scroll-table-container"
      ).clientWidth;
      let colWidth = COL_WIDTH;
      let scrollLeft = document.querySelector(
        "#horizontal-virtual-scroll-table-container"
      ).scrollLeft;
      let startIndex = Math.floor(scrollLeft / colWidth) - 1;
      startIndex = Math.max(0, startIndex);
      let endIndex = startIndex + Math.floor(viewportWidth / colWidth) + 1;
      endIndex = Math.min(data[0].length - 1, endIndex);

      let tableOverflowX =
        Math.floor(scrollLeft / colWidth) * colWidth - colWidth;

      console.log(startIndex, " ", endIndex);
      this.privateState.currentData = data.map(row => {
        return row.slice(startIndex, endIndex + 1);
      });
      console.log(this.privateState.currentData);

      this.styles["current-contents"][
        "transform"
      ] = `translateX(${tableOverflowX}px)`;
    },
    getAllItemsCount() {
      return document.querySelectorAll("*").length;
    }
  }
};
</script>