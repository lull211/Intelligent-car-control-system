<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="任务名称" prop="taskName">
        <el-input
          v-model="queryParams.taskName"
          placeholder="请输入任务名称"
          clearable
          size="small"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="任务类型" prop="taskType">
        <el-select v-model="queryParams.taskType" placeholder="请选择任务类型" clearable size="small">
          <el-option v-for="item in TypeList" :key="item.id"  :label="item.taskType" :value="item.taskType">
          </el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="无人机编号" prop="taskDrone">
        <el-input
          v-model="queryParams.taskDrone"
          placeholder="请输入无人机编号"
          clearable
          size="small"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="驾驶员" prop="taskDriver">
        <el-select v-model="queryParams.taskDriver" placeholder="请选择驾驶员" clearable size="small">
          <el-option v-for="item in PilotsList" :key="item.id"  :label="item.driverName" :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="驾驶员手机号码" prop="driverPhone">
        <el-input
          v-model="queryParams.driverPhone"
          placeholder="请输入驾驶员手机号码"
          clearable
          size="small"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="航线" prop="taskAirline">
        <el-select v-model="queryParams.taskAirline" placeholder="请选择航线" clearable size="small">
          <el-option v-for="item in AirlineList" :key="item.id"  :label="item.airlineName" :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="任务地点" prop="taskAddress">
        <el-input
          v-model="queryParams.taskAddress"
          placeholder="请输入任务地点"
          clearable
          size="small"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="任务状态" prop="taskState">
        <el-select v-model="queryParams.taskState" placeholder="请选择驾驶员" clearable size="small">
          <el-option v-for="item in taskStateDic" :key="item.id"  :label="item.State" :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="el-icon-plus"
          size="mini"
          @click="handleAdd"
          v-hasPermi="['missionlist:missionlist:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="el-icon-edit"
          size="mini"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['missionlist:missionlist:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['missionlist:missionlist:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          v-hasPermi="['missionlist:missionlist:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="missionlistList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="任务名称" align="center" prop="taskName" />
      <el-table-column label="任务类型" align="center" prop="taskType" />
      <el-table-column label="任务时间" align="center" prop="taskTime" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.taskTime, '{y}-{m}-{d} {h}:{i}:{s}')}}</span>
        </template>
      </el-table-column>
      <el-table-column label="结束时间" align="center" prop="endTime" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.endTime, '{y}-{m}-{d} {h}:{i}:{s}')}}</span>
        </template>
      </el-table-column>
      <el-table-column label="无人机编号" align="center" prop="taskDrone" />
      <el-table-column label="驾驶员" align="center" prop="DiverName" />
      <el-table-column label="驾驶员手机号码" align="center" prop="DriverPhone" />
<!--      <el-table-column label="主键id" align="center" prop="id" />-->
      <el-table-column label="航线" align="center" prop="AirLineName" />

      <el-table-column label="任务封面图片" align="center" prop="taskImages">
        <template slot-scope="scope">
          <el-image :src="scope.row.taskImages">
          </el-image>
        </template>
      </el-table-column>

      <el-table-column label="任务地点" align="center" prop="taskAddress" />
