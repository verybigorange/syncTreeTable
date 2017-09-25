<template>  
  <div class="hello">  
    <sync-tree-grid :columns="columns" :tree-structure="true" :datainit="datainit" :buttons="buttons" @syncTreeGridData="syncTreeGridData"></sync-tree-grid>  
  </div>  
</template>  
  
<script>  
import SyncTreeGrid from './SyncTreeGrid'  
export default {  
  name: 'hello',  
  data () {  
    return {  
      columns: [  
          {  
            text: '字典项名称',  
            dataIndex: 'name'  
          },  
          {  
            text: '字典项编码',  
            dataIndex: 'code'  
          },  
          {  
            text: '字典项等级',  
            dataIndex: 'level'  
          }  
        ],  
      datainit: [{
					"code": "100",
					"createTime": "2017-09-23 04:34:13",
					"dictId": "c9ac7dde9ddd11e7ae293cbf104428a9",
					"enable": "YES",
					"id": 1000,
					"leaf": "NO",
					"level": 1,
					"name": "testItem",
					"path": "1000",
					"remark": "testRemark"
				},
				{
					"code": "100101",
					"createTime": "2017-09-23 04:33:24",
					"dictId": "c9ac7dde9ddd11e7ae293cbf104428a9",
					"enable": "YES",
					"id": 1001,
					"leaf": "YES",
					"level": 2,
					"name": "item2",
					"path": "1000.1001",
					"remark": "test1"
				},
				{
					"code": "100102",
					"createTime": "2017-09-23 04:32:51",
					"dictId": "c9ac7dde9ddd11e7ae293cbf104428a9",
					"enable": "YES",
					"id": 1002,
					"leaf": "YES",
					"level": 2,
					"name": "item3",
					"path": "1000.1002",
					"remark": "test2"
				},
				{
					"code": "101",
					"createTime": "2017-09-23 04:34:15",
					"dictId": "c9ac7dde9ddd11e7ae293cbf104428a9",
					"enable": "YES",
					"id": 1003,
					"leaf": "NO",
					"level": 1,
					"name": "item4",
					"path": "1003",
					"remark": "test3"
				},
				{
					"code": "101101",
					"createTime": "2017-09-23 04:35:08",
					"dictId": "c9ac7dde9ddd11e7ae293cbf104428a9",
					"enable": "YES",
					"id": 1004,
					"leaf": "NO",
					"level": 2,
					"name": "item5",
					"path": "1003.1004",
					"remark": "test4"
				},
				{
					"code": "101101101",
					"createTime": "2017-09-23 04:36:12",
					"dictId": "c9ac7dde9ddd11e7ae293cbf104428a9",
					"enable": "YES",
					"id": 1005,
					"leaf": "YES",
					"level": 3,
					"name": "item6",
					"path": "1003.1004.1005",
					"remark": "test5"
				}
            ],
            buttons:[
                {
                    type:"success",
                    text:"删除",
								},
								{
									type:"success",
									text:"编辑",
								},
									{
									type:"success",
									text:"添加子菜单",
								},
								
            ]  
    }
    
  },  
  components: {  
    SyncTreeGrid  
	},
	methods:{
		syncTreeGridData(arr){
			console.log(arr)
		}
	}  
}  
</script>  
  
<!-- Add "scoped" attribute to limit CSS to this component only -->  
<style scoped>  
</style>  