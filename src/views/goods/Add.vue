<template>
	<div>
		<!-- 面包屑导航区 -->
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/home   ' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>商品管理</el-breadcrumb-item>
			<el-breadcrumb-item>添加列表</el-breadcrumb-item>
		</el-breadcrumb>
		<el-card>
			<el-alert title="添加商品信息" type="info" center show-icon :closable="false"> </el-alert>
			<el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
				<el-step title="基本属性"></el-step>
				<el-step title="商品参数"></el-step>
				<el-step title="商品属性"></el-step>
				<el-step title="商品图片"></el-step>
				<el-step title="商品内容"></el-step>
				<el-step title="完成"></el-step>
			</el-steps>
			<!-- tab栏区 -->
			<el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
				<el-tabs :tab-position="'left'" v-model="activeIndex" :before-leave="beforTabLeave" @tab-click="tabCliked">
					<el-tab-pane label="基本信息" name="0">
						<el-form-item label="商品名称" prop="goods_name">
							<el-input v-model="addForm.goods_name"></el-input>
						</el-form-item>
						<el-form-item label="商品价格" prop="goods_price">
							<el-input v-model="addForm.goods_price" type="number"></el-input>
						</el-form-item>
						<el-form-item label="商品重量" prop="goods_weight">
							<el-input v-model="addForm.goods_weight" type="number"></el-input>
						</el-form-item>
						<el-form-item label="商品数量" prop="goods_number">
							<el-input v-model="addForm.goods_number" type="number"></el-input>
						</el-form-item>
						<el-form-item label="商品分类" prop="goods_cat">
							<el-cascader
								v-model="addForm.goods_cat"
								:options="catelist"
								:props="{
									expandTrigger: 'hover',
									label: 'cat_name',
									value: 'cat_id',
									children: 'children',
								}"
								@change="handleChange"
							></el-cascader>
						</el-form-item>
					</el-tab-pane>
					<el-tab-pane label="商品参数" name="1">
						<el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
							<!-- 复选框组 -->
							<el-checkbox-group v-model="item.attr_vals">
								<el-checkbox border :label="cb" v-for="(cb, i) in item.attr_vals" :key="i"> </el-checkbox>
							</el-checkbox-group>
						</el-form-item>
					</el-tab-pane>
					<el-tab-pane label="商品属性" name="2"
						><el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id"> <el-input v-model="item.attr_vals"></el-input> </el-form-item
					></el-tab-pane>
					<el-tab-pane label="商品图片" name="3">
						<el-upload
							action="http://127.0.0.1:8888/api/private/v1/upload"
							:on-preview="handlePreview"
							:on-remove="handleRemove"
							list-type="picture"
							:headers="heraderObje"
							:on-success="handleSuccess"
						>
							<el-button size="small" type="primary">点击上传</el-button>
						</el-upload>
					</el-tab-pane>
					<el-tab-pane label="商品内容" name="4">
						<quill-editor v-model="addForm.goods_introduce"></quill-editor>
						<!-- 添加商品按钮 -->
						<el-button type="primary" class="AddGoods" @click="add">添加商品</el-button>
					</el-tab-pane>
				</el-tabs>
			</el-form>
		</el-card>

		<!-- 图片预览对话框 -->
		<el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
			<img :src="previewPath" alt="" class="previewImg" />
		</el-dialog>
	</div>
</template>

<script>
import { getCateList, getCateAttr, postGoods } from "@/api/index";
import _ from "lodash";

