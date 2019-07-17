<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="关键字"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getProducts">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="displayProperties">显示属性</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="displaySkuProperties">Sku属性</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="onSale">上架</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="offSale">下架</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="products" highlight-current-row v-loading="listLoading" @selection-change="selsChange"
				  @row-click="getProductRow"  style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" width="60">
			</el-table-column>
			<el-table-column prop="name" :show-overflow-tooltip="true" label="标题" width="180" sortable>
			</el-table-column>
			<el-table-column prop="subName" :show-overflow-tooltip="true" label="副标题" width="200" sortable>
			</el-table-column>
			<el-table-column prop="onSaleTimeStr" label="上架时间" width="200" sortable>
			</el-table-column>
			<el-table-column prop="offSaleTimeStr" label="下架时间" width="200" sortable>
			</el-table-column>
			<el-table-column prop="state" label="状态" width="200" :formatter="formatState" sortable>
				<!--<template slot-scope="scope">
                    <el-popover placement="top">


                        <div v-if="scope.row.state ==1" slot="reference" class="name-wrapper">
                            <el-tag size="medium" class="stateStyle">上架</el-tag>
                        </div>
                        <div v-else-if="scope.row.state ==0" slot="reference" class="name-wrapper">
                            <el-tag size="medium">下架</el-tag>
                        </div>
                    </el-popover>
                </template>-->
			</el-table-column>
			<el-table-column label="操作" width="150">
				<template scope="scope">
					<el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
					<el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="10" :total="total" style="float:right;">
			</el-pagination>
		</el-col>
		<!--sku属性编辑页面-->
		<el-dialog title="sku属性编辑" v-model="displaySkuPropertiesVisible" :close-on-click-modal="false">
			<el-card class="box-card">
				<div v-for="p in skuProperties" :key="p" class="text item">
					{{ p.specName }}:
					<!--sku的值的输入框的多少是根据你的值的多少来显示的:
                      后台的属性字段中有一个skuValues的字段:是一个字符串的数组
                      p.skuValues
                      i in 3====>3

                    -->
					<div v-for="i in p.skuValues.length+1" :key="i">
						<el-input v-model="p.skuValues[i-1]" auto-complete="off" style="width: 80%"></el-input>
						<!--splice(i-1,1):数组的移除:
                            　splice(index,len,[item])    注释：该方法会改变原始数组。
                            splice有3个参数，它也可以用来替换/删除/添加数组内某一个或者几个值
                            index:数组开始下标        len: 替换/删除的长度       item:替换的值，删除操作的话 item为空
                          -->
						<el-button type="primary" @click="p.skuValues.splice(i-1,1)" :loading="editLoading">删除
						</el-button>
					</div>
				</div>
			</el-card>
			<!--table-->
			<!--动态表格-->
			<el-table :data="skuDatas">
				<!--cols一堆列,col哪一列-->
				<template v-for="(col ,index) in cols">
					<el-table-column :prop="col.prop" sortable :label="col.label" v-if="['price','availableStock'].includes(col.prop)">
						<!--scope:作用域的问题-->
						<template scope="scope">
							<el-input auto-complete="off" v-model="skuDatas[scope.$index].price"  style="width: 400px" v-if="'price'===col.prop"/>
							<el-input auto-complete="off" v-model="skuDatas[scope.$index].availableStock" style="width: 400px" v-if="'availableStock'===col.prop"/>
						</template>
					</el-table-column>
					<!--只做显示-->
					<el-table-column :prop="col.prop" sortable :label="col.label" v-if="!['price','availableStock'].includes(col.prop)">
					</el-table-column>
				</template>
			</el-table>


			<div slot="footer" class="dialog-footer">
				<el-button @click.native="displaySkuPropertiesVisible = false">取消</el-button>
				<el-button type="primary" @click.native="skuPropertiesSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--显示属性编辑页面-->
		<el-dialog title="显示属性编辑" v-model="displayPropertiesVisible" :close-on-click-modal="false">
			<!--遍历：viewProperties-->
			<el-card class="box-card">
				<div v-for="o in viewProperties" :key="o" class="text item">
					{{o.specName}}:<el-input v-model="o.viewValue" auto-complete="off"></el-input>
				</div>
			</el-card>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="displayPropertiesVisible = false">取消</el-button>
				<el-button type="primary" @click.native="viewPropertiesSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>
		<!--编辑/添加界面-->
		<el-dialog title="编辑" v-model="formVisible" :close-on-click-modal="false">
			<el-form :model="form" label-width="80px" :rules="formRules" ref="form">
				<el-form-item label="标题" prop="name">
					<el-input v-model="form.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="副标题" prop="subName">
					<el-input v-model="form.subName" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="商品分类" prop="productType">
					<select-tree width="200" :options="productTypes" v-model="form.productType"/>
				</el-form-item>

				<el-form-item label="品牌" prop="brandId">
					<el-select v-model="form.brandId" clearable placeholder="请选择">
						<el-option
								v-for="item in brands"
								:key="item.id"
								:label="item.name"
								:value="item.id">
						</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="简介" prop="productExt.description">
					<el-input v-model="form.productExt.description" auto-complete="off"></el-input>
				</el-form-item>

				<el-form-item label="详情" prop="productExt.richContent">
					<div class="edit_container">
						<quill-editor v-model="form.productExt.richContent" ref="myQuillEditor" class="editer"
									  :options="editorOption" @ready="onEditorReady($event)"></quill-editor>
					</div>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="formVisible = false">取消</el-button>
				<el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>
	</section>