<!--      <el-table-column label="删除码" align="center" prop="deleteCode" />-->
      <el-table-column label="描述" align="center" prop="extraExplain" />

      <el-table-column label="任务状态" align="center" prop="taskState">
        <template slot-scope="scope">
          <p v-if="scope.row.taskState == 0">
            未开始
          </p>
          <p v-if="scope.row.taskState == 1">
            进行中
          </p>
          <p v-if="scope.row.taskState == 2">
            已结束
          </p>
        </template>
      </el-table-column>

      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">

          <p v-if="scope.row.taskState == 2">

            <!-- 视频播放-->
            <el-dialog :title="title" :visible.sync="videoFlag" width="500px" height="500px">
              <vue-ali-player
                :useH5Prism=true
                ref="player"
                control-bar-visibility="hover"
                :source="url"
              ></vue-ali-player>
            </el-dialog>

            <el-button
              size="mini"
              type="text"
              @click="openVideo(scope.row)"
              v-hasPermi="['mymissionlist:mymissionlist:query']"
            >查看回放</el-button>

          </p>

          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['missionlist:missionlist:edit']"
          >修改</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['missionlist:missionlist:remove']"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改任务列表对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="任务名称" prop="taskName">
          <el-input v-model="form.taskName" placeholder="请输入任务名称" />
        </el-form-item>
        <el-form-item label="任务类型" prop="taskType">
          <el-select v-model="form.taskType" placeholder="请选择任务类型" clearable size="small">
            <el-option v-for="item in TypeList" :key="item.id"  :label="item.taskType" :value="item.taskType">
            </el-option>
          </el-select>

        </el-form-item>
        <el-form-item label="任务时间" prop="taskTime">
          <el-date-picker clearable size="small"
            v-model="form.taskTime"
            type="date"
            value-format="yyyy-MM-dd"
            placeholder="选择任务时间">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="无人机编号" prop="taskDrone">
          <el-input v-model="form.taskDrone" placeholder="请输入无人机编号" />
        </el-form-item>

        <el-form-item label="驾驶员" prop="taskDriver">
          <el-select v-model="form.taskDriver" placeholder="请选择驾驶员">
            <el-option v-for="item in PilotsList" :key="item.id"  :label="item.driverName+'  '+  item.driverPhone" :value="item.id.toString()">
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="航线" prop="taskAirline">
          <el-select v-model="form.taskAirline" placeholder="请选择航线" clearable size="small">
            <el-option v-for="item in AirlineList" :key="item.id"  :label="item.airlineName" :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="任务封面图片" prop="taskImages">
          <UploadImage v-model="form.taskImages" :limit="1"></UploadImage>
        </el-form-item>
        <el-form-item label="任务地点" prop="taskAddress">
          <el-input v-model="form.taskAddress" placeholder="请输入任务地点" />
        </el-form-item>
        <el-form-item label="描述" prop="extraExplain">
          <el-input v-model="form.extraExplain" placeholder="请输入描述" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { listMissionlist, getMissionlist, delMissionlist, addMissionlist, updateMissionlist } from "@/api/missionlist/missionlist";
import {listTasktype} from "@/api/tasktype/tasktype";
import {listPilots} from "@/api/pilots/pilots";
import UploadImage from "@/components/ImageUpload";
import {getPilots} from "@/api/pilots/pilots";
import {getAirline, listAirline} from "../../../api/airline/airline";
import {getMyflyrecord} from "../../../api/myflyrecord/myflyrecord";
import {getFlyrecord} from "../../../api/flyrecord/flyrecord";
import VueAliPlayer from "../../../components/VueAliPlayer/VueAliPlayer";