export default {
	created() {
		this.getCateList();
	},
	data() {
		return {
			activeIndex: "0",
			addForm: {
				goods_name: "",
				goods_price: 0,
				goods_weight: 0,
				goods_number: 0,
				goods_cat: [],
				pics: [],
				//商品的详情描述
				goods_introduce: "",
				attrs: [],
			},
			catelist: [],
			addFormRules: {
				goods_name: [
					{
						required: true,
						message: "请输入商品名称",
						trigger: "blur",
					},
				],
				goods_price: [
					{
						required: true,
						message: "请输入商品价格",
						trigger: "blur",
					},
				],
				goods_weight: [
					{
						required: true,
						message: "请输入商品重量",
						trigger: "blur",
					},
				],
				goods_number: [
					{
						required: true,
						message: "请输入商品数量",
						trigger: "blur",
					},
				],
				goods_cat: [
					{
						required: true,
						message: "请选择商品分类",
						trigger: "blur",
					},
				],
			},
			manyTableData: [],
			onlyTableData: [],
			heraderObje: {
				Authorization: window.sessionStorage.getItem("token"),
			},
			previewPath: "",
			previewVisible: false,
		};
	},
	methods: {
		// 获取所有商品列表
		async getCateList() {
			const res = await getCateList();
			if (res.meta.status !== 200) {
				return this.$message.error("获取商品列表失败");
			}
			//   console.log(res)
			this.catelist = res.data;
		},
		handleChange() {
			if (this.addForm.goods_cat.length !== 3) {
				this.addForm.goods_cat = [];
			}
		},
		beforTabLeave(activeName, oldActiveName) {
			//   console.log(activeName, oldActiveName)
			if (oldActiveName === "0" && this.addForm.goods_cat.length !== 3) {
				this.$message.error("请选择商品分类");
				return false;
			}
		},
		async tabCliked() {
			// 证明访问的是动态参数
			if (this.activeIndex === "1") {
				const res = await getCateAttr(this.cateId, { sel: "many" });
				// console.log(res)
				if (res.meta.status !== 200) return this.$message.error("获取动态属性失败");
				res.data.forEach(item => {
					item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(" ");
					//   console.log(item.attr_vals)
				});

				this.manyTableData = res.data;
			} else if (this.activeIndex === "2") {
				const res = await getCateAttr(this.cateId, { sel: "only" });

				if (res.meta.status !== 200) return this.$message.error("获取静态属性失败");
				this.onlyTableData = res.data;
			}
		},
		//处理图片预览效果
		handlePreview(file) {
			console.log(file);
			this.previewPath = file.response.data.url;
			this.previewVisible = true;
		},
		//处理移除图片的操作
		handleRemove(file) {
			console.log(file);
			const filePath = file.response.data.tmp_path;
			const i = this.addForm.pics.findIndex(x => {
				x.pic === filePath;
			});
			this.addForm.pics.splice(i, 1);
			console.log(this.addForm);
		},
		//监听图片上传成功的事件
		handleSuccess(response) {
			console.log(response);
			const picInfo = { pic: response.data.tmp_path };
			this.addForm.pics.push(picInfo);
			console.log(picInfo);
		},
		//添加商品
		add() {
			this.$refs.addFormRef.validate(async valid => {
				if (!valid) {
					return this.$message.error("请填写必要的表单项");
				}
				const form = _.cloneDeep(this.addForm);
				form.goods_cat = form.goods_cat.join(",");
				//处理动态参数和静态属性
				this.manyTableData.forEach(item => {
					const newInfo = {
						attr_id: item.attr_id,
						attr_value: item.attr_vals.join(" "),
					};
					this.addForm.attrs.push(newInfo);
				});
				this.onlyTableData.forEach(item => {
					const newInfo = {
						attr_id: item.attr_id,
						attr_value: item.attr_vals.join,
					};
					this.addForm.attrs.push(newInfo);
				});
				form.attrs = this.addForm.attrs;
				console.log(form);

				//发起请求添加商品
				//商品名称必须是唯一的
				const res = await postGoods(form);
				console.log(res);
				if (res.meta.status !== 201) return this.$message.error(res.meta.msg);
				this.$message.success("添加商品成功");
				this.$router.push("/goods");
			});
		},
	},
	computed: {
		cateId() {
			if (this.addForm.goods_cat.length === 3) {
				return this.addForm.goods_cat[2];
			}
			return null;
		},
	},
};
</script>

<style lang="scss" scoped>
.el-checkbox {
	margin: 0 10px 0 0 !important;
}
.previewImg {
	width: 100%;
}
.AddGoods {
	margin-top: 15px;
}
</style>
