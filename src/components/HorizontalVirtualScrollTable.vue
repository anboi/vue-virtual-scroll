<template>
    <div :style="styles['table-container']" v-on:scroll="onTableXScroll" id="horizontal-virtual-scroll-table-container">
        <table :style="styles['table']">
            <thead>
                <tr>
                    <th 
                        v-for="(field, index) in getTableData['schema']" 
                        :key="index"
                        :style="styles['col']"
                    >
                        {{field}}
                    </th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(row, index) in getTableData['data']" :key="index">
                    <td v-for="(col, index) in row" :key="index">
                        {{col}}
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<script>
    var randomstring = require("randomstring");
    
    var csv=require("csvtojson");
    export default {
        name: "HorizontalVirtualScrollTable",
        data(){
            const csvFilePath = './acho_export_virtual_scroll.csv';

            csv()
            .fromFile(csvFilePath)
            .then((jsonObj)=>{
                console.log(jsonObj);
                /**
                 * [
                 * 	{a:"1", b:"2", c:"3"},
                 * 	{a:"4", b:"5". c:"6"}
                 * ]
                 */ 
})
            // function getTableData(){
            //     let table = {
            //         'schema': [],
            //         'data': [],
            //     };
            //     for(let i = 0; i < this.privateState.rowNum; i++){
            //         let row = [];
            //         for(let j = 0; j < this.privateState.colNum; j++){
            //             if(i == 0) table['schema'].push('field_' + j);
            //             row.push(randomstring.generate(this.privateState.stringLen));
            //         }
            //         table['data'].push(row);
            //     }
            //     return table;
            // }
            return {
                privateState: {
                    rowNum: 100,
                    colNum: 1000,
                    stringLen: 10,
                    currentData: {
                        // 'schema': this.getTableData['schema'].slice(0, 12),
                        // 'data': this.getTableData['data'].slice(0, 12),
                    },
                    tableOverflowX: 0,
                },
                sharedState: {

                },
                styles: {
                    'table-container': {
                        maxHeight: '50vh',
                        maxWidth: '80vw',
                        overflow: 'auto',
                    },
                    'table': {
                        tableLayout: 'fixed',
                        width: '100%',
                    },
                    col: {
                        width: '200px',
                    }
                },
            }
        },
        computed: {
            getTableData: function(){
                let table = {
                    'schema': [],
                    'data': [],
                };
                for(let i = 0; i < this.privateState.rowNum; i++){
                    let row = [];
                    for(let j = 0; j < this.privateState.colNum; j++){
                        if(i == 0) table['schema'].push('field_' + j);
                        row.push(randomstring.generate(this.privateState.stringLen));
                    }
                    table['data'].push(row);
                }
                return table;
            }
        },
        methods: {
            onTableXScroll(){
                let data = this.getTableData;
                let viewportWidth = document.querySelector('#horizontal-virtual-scroll-table-container').clientWidth;
                let colWidth = 200;
                let scrollLeft = document.querySelector('#horizontal-virtual-scroll-table-container').scrollLeft;
                let startIndex = Math.floor(scrollLeft / colWidth) - 1;
                startIndex = Math.max(0, startIndex);
                let endIndex = startIndex + Math.floor(viewportWidth / colWidth) + 1;
                endIndex = Math.min(data['data'].length - 1, endIndex);
                this.privateState.tableOverflowX = scrollLeft - (colWidth + Math.floor(scrollLeft / colWidth) * colWidth);

                this.privateState.currentData['schema'] = data['schema'].slice(startIndex, endIndex + 1);
                this.privateState.currentData['data'] = data['data'].slice(startIndex, endIndex + 1);

                console.log(this.privateState.currentData['schema'])
            },
        }
    }
</script>