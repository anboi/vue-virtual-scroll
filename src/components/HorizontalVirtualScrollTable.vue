<template>
    <v-container>
      <v-row :justify="'center'" :align="'center'" :style="{height: '100vh'}">
        <div v-if="!privateState.currentData">
            <v-progress-circular
              :width="3"
              color="green"
              indeterminate
            ></v-progress-circular>
        </div>
        <v-sheet
          color="white"
          elevation="2"
          rounded
        >
          <v-row no-gutters>
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
                                <tr v-for="(row, index) in privateState.currentData.slice(1)" :key="index" :style="styles['row']">
                                    <td v-for="(col, index) in row" :key="index" :style="styles['col']">
                                        {{col}}
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
          </v-row>
          <div :style="{height: '2rem'}" />
          <v-row :justify="'center'" no-gutters>
            <p>document.querySelectorAll('*').length: {{getAllItemsCount()}}</p>
          </v-row>
        </v-sheet>
      </v-row>
    </v-container>
</template>

<script>
/* npm package for converting csv to json format */
var Papa = require("papaparse");

const TABLE_HEIGHT = 50;
const TABLE_WIDTH = 80;
const COL_WIDTH = 200;
const ROW_HEIGHT = 100;

export default {
  name: "HorizontalVirtualScrollTable",
  data() {
    return {
      privateState: {
        tableData: null,
        currentData: null,
        showTable: false,
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
          width: TABLE_WIDTH + "vw",
          borderCollapse: "collapse",
          
        },
        row: {
          height: ROW_HEIGHT + "px",
          border: "1px solid #ddd",
        },
        col: {
          width: COL_WIDTH + "px",
          textAlign: 'left',
          border: "1px solid #ddd",
          padding: '10px',
        },
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
  computed: {},
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
      tableOverflowX = startIndex == 0 ? 0 : tableOverflowX;
      this.privateState.currentData = data.map(row => {
        return row.slice(startIndex, endIndex + 1);
      });
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