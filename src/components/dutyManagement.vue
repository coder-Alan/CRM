<template>
<!-- 值班信息表 -->
    <div class="user-container">
        <div class="filter-container">
            <label for="searchName" class="filter-name">值班编号：</label>
            <el-input id="searchName" v-model.trim="listQuery.zCode" style="padding-right:20px;" class="filter-input" placeholder="请输入" />
            <label for="searchNickName" class="filter-name">值班人员：</label>
            <el-input id="searchNickName" v-model.trim="listQuery.zPeople" class="filter-input" placeholder="请输入" />
            <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">搜索</el-button>
            <el-button v-waves class="filter-item" type="success" icon="el-icon-edit" @click="handleAdd">添加</el-button>
            <el-button v-waves class="filter-item" type="danger" icon="el-icon-delete" @click="handleRemove">批量删除</el-button>
        </div>

        <el-table
        v-loading="listLoading"
        :data="list"
        border
        fit
        highlight-current-row
        class="user-table"
        @selection-change="handleSelectionChange"
        >
            <el-table-column
                type="selection"
                align="center"
                width="50"
            />
            <el-table-column label="值班编号" width="120" align="center">
                <template slot-scope="scope">
                    <span>{{ scope.row.zCode }}</span>
                </template>
            </el-table-column>
            <el-table-column label="值班人员" width="170" align="center">
                <template slot-scope="scope">
                    <span>{{ scope.row.zPeople }}</span>
                </template>
            </el-table-column>
            <el-table-column label="所属科室" width="200" align="center">
                <template slot-scope="scope">
                    <span>{{ scope.row.zRoom }}</span>
                </template>
            </el-table-column>
            <el-table-column label="值班地点" width="200" align="center">
                <template slot-scope="scope">
                    <span>{{ scope.row.zPlaces }}</span>
                </template>
            </el-table-column>
            <el-table-column label="值班星期" width="200" align="center">
                <template slot-scope="scope">
                    <span>{{ scope.row.zDay }}</span>
                </template>
            </el-table-column>
            <el-table-column label="值班班次" width="200" align="center">
                <template slot-scope="scope">
                    <span>{{ scope.row.zClasses }}</span>
                </template>
            </el-table-column>
            <el-table-column label="备注" align="center">
                <template slot-scope="scope">
                    <span>{{ scope.row.remarks }}</span>
                </template>
            </el-table-column>
            <el-table-column label="操作" width="150" align="center">
                <template slot-scope="scope">
                <el-button type="success" size="mini" @click="handleEdit(scope.row)">编辑</el-button>
                <el-button type="danger" size="mini" @click="handleDelete(scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>

        <Pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.pageSize" @pagination="getList" />

        <!-- 添加、编辑对话框 -->
        <el-dialog :title="dialogTitle" :visible.sync="dialogFormVisible" width="30%">
            <el-form :model="ruleForm" status-icon :rules="rules" ref="ruleForm" label-width="90px" class="form">
                <el-form-item label="值班编号:" prop="zCode">
                    <el-input type="text" v-model="ruleForm.zCode" @blur='testZCode' autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="值班人员:" prop="zPeople">
                    <el-input type="text" v-model="ruleForm.zPeople" @blur='testZPeople' autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="所属科室:" prop="zRoom">
                    <el-input type="text" v-model="ruleForm.zRoom" autocomplete="off" :disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="值班地点:" prop="zPlaces">
                    <el-input type="text" v-model="ruleForm.zPlaces" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="值班星期:" prop="zDay" required>
                    <el-select v-model="ruleForm.zDay" placeholder="请选择">
                        <el-option
                        v-for="item in weeks"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="值班班次:" prop="zClasses" required>
                    <el-select v-model="ruleForm.zClasses" placeholder="请选择">
                        <el-option
                        v-for="item in zClassesList"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="备注:" prop="remarks">
                    <el-input type="textarea" :rows="2" placeholder="请输入内容" v-model="ruleForm.remarks" autocomplete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="handleAddCancel">取 消</el-button>
                <el-button type="primary" @click="handleAddSure('ruleForm')">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
import {
    addDuty,
    queryDutyList,
    testZCode,
    testZPeople,
    queryAllBName,
    querySingleDuty,
    updateDuty,
    deleteDuty,
} from '../api/duty'
import waves from '../directive/waves' // 按钮水波纹
import Pagination from './Pagination/index' // secondary package based on el-pagination