</template>

<script>
    import SelectTree from '@/components/SelectTree.vue';
    import { quillEditor } from "vue-quill-editor"; //调用编辑器
    import 'quill/dist/quill.core.css'
   import 'quill/dist/quill.snow.css'
   import 'quill/dist/quill.bubble.css'
    export default {
        name: 'about',
		components: {
             SelectTree, quillEditor
		   },
        data() {
            return {
                editorOption: {},
                // 默认选中值
				selected: 'A',
                      // 数据默认字段
				defaultProps: {
					    parent: 'pid',   // 父级唯一标识
					    value: 'id',          // 唯一标识
					    label: 'name',       // 标签显示
					    children: 'children', // 子级
					  },
                productTypes:[],
                filters: {
                    keyword: ''
                },
                products: [],
				brands:[],
				productView:null,//选中行的数据
                total: 0,
                page: 1,
                listLoading: false,
                sels: [],//列表选中列
                fileList2: [],
                formVisible: false,//编辑界面是否显示
                viewProperties:[],
                displayPropertiesVisible:false,//显示属性页面是否显示
                skuProperties:[],
                displaySkuPropertiesVisible:false,
                skuDatas: [],//sku的属性
                cols:[],//table的头的数组
                editLoading: false,
                formRules: {
                    name: [
                        { required: true, message: '请输入姓名', trigger: 'blur' }
                    ]
                },
                //编辑界面数据
                //编辑界面数据:定义模型
                form: {
                    name: '',
                    subName: '',
                    brandId: '',
                    productType: '',
                    productExt: {"description": "", "richContent": ""}
                },
            }
        },
        watch:{
            skuProperties:{
                handler(curVal,oldVal){
                    // 过滤掉用户没有填写数据的规格参数
                    const arr = this.skuProperties.filter(s => s.skuValues.length > 0);
                    // 通过reduce进行累加笛卡尔积
                    var skus =  arr.reduce((last, spec) => {
                        const result = [];
                        last.forEach(o => {
                            spec.skuValues.forEach(option => {
                                // option //一个一一个值 黄皮肤
                                const obj = {};
                                Object.assign(obj, o);
                                obj[spec.specName] = option;
                                result.push(obj);
                            })
                        })
                        return result
                    }, [{}]);
                    //添加不存在的列
                    skus.forEach(function (item) {
                        item['price'] = '';
                        item['availableStock'] = '';
                    })
                    this.skuDatas = skus;

                    let headers = [];
                    //现在没有一定有字段 库存 价格  颜色
                    //skus [{"身高":170,"三维":"xxx",价格:18,库存:18,是否可用:0},{"身高":170,"三维":"xxx",价格:18,库存:18,是否可用:0}]
                    //获取这个对象的所有的key
                    Object.keys(this.skuDatas[0]).forEach(sku=>{
                        let value = sku;
                        if(sku=='price'){
                            value = '价格'
                        }
                        if(sku=='availableStock'){
                            value = '库存'
                        }
                        let col =  {"label":value,"prop":sku};
                        headers.push(col);
                    });
                    this.cols = headers;
                },
                deep:true
            }
        },

        computed: {
            editor() {
                return this.$refs.myQuillEditor.quill
            }},
        methods: { //方法\
            offSale(){
                var ids = this.sels.map(item => item.id).toString();
                this.$confirm('确认下架选中商品吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    //NProgress.start();
                    let para = { ids: ids ,
                        type:2
                    };
                    this.$http.post("/product/product/onSaleOrOffSale",para).then((res) => {
                        if(res.data.success){
                            this.$message({
                                message: '操作成功!',
                                type: 'success'
                            });
                        }else{
                            this.$message({
                                message: '操作失败!',
                                type: 'error'
                            });
                        }
                    });
                });
			},
            onSale(){
                console.debug("666666666666666666699999999999999999999999999")
                console.debug(this.productView);
                var ids = this.sels.map(item => item.id).toString();
                this.$confirm('确认上架选中商品吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    //NProgress.start();
                    let para = { ids: ids ,
                    			type:1
                    };
                    this.$http.post("/product/product/onSaleOrOffSale",para).then((res) => {
                        if(res.data.success){
                            this.$message({
                                message: '操作成功!',
                                type: 'success'
                            });
                        }else{
                            this.$message({
                                message: '操作失败!',
                                type: 'error'
                            });
                        }
                    });
                });
			},
            skuPropertiesSubmit(){
                let para = {"productId":this.productView.id,
                    "skuProperties":this.skuProperties,
                    "skuDatas":this.skuDatas,
                }
                console.debug(para);
                this.$http.post("/product/product/updateSkuProperties",para).then((res) => {
                    if(res.data.success){
                        this.$message({
                            message: '操作成功!',
                            type: 'success'
                        });
                    }else{
                        this.$message({
                            message: '操作失败!',
                            type: 'error'
                        });
                    }
                });
			},
            viewPropertiesSubmit(){
                        let para = {"productId":this.productView.id,
									"ViewsProperties":this.viewProperties
                        }

                        this.$http.post("/product/productExt/updateViewsProperties",para).then((res) => {
                            if(res.data.success){
                                this.$message({
                                    message: '操作成功!',
                                    type: 'success'
                                });
                            }else{
                                this.$message({
                                    message: '操作失败!',
                                    type: 'error'
                                });
                            }
                        });
			},
            getProductRow(row){
                console.debug("=================================================")
                console.debug(row);
                this.productView=row;

			},
            displaySkuProperties(){
                if(this.productView){
                    var productType=this.productView.productType;
                    var productId=this.productView.id;
                    //后台查询显示属性数据返回  viewProperties
                    this.$http.get("/product/specification/querySkuProperties/"+productType+"/"+productId)
                    // this.$http.get("/product/specification/querySkuProperties/"+productType)
                        .then(res=>{
                            console.log(res);
                            this.skuProperties = res.data;
                        });
                    this.displaySkuPropertiesVisible=true;

                }else {
                    this.$message({
                        message: '请选中后再编辑!',
                        type: 'warning'
                    });
                }
			},
            displayProperties(){
                if(this.productView){
                    var productType=this.productView.productType;
                    var productId=this.productView.id;
                    console.debug("产品类型"+productType);
                    //后台查询显示属性数据返回  viewProperties
                    this.$http.get("/product/specification/queryViewProperties/"+productType+"/"+productId)
                        .then(res=>{
                            console.log(res);
                            this.viewProperties = res.data;
                        });
                    this.displaySkuPropertiesVisible=true;

				}else {
                    this.$message({
                        message: '请选中后再编辑!',
                        type: 'warning'
                    });
				}
			},
            onEditorReady(editor) {
            },
            getTreeData(){
                // 发送一个异步请求: get请求 /product/productType/treeData
                this.$http.get("/product/productType/treeData")
                    .then(res=>{
                        console.log(this);
                        this.productTypes = res.data;
                    });

            },
            getBrands() {
                //异步请求:
                this.$http.get("/product/brand/list")
                    .then((res) => {
                        console.debug("888888888888888888888")
                        console.log(res);
                        this.brands=res.data;
                    });
            },
            formatState: function (row, column) {
                return row.state === 1 ? '下架' : '上架'
            },
            handleSuccess(response, file, fileList){
                console.debug(file);
                //上传成功回调
                this.form.logo = file.response.object;
            },
            handleRemove(file, fileList) {
                var filePath =file.response.object;
                this.$http.get("/common/common/delete?filePath="+filePath)
                    .then(res=>{
                        if(res.data.success){
                            this.$message({
                                message: '删除成功!',
                                type: 'success'
                            });
                        }else{
                            this.$message({
                                message: '删除失败!',
                                type: 'error'
                            });
                        }
                    })
            },
            handlePreview(file) {
                console.log(file);
            },
            handleCurrentChange(val) {
                this.page = val;
                this.getProducts();
            },
            //获取品牌的列表:
            getProducts() {
                //查询条件
                let para = {
                    page: this.page,
                    // keyword: this.filters.keyword
                };
                //加载
                this.listLoading = true;
                //异步请求:
                this.$http.post("/product/product/json",para)
                    .then((res) => {
                        console.log(res);
                        this.total = res.data.total;
                        this.products = res.data.rows;
                        this.listLoading = false;
                    });
            },
            //删除
            handleDel: function (index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    let id =  row.id ;
                    this.$http.delete("/product/product/delete/"+id)
                        .then((res) => {
                            let { msg, success, object } = res.data;
                            if (!success) {
                                this.$message({
                                    message: msg,
                                    type: 'error'
                                });
                            } else {
                                this.listLoading = false;
                                //NProgress.done();
                                this.$message({
                                    message: '删除成功',
                                    type: 'success'
                                });
                                this.getProducts();
                            }
                        });
                }).catch(() => {
                });
            },
            //显示编辑界面
            handleEdit: function (index, row) {
                console.debug("55555555555555");
                console.debug(row);
                this.formVisible = true;
                //数据回显
                this.form =row;

            },
            //显示新增界面
            handleAdd: function () {
                this.formVisible = true;
                this.form = {
                    name: '',
                    subName: '',
                    brandId: '',
                    productType: '',
                    productExt: {"description": "", "richContent": ""}
                };
            },
            //编辑
            editSubmit: function () {
                this.$refs.form.validate((valid) => {
                    if (valid) {
                        console.log(this.fileList2);
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.editLoading = true;
                            let para = Object.assign({}, this.form);
                            this.$http.post("/product/product/save",para).then((res) => {
                                this.editLoading = false;
                                this.$message({
                                    message: '提交成功',
                                    type: 'success'
                                });
                                this.$refs['form'].resetFields();
                                this.formVisible = false;
                                this.getProducts();
                            });
                        });
                    }
                });
            },
            selsChange: function (sels) {
                this.sels = sels;
            },
            //批量删除
            batchRemove: function () {
                var ids = this.sels.map(item => item.id).toString();
                this.$confirm('确认删除选中记录吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    let para = { ids: ids };
                    batchRemoveUser(para).then((res) => {
                        this.listLoading = false;
                        //NProgress.done();
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                        this.getProducts();
                    });
                }).catch(() => {

                });
            }
        }, // $(function()) 加载完毕后执行
        mounted() {
            this.getProducts();
            this.getBrands();
            this.getTreeData();
        }
    }

</script>

<style scoped>

</style>