export default {
  name: "Missionlist",
  //注册组件
  components:{
    VueAliPlayer,
    UploadImage
  },
  data() {
    return {

      videoFlag: false,
      url: 'https://www.w3school.com.cn/example/html5/mov_bbb.mp4',

      //任务状态下拉框
      taskStateDic:[{
        "id": 0,
        "State":"未开始"
      },{
        "id": 1,
        "State":"进行中"
      },{
        "id": 2,
        "State":"已结束"
      }],

      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 任务列表表格数据
      missionlistList: [],
      //任务类型列表
      TypeList:[],
      //驾驶员列表
      PilotsList:[],
      //航线列表
      AirlineList:[],

      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        taskName: null,
        taskType: null,
        taskTime: null,
        taskDrone: null,
        taskDriver: null,
        driverPhone: null,
        taskAirline: null,
        taskImages: null,
        taskAddress: null,
        deleteCode: null,
        extraExplain: null,
        taskState: null
      },

      // 表单参数
      form: {
      },
      // 表单校验
      rules: {
        taskName: [
          { required: true, message: "任务名称不能为空", trigger: "blur" }
        ],
        taskType: [
          { required: true, message: "任务类型不能为空", trigger: "change" }
        ],
        taskTime: [
          { required: true, message: "任务时间不能为空", trigger: "blur" }
        ],
        taskDrone: [
          { required: true, message: "无人机编号不能为空", trigger: "blur" }
        ],
        taskDriver: [
          { required: true, message: "驾驶员不能为空", trigger: "blur" }
        ]
      }
    };
  },
  created() {
    this.getList();
    this.loading = false;
  },
  methods: {
    openVideo(row){
      this.title = "查看回放";
      getFlyrecord(row.id).then(response => {
        this.url=response.data.flyVideo;
        this.videoFlag = true;
      })

    },

    /** 查询任务列表列表 */
    getList() {
      listMissionlist(this.queryParams).then(response => {
        this.missionlistList = response.rows;
        this.total = response.total;
        /** 获取驾驶员信息 */
        listPilots(this.queryParams).then(response => {
          this.PilotsList = response.rows;
        });
        /** 获取任务类型信息 */
        listTasktype().then(response => {
          this.TypeList = response.rows;
        });

        /** 获取航线 */
        listAirline().then(response=>{
          this.AirlineList = response.rows
        })

        for (let i = 0; i < this.missionlistList.length; i++) {
          if (this.missionlistList[i].taskDriver){
            getPilots(this.missionlistList[i].taskDriver).then(resp=>{
              this.$set(this.missionlistList[i],'DiverName',resp.data.driverName);
              this.$set(this.missionlistList[i],'DriverPhone',resp.data.driverPhone);
            })
          }else{//ID 为0时
            this.missionlistList[i].DriverName="无";
            this.missionlistList[i].DriverPhone="无";
          }

          if(this.missionlistList[i].taskAirline){
            getAirline(this.missionlistList[i].taskAirline).then(resp => {
              this.missionlistList[i].AirLineName = resp.data.airlineName
            })
          }
          else{
            this.missionlistList[i].AirLineName = "无"
          }
        }
      });

    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 表单重置
    reset() {
      this.form = {
        taskName: null,
        taskType: null,
        taskTime: null,
        taskDrone: null,
        taskDriver: null,
        driverPhone: null,
        id: null,
        taskAirline: null,
        taskImages: null,
        taskAddress: null,
        deleteCode: null,
        extraExplain: null,
        taskState: null
      };
      this.resetForm("form");
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNum = 1;
      console.log(this.queryParams)
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.resetForm("queryForm");
      this.handleQuery();
    },
    // 多选框选中数据
    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.id)
      this.single = selection.length!==1
      this.multiple = !selection.length
    },
    /** 新增按钮操作 */
    handleAdd() {

      this.reset();
      this.open = true;
      this.title = "添加任务列表";

    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset();
      const id = row.id || this.ids
      getMissionlist(id).then(response => {
        this.form = response.data;
        this.open = true;
        this.title = "修改任务列表";
      });
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["form"].validate(valid => {
        if (valid) {
          getPilots(this.form.taskDriver).then(resp =>{
            this.form.driverPhone = resp.data.driverPhone
            if (this.form.id != null) {
              updateMissionlist(this.form).then(response => {
                this.$modal.msgSuccess("修改成功");
                this.open = false;
                this.getList();
              });
            } else {
              this.form.taskState = 0;
              addMissionlist(this.form).then(response => {
                this.$modal.msgSuccess("新增成功");
                this.open = false;
                this.getList();
              });
            }
          });
        }
      });
    },
    /** 删除按钮操作 */
    handleDelete(row) {
      const ids = row.id || this.ids;
      this.$modal.confirm('是否确认删除任务列表编号为"' + ids + '"的数据项？').then(function() {
        return delMissionlist(ids);
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("删除成功");
      }).catch(() => {});
    },
    /** 导出按钮操作 */
    handleExport() {
      this.download('missionlist/missionlist/export', {
        ...this.queryParams
      }, `missionlist_missionlist.xlsx`)
    }
  }
};
</script>