export default {
    data() {
        var validateZCode = (rule, value, callback) => {
            setTimeout(() => {
                if (this.isRegisteredZCode == 201) {
                    callback(new Error('该值班编号已存在'));
                    this.isRegisteredZCode = ''
                } else if (value === '') {
                    callback(new Error('请输入值班编号'));
                } else {
                    callback();
                }
            }, 500)
        };
        var validateZPeople = (rule, value, callback) => {
            setTimeout(() => {
                if (this.isRegisteredZPeople == 201) {
                    callback(new Error('该员工不存在'));
                    this.isRegisteredZPeople = ''
                } else if (value === '') {
                    callback(new Error('请输入员工姓名'));
                } else {
                    callback();
                }
            }, 500)
        };
        var validateZPlaces = (rule, value, callback) => {
            if (value === '') {
                callback(new Error('请输入值班地点'));
            } else {
                callback();
            }
        };
        var validateZDay = (rule, value, callback) => {
            if (value === '') {
                callback(new Error('请输入值班星期'));
            } else {
                callback();
            }
        };
        return {
            list: [],
            total: 0,
            listQuery: {
                zCode: '',
                zPeople: '',
                page: 1,
                pageSize: 10
            },
            listLoading: false, // 加载中
            dialogFormVisible: false, // 添加、编辑弹出框
            dialogTitle: '',
            // 添加、编辑表单验证
            ruleForm: {
                zCode: '',
                zPeople: '',
                zRoom: '',
                zPlaces: '',
                zDay: '',
                zClasses: '',
                remarks: '',
            },
            rules: {
                zCode: [{ validator: validateZCode, trigger: 'blur' }],
                zPeople: [{ validator: validateZPeople, trigger: 'blur' }],
                zPlaces: [{ validator: validateZPlaces, trigger: 'blur' }],
                // zDay: [{ validator: validateZDay, trigger: 'blur' }],
            }, 
            handle: '', // 当前操作
            currentDelete: '', // 当前删除(单个/批量)对象
            weeks: [{
                value: '星期一',
                label: '星期一'
            }, {
                value: '星期二',
                label: '星期二'
            }, {
                value: '星期三',
                label: '星期三'
            }, {
                value: '星期四',
                label: '星期四'
            }, {
                value: '星期五',
                label: '星期五'
            }, {
                value: '星期六',
                label: '星期六'
            }, {
                value: '星期天',
                label: '星期天'
            }],
            zClassesList: [],
        }
    },
    watch: {
    },
    components: { 
        Pagination,
    },
    directives: { waves },
    created() {
        this.getList()
        queryAllBName().then(res => {
            let data = res.data.data
            if (data.code == 200) {
                data.data.forEach(item => {
                    this.zClassesList.push({
                        value: item.bName,
                        label: item.bName
                    })
                })
            }
        }), (err) => {
            console.log(err)
        }
    },
    methods: {
        getList() {
            this.listLoading = true
            queryDutyList(this.listQuery).then(res => {
                let data = res.data.data
                if (data.code == 200) {
                    this.listLoading = false
                    this.list = data.data
                    this.total = data.total
                }
            }), (err) => {
                console.log(err)
            }
        },
        handleSelectionChange(val) {
            let zCodeList = []
            val.forEach(item => {
                zCodeList.push(item.zCode)
            })
            let str = ''
            zCodeList.forEach(item => {
                str += ("'" + item + "',")
            })
            str = str.slice(0, str.lastIndexOf(','))
            this.currentDelete = str
        },
        handleFilter() {
            this.listLoading = true
            this.listQuery.page = 1
            this.listQuery.pageSize = 10
            queryDutyList(this.listQuery).then(res => {
                let data = res.data.data
                if (data.code == 200) {
                    this.listLoading = false
                    this.list = data.data
                    this.total = data.data.length
                }
            }), (err) => {
                console.log(err)
            }
        },
        handleAdd() {
            this.dialogTitle = '添加'
            this.handle = 'add'
            this.dialogFormVisible = true
            this.ruleForm = {
                zCode: '',
                zPeople: '',
                zRoom: '',
                zPlaces: '',
                zDay: '',
                zClasses: '',
                remarks: '',
            }
        },
        handleRemove() {
            this.$confirm('是否确定批量删除选中项', '批量删除', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                if (this.currentDelete != '') {
                    deleteDuty({zCode: this.currentDelete}).then(res => {
                        let data = res.data.data
                        if (data.code == 200) {
                            this.listLoading = false
                            this.$message({
                                message: '批量删除成功',
                                type: 'success'
                            });
                        }
                        this.getList()
                    }), (err) => {
                        console.log(err)
                    }
                } else {
                    this.$message({
                    type: 'info',
                    message: '请选择删除项'
                }); 
                }
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });          
            });
        },
        handleEdit(val, formName) {
            this.dialogTitle = '编辑'
            this.handle = 'edit'
            this.dialogFormVisible = true
            this.ruleForm.zCode = val.zCode
            this.ruleForm.zPeople = val.zPeople
            this.ruleForm.zRoom = val.zRoom
            this.ruleForm.zPlaces = val.zPlaces
            this.ruleForm.zDay = val.zDay
            this.ruleForm.zClasses = val.zClasses
            this.ruleForm.remarks = val.remarks
        },
        handleDelete(val) {
            this.currentDelete = "'" + val.zCode + "'"
            this.$confirm('是否确定删除', '删除', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                deleteDuty({zCode: this.currentDelete}).then(res => {
                    let data = res.data.data
                    if (data.code == 200) {
                        this.listLoading = false
                        this.$message({
                            message: data.message,
                            type: 'success'
                        });
                    }
                    this.getList()
                }), (err) => {
                    console.log(err)
                }
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });          
            });
        },
        handleAddSure(formName) {
            if (this.handle == 'add') {
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        this.dialogFormVisible = false
                        console.log('提交的表单', this.ruleForm)
                        addDuty(this.ruleForm).then(res => {
                            let data = res.data.data
                            if (data.code == 200) {
                                this.$message({
                                    message: '添加成功',
                                    type: 'success'
                                });
                                this.ruleForm = {
                                    zCode: '',
                                    zPeople: '',
                                    zRoom: '',
                                    zPlaces: '',
                                    zDay: '',
                                    zClasses: '',
                                    remarks: '',
                                }
                                this.getList()
                            }
                        })
                    } else {
                        console.log('表单验证失败')
                    }
                })
            } else if (this.handle == 'edit') {
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        this.dialogFormVisible = false
                        console.log('提交的表单', this.ruleForm)
                        updateDuty(this.ruleForm).then(res => {
                            let data = res.data.data
                            if (data.code == 200) {
                                this.$message({
                                    message: data.message,
                                    type: 'success'
                                });
                                this.ruleForm = {
                                    zCode: '',
                                    zPeople: '',
                                    zRoom: '',
                                    zPlaces: '',
                                    zDay: '',
                                    zClasses: '',
                                    remarks: '',
                                }
                                this.getList()
                            }
                        })
                    } else {
                        console.log('表单验证失败')
                    }
                })
            }
            this.handle = ''
        },
        handleAddCancel() {
            this.dialogFormVisible = false
            this.ruleForm = {
                zCode: '',
                zPeople: '',
                zRoom: '',
                zPlaces: '',
                zDay: '',
                zClasses: '',
                remarks: '',
            }
        },
        testZCode() {
            if (this.ruleForm.zCode != '') {
                testZCode({zCode: this.ruleForm.zCode}).then(res => {
                    if (res.data.data.code == 201) {
                        this.isRegisteredZCode = 201
                    } else {
                        this.isRegisteredZCode = ''
                    }
                })
            }
        },
        testZPeople() {
            if (this.ruleForm.zPeople != '') {
                testZPeople({zPeople: this.ruleForm.zPeople}).then(res => {
                    if (res.data.data.code == 201) {
                        this.isRegisteredZPeople = 201
                    } else {
                        this.isRegisteredZPeople = ''
                    }
                    if (res.data.data.code == 200) {
                        // 科室
                        this.ruleForm.zRoom = res.data.data.data.yDepartment
                    } else {
                        this.ruleForm.zRoom = ''
                    }
                })
            }
        },
    },
}
</script>

<style lang="scss">
.user-container {
    .user-table {
        width: 99%;
        margin-top: 20px;
    }
    .filter-name {
        font-size: 14px;
    }
    .filter-input {
        width: 200px;
    }
    .filter-item {
        margin-left: 10px;
    }
    .el-table th.gutter{
        display: table-cell!important;
    }
    .el-upload__tip {
        position: absolute;
        left: 120px;
        bottom: 0;
        font-size: 12px;
        color: #606266;
        margin-top: 7px;
    }
    .avatar-uploader .el-upload {
        border: 1px dashed #d9d9d9;
        border-radius: 6px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
    }
    .avatar-uploader .el-upload:hover {
        border-color: #409EFF;
    }
    .avatar-uploader-icon {
        font-size: 20px;
        color: #8c939d;
        width: 100px;
        height: 100px;
        line-height: 100px;
        text-align: center;
    }
    .avatar {
        width: 100px;
        height: 100px;
        display: block;
    }
    .el-dialog__wrapper {
        display: flex;
        justify-content: center;
        align-items: center;
        .el-dialog {
            margin: 0 !important;
        }
    }
    .el-dialog__body {
        padding: 0 30px;
    }
    .el-dialog__footer {
        padding: 0 30px 20px;
    }
}
</style>
