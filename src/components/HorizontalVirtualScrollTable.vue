<template>
    <v-container>
      <v-row :justify="'center'" :align="'center'" :style="{height: '100vh',}">
        <v-col>
          <v-row :justify="'center'">
            <p :style="{fontSize: '2rem', fontWight: 'bold',}">Horizontal Virtual Scroll Table</p>
          </v-row>
          <v-row :justify="'center'" :style="{margin: '2rem 0 2rem 0'}">
            <div v-if="!privateState.currentData">
                <v-progress-circular
                  :width="3"
                  color="green"
                  indeterminate
                ></v-progress-circular>
            </div>
          </v-row>
          <v-row :justify="'center'">
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
                                            v-for="(item, index) in privateState.currentData[0]" 
                                            :key="index"
                                            :style="getHeaderStyle(item['originalIndex'])"
                                        >
                                            {{item['data']}}
                                        </th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr v-for="(row, index) in privateState.currentData.slice(1)" :key="index" :style="styles['row']">
                                        <td v-for="(col, index) in row" :key="index" :style="styles['col']">
                                            <div v-if="!col" :style="{color: 'lightgrey'}">null</div>
                                            <div v-else>{{col}}</div>
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
        </v-col>
      </v-row>
    </v-container>
</template>

<script>
/* npm package for converting csv to json format */
var Papa = require("papaparse");
import colors from 'vuetify/lib/util/colors'

const TABLE_HEIGHT = 50;
const TABLE_WIDTH = 80;
const COL_FONT_SIZE = 12;
const COL_PADDING = 10;
const ROW_HEIGHT = 100;

export default {
  name: "HorizontalVirtualScrollTable",
  data() {
    return {
      privateState: {
        tableData: null,
        currentData: null,
        colsWidth: [],
        accuColsWidth: [],
      },
      sharedState: {},
      classes: {},
      styles: {
        "table-container": {
          maxHeight: TABLE_HEIGHT + "vh",
          width: TABLE_WIDTH + "vw",
          overflow: "auto",
        },
        "whole-contents": {
          width: "100%",
          overflow: "hidden",
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
        // header: {
        //   width: COL_WIDTH + "px",
        //   textAlign: 'left',
        //   border: "1px solid #ddd",
        //   padding: '10px',
        //   backgroundColor: colors.green.lighten5,
        // },
        col: {
          textAlign: 'left',
          border: "1px solid #ddd",
          padding: COL_PADDING + 'px',
        },
      },
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
          results.data[0] = results.data[0].map((item, index) => {
            return {
              "originalIndex": index,
              "data": item,
            };
          });
          let first50Rows = results.data.slice(0, 52);
          that.privateState.tableData = first50Rows;

          let colsWidth = Array(first50Rows[0].length).fill(100);

          /* calculate each column width */
          for(let i = 0; i < first50Rows.length; i++){
            for(let j = 0; j < first50Rows[i].length; j++){
              if(first50Rows[i][j].length * COL_FONT_SIZE + COL_PADDING * 2 > colsWidth[j]){
                  colsWidth[j] = first50Rows[i][j].length * COL_FONT_SIZE + COL_PADDING * 2;
              }
            }
          }
          that.privateState.colsWidth = [...colsWidth];
          
          /* calculate accumulated columns width */
          that.privateState.accuColsWidth = Array(first50Rows[0].length).fill(0);
          that.privateState.accuColsWidth[0] = that.privateState.colsWidth[0];
          for(let i = 1; i < results.data[0].length; i++){
            that.privateState.accuColsWidth[i] = that.privateState.accuColsWidth[i - 1] + that.privateState.colsWidth[i];
          }

          /* initialize table */
          let startIndex = that.getColStartIndex();
          let endIndex = that.getColEndIndex();

          that.privateState.currentData = first50Rows.map(row => {
            return row.slice(startIndex, endIndex + 1);
          });

          /* set the total width of all columns */
          that.styles["whole-contents"].width =
            that.privateState.accuColsWidth[that.privateState.accuColsWidth.length - 1] + "px";
        }
      }
    );
  },
  computed: {
    
  },
  methods: {
    getHeaderStyle: function(originalIndex){
      return {
        width: this.privateState.colsWidth[originalIndex] + "px",
        textAlign: 'left',
        border: "1px solid #ddd",
        padding: COL_PADDING + 'px',
        backgroundColor: colors.green.lighten5,
        fontSize: COL_FONT_SIZE + 'px',
      };
    },
    getFirstAccuColIndexGreaterThanWidth(width){
      /* binary search */
      var start = 0;
      var end = this.privateState.accuColsWidth.length - 1;
      var mid = Math.floor((start + end) / 2);

      while(start != end) {
        mid = Math.floor((start + end) / 2);
        if(this.privateState.accuColsWidth[mid] > width){
          end = mid;
        }else{
          start = mid + 1;
        }
      }

      return start;
    },
    getColStartIndex() {
      let scrollLeft = document.querySelector(
        "#horizontal-virtual-scroll-table-container"
      ).scrollLeft;
      let startIndex = this.getFirstAccuColIndexGreaterThanWidth(scrollLeft);
      return Math.max(startIndex - 1, 0);
    },
    getColEndIndex() {
      let scrollLeft = document.querySelector(
        "#horizontal-virtual-scroll-table-container"
      ).scrollLeft;
      let viewportWidth = document.querySelector(
        "#horizontal-virtual-scroll-table-container"
      ).clientWidth;
      let totalLeft = scrollLeft + viewportWidth;
      let endIndex = this.getFirstAccuColIndexGreaterThanWidth(totalLeft);
      return Math.min(endIndex + 1, this.privateState.accuColsWidth.length - 1);
    },
    onTableXScroll() {
      let startIndex = this.getColStartIndex();
      let endIndex = this.getColEndIndex();
      this.privateState.currentData = this.privateState.tableData.map(row => {
        return row.slice(startIndex, endIndex + 1);
      });
      let overflowX = startIndex === 0 ? 0 : this.privateState.accuColsWidth[Math.max(startIndex - 1, 0)];
      this.styles["current-contents"][
        "transform"
      ] = `translateX(${overflowX}px)`;
    },
    getAllItemsCount() {
      return document.querySelectorAll("*").length;
    }
  }
};
</script>