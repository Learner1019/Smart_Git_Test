# 项目代码笔记 cnms_od_fr_update

## 日期: 2025-04-29

### 会议

#### 初步了解代码遇到的问题
- **主要讨论点**:
  - AEB在ADAS系统中是如何多模块协同工作的，数据流是什么样的
  - 短时间内的工作内容和计划
- **讨论结果**:
  - 熟悉Fct内部的三个Proc模块，重点看一下MainProc并挑一个功能模块深入理解
  - 完成环境安装并验证其正确性：
    - apl/seres/tools/builder/cmake_build.bat -> build/../log 
    - .............../gtest_coco/Gtest_build.bat
  - 学习资料目录
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\CoC\V&V\Training_Materials\Competence_Materials\01_ADAS_System\02_Automobile EE architecture
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\CoC\V&V\Training_Materials\Competence_Materials\02_Radar_System&Archtecture
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\CoC\V&V\Training_Materials\Competence_Materials\03_MPC_System&Archtecture
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\CoC\V&V\Training_Materials\Competence_Materials\08_FCT\01_Longitudinal_Function\01_PSS
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\CoC\V&V\Training_Materials\Competence_Materials\10_PMT\07_FLUX usage

---

## 日期: 2025-04-30
### 阅读代码
#### RunnableMainProc::mainProcRun
- **描述**: RunnableMainProc类的核心函数，负责执行主流程的业务逻辑。
- **代码片段**:
  ```
  MainProcInput mainProcIn{};  // 创建一个MainProcInput对象mainProcIn，用于存储主流程的输入数据
    MainProcInput包含成员：
      Preproc的输入  PreProc类型，包含VehicleInfo类（驾驶员横纵向活动、传感器安全带状态、车辆状态、HMI设置、按钮状态等）
        m_portPreProc_in
      Fct功能参数  CCal类型，其中包含ACC、AEB等功能以及通用参数->经过多层封装的加速度、车速、转向角、目标跟随场景参数（驾驶员意图匹配、碰撞概率）等需要激活功能的阈值
        m_portParamsFct_in
      主车运动状态参数（加速度、偏航率、横纵向速度、转向角等运动状态的具体数值）per
        m_portHostVehicleMovementStates_in
      主车质量、几何、传感器特性  per
        m_portHostVehicleParameters_in
      主车历史状态  per
        m_portHostVehicleHistory_in
      道路拓扑信息（数据是否更新、时间戳、路径标识、多条道路段的几何属性）per
        m_portMapRoadTopology_in (条件编译)
      目标选择策略  sit
        m_portSelectedBehaviorStrategyInformation_in (条件编译)
      行为评估相关输入 (条件编译)
        m_portBehaviorEvaluationCfmFrontTrafficCoDriverLon_in{};
        m_portBehaviorEvaluationTiplForwardCollisionAvoidance_in{};
        m_portBehaviorEvaluationTiplLaneChangeAssisting_in{};
        m_portBehaviorEvaluationTiplLaneChangeCoDriver_in{};
        m_portBehaviorEvaluationTiplLaneKeepingAssisting_in{};
        m_portBehaviorEvaluationImbaFollowPrecedingObject_in{};
        m_portBehaviorEvaluationTiplLaneKeepingCoDriverLat_in{};
        m_portBehaviorEvaluationTiplTurnCoDriver_in{};
        m_portBehaviorEvaluationTixlIntersectionCoDriverLon_in{};
      交通相关输入 (条件编译) sit
        m_portTrafficContext_in{}; //驾驶方向估计
        m_portTrafficRulesAnnotatedLanes_in{}; //车道交通规则
        m_portTiaParallelLanes_in{}; //主车道周边道路信息
        m_portRoadModel_in{}; //主车道信息
      驾驶员监控相关输入 (条件编译)  sit
        m_portDmsDriverModel_in{}; //驾驶员状态（疲劳程度、专注度等）
      其他输入 (条件编译) sit
        m_portIntrospectionTiplBehaviorIndependent_in{};
        m_portDriverBehavior_in{};
        m_portSituativeSystemUncertainty_in{};
      故障相关输入
        m_failureClustersPort_in{};
      地图属性相关输入 (条件编译) per
        m_portMapProperties_in{};
      系统时间
        m_systemTime_in{};
      功能状态
        m_portFunctionStates_in{};
  
  const vfc::CSI::si_milli_second_ui32_t systemTime = this->getSystemTime().m_time; //获取系统时间
  mainProcIn.m_systemTime_in = Dc::Ccd::InputSample<vfc::CSI::si_milli_second_ui32_t>{&systemTime, true}; //将系统时间封装为一个InputSample对象，InputSample的构造函数：explicit InputSample(const T* sample, bool hasNewData);

  mainProcPrepareInput(mainProcIn);  //将所有启用的输入端口数据提取并封装到 MainProcInput 对象mainProcIn中，确保主处理逻辑能够访问到最新的输入数据

  mainProcPrepareOutput(request, mainProcOut);  //负责根据分配请求初始化输出端口，封装到 mainProcOut，用于传递主处理逻辑的输出

  m_mainProcBusinessLogic.run(mainProcIn, mainProcOut);  //根据输入端口状态，分配输出端口的转发逻辑到controller
  
  deliverOutputPorts(request);  //传递输出端口的数据
  ```
  **问题**:
  - 客户化参数是什么
  - 行为评估是如何进行的
  - 第二个脚本安装报错，详细内容为
    - Fatal Error (Squish Coco): Fatal Error (Squish Coco): Could not write file 'CMakeFiles\apl_fct_postprocLongcomf_gtest_unit_tests.dir\hmi_calc\hmiInfo\feasibility_calc\fct_unittest_longOTAFeasibilityCalculator.cpp.obj.csmes':Error opening 'CMakeFiles\apl_fct_postprocLongcomf_gtest_unit_tests.dir\hmi_calc\hmiInfo\feasibility_calc\fct_unittest_longOTAFeasibilityCalculator.cpp.obj.csmes': No such file or directory
    apl\seres\components\fct\postproc\apl_fct_longcomf\test\unit_test\CMakeFiles\apl_fct_postprocLongcomf_gtest_unit_tests.dir\build.make:120: recipe for target 'apl/seres/components/fct/postproc/apl_fct_longcomf/test/unit_test/CMakeFiles/apl_fct_postprocLongcomf_gtest_unit_tests.dir/hmi_calc/hmiInfo/feasibility_calc/fct_unittest_longOTAFeasibilityCalculator.cpp.obj' failed
    - CMakeFiles\Makefile2:127816: recipe for target 'apl/seres/components/fct/postproc/apl_fct_longcomf/test/unit_test/CMakeFiles/apl_fct_postprocLongcomf_gtest_unit_tests.dir/all' failed
    mingw32-make.exe[2]: *** [apl/seres/components/fct/postproc/apl_fct_longcomf/test/unit_test/CMakeFiles/apl_fct_postprocLongcomf_gtest_unit_tests.dir/all] Error 2
    CMakeFiles\Makefile2:127784: recipe for target 'apl/seres/components/fct/postproc/apl_fct_longcomf/test/CMakeFiles/apl_fct_postprocLongcomf_gtest.dir/rule' failed
    mingw32-make.exe[1]: *** [apl/seres/components/fct/postproc/apl_fct_longcomf/test/CMakeFiles/apl_fct_postprocLongcomf_gtest.dir/rule] Error 2
    Makefile:35457: recipe for target 'apl_fct_postprocLongcomf_gtest' failed
    mingw32-make.exe: *** [apl_fct_postprocLongcomf_gtest] Error 2
    "app_fw_tests make_target "
    - C:\Users\YXU4WX\Project\cnms_od_fr_update\apl\seres\components\fct\fct\aeb\test\unit_test\src\fct_x_unittest_functiongroup_aeb.cpp:709: Failure
      Value of: m_mainProcOutput.getFunctionStates().m_states.m_aebTiplLongFullBrakeState
      Expected: is equal to available
        Actual: suppressed
      C:\Users\YXU4WX\Project\cnms_od_fr_update\apl\seres\components\fct\fct\aeb\test\unit_test\src\fct_x_unittest_functiongroup_aeb.cpp:720: Failure
      Value of: m_mainProcOutput.getFunctionStates().m_states.m_aebTiplLongFullBrakeState
      Expected: is equal to active
        Actual: suppressed
      [  FAILED  ] FunctionGroupAebExtUnitTest.testStateMachineTransitionsForTiplLongFullBrake (25 ms)
    - C:\Users\YXU4WX\Project\cnms_od_fr_update\apl\seres\components\fct\fct\aeb\test\unit_test\src\fct_x_unittest_functiongroup_aeb.cpp:794: Failure
      Value of: m_mainProcOutput.getFunctionStates().m_states.m_aebTiplLongPartialBrakeState
      Expected: is equal to available
        Actual: suppressed
      C:\Users\YXU4WX\Project\cnms_od_fr_update\apl\seres\components\fct\fct\aeb\test\unit_test\src\fct_x_unittest_functiongroup_aeb.cpp:805: Failure
      Value of: m_mainProcOutput.getFunctionStates().m_states.m_aebTiplLongPartialBrakeState
      Expected: is equal to active
        Actual: suppressed
      [  FAILED  ] FunctionGroupAebExtUnitTest.testStateMachineTransitionsForTiplLongPartialBrake (28 ms)
    - ...很多
---

## 日期: 2025-05-06

### 会议

#### 运行Gtest_build.bat遇到的问题
- **主要讨论点**:
  - CMake报错：
    - makeLog_apl_fct_aeb_gtest
      ```
      Error (Squish Coco): Source file 'C:\Users\YXU4WX\Project\cnms_od_fr_update\cnms_fw\components\fct\ods_fct_common\inc\fct_common\port_interfaces\processors\fct_processorSignals_objectData.hpp' is different:
      Error (Squish Coco):   Version 1: '      : m_obstacleSensorContribution{perFusionObject.m_obstacleSensorContribution}'
      Error (Squish Coco):   Version 2: '      // qacpp-2516: The perFusionObject is read from the input port. So it is located in a mempool which is'
      Error (Squish Coco): Error when importing the information from 'unit_test/CMakeFiles/apl_fct_aeb_gtest_unit_tests.dir/src/conditions/failure_based/fct_unittest_failureReactionCondition.cpp.obj'
      apl\seres\components\fct\fct\aeb\test\CMakeFiles\apl_fct_aeb_gtest.dir\build.make:246: recipe for target 'seres/fct/apl_fct_aeb_gtest.exe' failed
      mingw32-make.exe[3]: *** [seres/fct/apl_fct_aeb_gtest.exe] Error -1
      CMakeFiles\Makefile2:127504: recipe for target 'apl/seres/components/fct/fct/aeb/test/CMakeFiles/apl_fct_aeb_gtest.dir/all' failed
      mingw32-make.exe[2]: *** [apl/seres/components/fct/fct/aeb/test/CMakeFiles/apl_fct_aeb_gtest.dir/all] Error 2
      CMakeFiles\Makefile2:127511: recipe for target 'apl/seres/components/fct/fct/aeb/test/CMakeFiles/apl_fct_aeb_gtest.dir/rule' failed
      mingw32-make.exe[1]: *** [apl/seres/components/fct/fct/aeb/test/CMakeFiles/apl_fct_aeb_gtest.dir/rule] Error 2
      Makefile:35353: recipe for target 'apl_fct_aeb_gtest' failed
      mingw32-make.exe: *** [apl_fct_aeb_gtest] Error 2
      ```
    - makeLog_apl_postproc_fct_aeb_gtest
      ```
      Fatal Error (Squish Coco): Fatal Error (Squish Coco): Could not write file 'CMakeFiles\apl_fct_postprocLongcomf_gtest_unit_tests.dir\hmi_calc\hmiInfo\feasibility_calc\fct_unittest_longOTAFeasibilityCalculator.cpp.obj.csmes':Error opening 'CMakeFiles\apl_fct_postprocLongcomf_gtest_unit_tests.dir\hmi_calc\hmiInfo\feasibility_calc\fct_unittest_longOTAFeasibilityCalculator.cpp.obj.csmes': No such file or directory
      apl\seres\components\fct\postproc\apl_fct_longcomf\test\unit_test\CMakeFiles\apl_fct_postprocLongcomf_gtest_unit_tests.dir\build.make:120: recipe for target 'apl/seres/components/fct/postproc/apl_fct_longcomf/test/unit_test/CMakeFiles/apl_fct_postprocLongcomf_gtest_unit_tests.dir/hmi_calc/hmiInfo/feasibility_calc/fct_unittest_longOTAFeasibilityCalculator.cpp.obj' failed
      mingw32-make.exe[3]: *** [apl/seres/components/fct/postproc/apl_fct_longcomf/test/unit_test/CMakeFiles/apl_fct_postprocLongcomf_gtest_unit_tests.dir/hmi_calc/hmiInfo/feasibility_calc/fct_unittest_longOTAFeasibilityCalculator.cpp.obj] Error -1
      CMakeFiles\Makefile2:127816: recipe for target 'apl/seres/components/fct/postproc/apl_fct_longcomf/test/unit_test/CMakeFiles/apl_fct_postprocLongcomf_gtest_unit_tests.dir/all' failed
      mingw32-make.exe[2]: *** [apl/seres/components/fct/postproc/apl_fct_longcomf/test/unit_test/CMakeFiles/apl_fct_postprocLongcomf_gtest_unit_tests.dir/all] Error 2
      CMakeFiles\Makefile2:127784: recipe for target 'apl/seres/components/fct/postproc/apl_fct_longcomf/test/CMakeFiles/apl_fct_postprocLongcomf_gtest.dir/rule' failed
      mingw32-make.exe[1]: *** [apl/seres/components/fct/postproc/apl_fct_longcomf/test/CMakeFiles/apl_fct_postprocLongcomf_gtest.dir/rule] Error 2
      Makefile:35457: recipe for target 'apl_fct_postprocLongcomf_gtest' failed
      mingw32-make.exe: *** [apl_fct_postprocLongcomf_gtest] Error 2
      ```
- **讨论结果**
  - 报错原因: 远程库同步了二次编译的bat文件，为了加速编译注释了一些初次编译不可忽略的命令，导致报错。

### 学习内容

#### SIT行为评估流程（Behavior Consistent Environment Representation， Behaviour Strategies）

- **参考资料**
  - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\CoC\V&V\Training_Materials\Competence_Materials\07_DA_Core\02_SIT

- **知识点**
  - FCT decision有关接口
    - Necessity (某个behavior不被执行的collision probability)
    - Accident-free driving (某个behavior被执行的collision probability)
    - Driver intention match (object 和 ego-vehicle的Driving Task match程度)
      - 场景举例：超车->加速->collision probability和AEB的Necessity增加->方向盘转动->DTR识别自车为lane change，前车为lane keeping->Driver intention match不高->阻止AEB触发
      - (上述猜测待验证)
  - SIT功能结构
    - Behavior Consistent Environment Representation
      - Lane Model Fusion 车道融合模型建立
      - Object Behavior Evaluation 初步目标过滤，排除鬼影、栅栏等
      - Driver Monitoring Information 驾驶员行为
    - Behavior Strategy
      - Traffic in Parallel Lanes 三车道环境分析
      - Imitated Behavior ... 基于模仿的前车行为分析
      - Collision Free Maneuver 避撞控制
      - Traffic in Crossing Lanes 交叉车道分析
    - Behavior

---

## 日期: 2025-05-07

### 阅读代码
- **任务**
  - AEB功能组成
  - 状态机跳转关系以及判断条件
  - FCT三个runnable模块的功能及运行过程
#### FunctionContainerAeb
- **描述**: 定义了 AEB 功能的容器类。
- **功能分类**:（https://inside-docupedia.bosch.com/confluence/display/CCD/AEB+Function+namings）
    - CFM 无交通规则、无车道模型的环境
      - FunctionCfmFullBrake：前向碰撞的完全制动。
      - FunctionCfmEba：前向碰撞的紧急制动辅助。
      - FunctionCfmPrefill：前向碰撞的制动预填充。
      - FunctionCfmWarningAcoustic：前向碰撞的声学警告。
    - TIPL 行驶方向与车道平行
      - FunctionTiplLongFullBrake：纵向完全制动。
      - FunctionTiplLongPartialBrake：纵向部分制动。
      - FunctionTiplLongPrefill：纵向制动预填充。
      - FunctionTiplLongWarningAcoustic：纵向声学警告。
      - FunctionTiplLongWarningHaptic：纵向触觉警告。
      - FunctionTiplLongLatentInformation：潜在信息处理。（潜在信息是什么？）
      - FunctionTiplTurnFullBrake：转向完全制动。
      - FunctionTiplTurnPrefill：转向制动预填充。
    - TIXL 行驶方向与车道交叉（主要为90度的交叉路口）
      - FunctionTixlFullBrake：交叉路口完全制动。
      - FunctionTixlPrefill：交叉路口制动预填充。
      - FunctionTixlWarningAcoustic：交叉路口声学警告。
    <!-- - IMBA 行驶方向与潜在可能碰撞目标的运动方向平行（没做吗？） -->
    - *启用AES后的转向制动
      - FunctionTiplLongAes：纵向主动紧急转向。
      - FunctionTiplLongEss：纵向紧急转向辅助。
      - FunctionTiplLongOncomingAes：纵向对向主动紧急转向。
      - FunctionTiplLongOncomingEss：纵向对向紧急转向辅助。
    - 静止状态
      - FunctionStandstill：静止状态处理。
#### enum class AebStates : vfc::uint8_t
- **描述**: 定义了 AEB 各功能的状态机状态。
- **状态分类**:(https://inside-docupedia.bosch.com/confluence/display/ESCCN/00_AEB+StateMachine)
  - invalid = 0U  是接口的初始状态，只有首次运行前使用，随后初始化为suppressed
  - suppressed    抑制状态，不触发
  - available     可用状态，条件满足紧急制动后会转为active
  - active        活动状态，控制系统行为
  - fadeOutAlign  (不是很清楚)
  - fadeOutRamp   (不是很清楚)
  - failure = 13U 故障，无法激活
<!-- - **跳转和判断条件**:
最后研究层级：m_functionCfmFullBrake 放在FunctionContainerAeb内置数组的某个function，可以设置状态，获取仲裁结果等 -->

---

## 日期: 2025-05-08

### 阅读代码
- **任务**
  - AEB功能组成
  - 状态机跳转关系以及判断条件
  - FCT三个runnable模块的功能及运行过程

- **对FCT三个runnable模块运作的一些理解**
  - **前置流程**：
    - 传感器数据->PER目标识别与跟踪, PER会传给FCT一些参数，比如主车运动状态参数（加速度、偏航率、横纵向速度、转向角等运动状态的具体数值）、主车质量/几何/传感器特性、主车历史状态、道路拓扑信息（数据是否更新、时间戳、路径标识、多条道路段的几何属性）-> SIT对PER提供的目标信息进行筛选(OBE)，根据道路拓扑信息建立车道融合模型(LMF)，结合驾驶员行为(DMI)，判断Behavior Strategy(TIPL、CFM、IMBA、TIXL)，得到对应的Behavior Evaluation传递给FCT(对AEB来说应该是Necessity，DriverIntentionMatch，TimetoBrake, Collision Probability, Self Assessment等)
  - **PreProc**:
    - 根据已知的传感器数据(大多valin但不知道具体是什么)，计算一些语义变量，例如根据四个车轮角速度得出驾驶方向
    - 运行过程是init -> run(updateReceiverPorts, 保存输入端口的数据封装到对应的输出端口或临时变量中，deliverSenderPorts，cleanUpReceiverPorts)
  - **MainProc**:
    - 处理来自PreProc,PER,SIT的输入
    - 运行过程是init -> run(updateInputPorts，mainProcPrepareInput(mainProcIn)根据输入端口启用状态封装数据，m_mainProcBusinessLogic.getOutputAllocationRequest(mainProcIn)分配输出端口的请求参数，mainProcPrepareOutput(request, mainProcOut)根据输出端口request状态封装数据，m_mainProcBusinessLogic.run()根据输入端口状态，分配输出端口的转发逻辑到controller,deliverOutputPorts)
    - controller执行最主要最宏观的逻辑，包括processor->functiongroup->decisionmaker->behaviordistributor
  - **PostProc**:
    - 代码没找到，准备重看一下视频

---

## 日期: 2025-05-09

### 阅读代码
- **任务**
  - AEB功能组成
  - 状态机跳转关系以及判断条件
  - FCT三个runnable模块的功能及运行过程

#### 层级关系
  - MainProcController
    - Function Groups (AEB，ACC)
      - Function 每个功能都与一个状态机（`StateMachine`）相关联 -> fct_x_functionContainer_aeb.hpp
      - Condition  定义状态机的条件，用于控制状态的跳转 -> fct_x_conditionContainer_aeb.hpp
      - Processor  根据不同参数计算evaluation  ->  fct_x_functiongroup_aeb.hpp -> fct_processor_metricThresholds_aeb.cpp
      - Behavior ->  fct_behaviorRequests_aeb.hpp  


#### 状态机跳转 StateMachineAeb
**当前状态**：`suppressed` (AEB 被抑制，无法激活)
- **跳转条件**：
  1. **进入 `failure` 状态**：
     - 如果检测到故障（`m_groupResults.m_isAnyTrueForFailure`）。
  2. **进入 `available` 状态**：
     - 如果没有抑制条件（`!m_groupResults.m_isAnyTrueForSuppression`），或者客户覆盖了抑制条件（`m_groupResults.m_isAnyTrueForCustomerSuppressionOverride`）。
     - 且没有中止条件（`!m_groupResults.m_isAnyTrueForAbortion`）和对齐条件（`!m_groupResults.m_isAnyTrueForAlign`）。
  3. **保持在 `suppressed` 状态**：
     - 如果上述条件均不满足。
  
**当前状态**：`available` (AEB 可用，但未激活)
- **跳转条件**：
  1. **进入 `failure` 状态**：
     - 如果检测到故障（`m_groupResults.m_isAnyTrueForFailure`）。
  2. **进入 `suppressed` 状态**：
     - 如果满足抑制条件（`m_groupResults.m_isAnyTrueForSuppression`），且客户未覆盖抑制条件（`!m_groupResults.m_isAnyTrueForCustomerSuppressionOverride`）。
     - 或者满足中止条件（`m_groupResults.m_isAnyTrueForAbortion`）或对齐条件（`m_groupResults.m_isAnyTrueForAlign`）。
  3. **进入 `active` 状态**：
     - 如果满足激活条件（`m_groupResults.m_areAllTrueForActivation`）。
  4. **保持在 `available` 状态**：
     - 如果上述条件均不满足。

**当前状态**：`active`  (AEB 已激活，正在执行紧急制动)
- **跳转条件**：
  1. **进入 `failure` 状态**：
     - 如果检测到故障（`m_groupResults.m_isAnyTrueForFailure`）。
  2. **进入 `fadeOutRamp` 状态**：
     - 如果计时器到期（`minimumActivationTimerElapsed`），且满足中止条件（`m_groupResults.m_isAnyTrueForAbortion`），但未满足渐变结束条件（`!m_groupResults.m_isAnyTrueForRampEnd`）。
  3. **进入 `fadeOutAlign` 状态**：
     - 如果计时器到期（`minimumActivationTimerElapsed`），且满足对齐条件（`m_groupResults.m_isAnyTrueForAlign`）。
  4. **进入 `available` 状态**：
     - 如果计时器到期（`minimumActivationTimerElapsed`），且不满足激活条件（`!m_groupResults.m_areAllTrueForActivation`）。
  5. **保持在 `active` 状态**：
     - 如果上述条件均不满足。

**当前状态**：`failure`  (AEB 进入故障状态)
- **跳转条件**：
  1. **保持在 `failure` 状态**：
     - 如果检测到故障（`m_groupResults.m_isAnyTrueForFailure`）。
  2. **进入 `suppressed` 状态**：
     - 如果未检测到故障。

#### 问题：
- Condition在PPT中为boolean值，但是代码中分activation, abortion, suppression, failure等
- ConditionGroup的概念没理解 fct_conditionGroup_aeb.hpp
- fadeOutRamp和fadeOutOut的作用不清楚，猜测可能是为了用户体验或者场景需求逐步减少数值(例如缓慢踩刹车？)
- PostProc在视频和ppt中介绍很少，应该是hmi？
- valin是什么

#### 解答：
- activation, abortion, suppression, failure等为conditionGroup的枚举值，具体到condition还是boolean值
- 每个conditionGroup相关的condition范围不同，判断状态机state时会check每个conditionGroup，当某个group的所有condition都满足时就会发生跳转
- 和aeb关系不大，是其他功能需要跳转的state
- fct_stateMachineOutput_aeb.cpp
- 不同客户的接口与项目功能内部接口的mapping(不一定是一对一mapping)


---

## 日期: 2025-05-12

### 阅读代码 二段刹三个时间点的计算
  - **WorkFlow**
    - `LaneKeepingCoDriverLonBehaviorEvaluator::evaluate`->
      - const BehaviorSpecification&                                                        behaviorSpecification
      - const TiplForwardCollisionAvoidanceEvaluatorParameters&                             evaluationParameter
      - const RunnableInterfaceTiplForwardCollisionAvoidance<BehaviorEvaluationConfigType>& runnableInterface
      - const TiplForwardCollisionAvoidanceEvaluatorInputData&                              evaluatorInputData
      - TiplForwardCollisionAvoidanceEvaluatorOutputData<BehaviorEvaluationConfigType>&     evaluatorOutputData
    - `runRpmEgoPrediction`->
      - const BehaviorSpecification&                            behaviorSpecification
      - const TiplForwardCollisionAvoidanceEvaluatorParameters& evaluationParameter
      - const ObjectsPointerArray&                              relevantObjects
      - const Utilities::optional<vfc::uint8_t>                 selectedRelevantObjectIndex
      - const EnvironmentDataTipl&                              environmentDataTipl
      - const RiRpmServiceStateRx&                              riRpmServiceStateRx
      - const vfc::CSI::si_metre_f32_t                          longitudinalSafetyDistance
      - bool                                                    selectedRelevantObjectIsOncoming
      - BehaviorEvaluation&                                     behaviorEvaluation
      - BehaviorPrivateData&                                    behaviorPrivateData
      - const OptionalRef<BehaviorOptionalPrivateData>          behaviorOptionalPrivateData
      - DebugVariablesTiplForwardCollisionAvoidanceEvaluator&   debugVariables
    - `runPrediction`->
      - const Sit::EnvironmentModel<MaxLanesValue, MaxLaneSegmentsValue>& environmentModel
      - const Sit::EgoData&                                               egoData
      - const ReactionPatternRequest&                                     reactionPatternRequest
      - const Second                                                      tPredictionHorizon
      - const ServiceState&                                               rpmServiceState
      - ReactionPattern&                                                  reactionPattern
      - OptionalRef<Introspection>                                        introspection
    - `EmergencyAvoidanceCalculator::calculate`->
      - const Second                                tStart
      - const Second                                tFinal
      - const Dc::Norms::NormalizedSeconds65536_u32 absoluteReferenceTimestamp
      - const IVirtualLanesView&                    virtualLanes
      - const PredictionObjects&                    objects
      - const EgoDataView&                          hostVehicleMovementState
      - const ReactionPatternRequest&               reactionPatternRequest
      - const ServiceState&                         serviceState
      - ReactionPattern&                            reactionPattern
    - `EmergencyBrakeCalculator::calculateEmergencyBrake`->
      - const ReactionPatternRequest&               reactionPatternRequest
      - const EgoDataView&                          egoDataView
      - const ServiceState&                         serviceState
      - const TargetObject&                         targetObject
      - const IVirtualLanesView&                    lanes
      - const Dc::Norms::NormalizedSeconds65536_u32 absoluteReferenceTimestamp
      - const Second                                tStart
      - const Second                                tFinal
      - ReactionPattern&                            reactionPattern
    - `calculateWithActiveEqualOrHigherBraking`/`calculateEmergencyBrakeKinematics`->
      - const Rpm::EmergencyBrakeParameters& emergencyBrakeParameters
      - const ObjectTrajectory&              trajectoryObject
      - const CurrentTrajectory&             trajectoryEgoCurrent
    - `calculateBrakeTimeEvents`->
      - const BrakeCalculator&          calculator
      - const EmergencyBrakeParameters& emergencyBrakeParameters
      - const CurrentTrajectory&        trajectoryEgoCurrent
      - const ObjectTrajectory&         trajectoryObject
    - `calculateTimeEventsForOneInterval`->
      - const EmergencyBrakeParameters&  emergencyBrakeParameters
      - const LongitudinalMovementState& egoFrontBumperState
      - const LongitudinalMovementState& objState
      - const Second                     tEgo2
      - const Second                     tObj2
    - `calculateAccelerationEventsForTwoStage`
      - const EmergencyBrakeParameters&  emergencyBrakeParameter
      - const LongitudinalMovementState& egoFrontBumperState
      - const LongitudinalMovementState& objState
      - **return** accelerationEvents.result2_1/result2_2/result2_3

---

## 日期: 2025-05-13

### UnitTest 工作
  - **Code path**: \\bosch.com\dfsrb\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\Function-DA\Comfort_Functions\Training_Materials\CR\pros\pl-rearcorner.zip
  - **Branch**: https://inside-docupedia.bosch.com/confluence/pages/viewpage.action?pageId=5646304099
  - **Mono**: release/xpeng/CRGVXPFIII-1360-develop-aries4.1-cp23
  - **Come bas**e: release/PJRCBASE-9042-develop-aries4.1-cp23
  - **Apl base**: develop-xpeng-cr5cb
  - **Build script**:  \apl\base\bindings\xpeng\buildscripts\testbuild_BaseC0SS_SINGLE.bat
  - **Files**:
    - golf_fct_brakeHmi.cpp
    - golf_fct_opticHmi.cpp
    - golf_fct_statesHmi.cpp
    - golf_fct_outputEdrData.cpp
    - golf_fct_spp.cpp
  - **进度**:
    - 环境编译

### pj-dc_stakeholder中二段刹功能函数matlab实现
  - **任务**:
    - rpm_calculateTimeEventsOfImmediateAsilVelocityReduction.m
  - **进度**:
    - matlab: rpm_calculateTimeEventsOfImmediateAsilVelocityReduction line 101
    - cpp: rpm_emergencyBrakeImmediateVelReducLimit line 171
  
---

## 日期: 2025-05-14

### UnitTest 工作
  - **Code path**: \\bosch.com\dfsrb\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\Function-DA\Comfort_Functions\Training_Materials\CR\pros\pl-rearcorner.zip
    - **Branch**: https://inside-docupedia.bosch.com/confluence/pages/viewpage.action?pageId=5646304099
    - **Mono**: release/xpeng/CRGVXPFIII-1360-develop-aries4.1-cp23
    - **Come bas**e: release/PJRCBASE-9042-develop-aries4.1-cp23
    - **Apl base**: develop-xpeng-cr5cb
    - **Build script**:  \apl\base\bindings\xpeng\buildscripts\testbuild_BaseC0SS_SINGLE.bat
    - **Files**:
      - golf_fct_brakeHmi.cpp
      - golf_fct_opticHmi.cpp
      - golf_fct_statesHmi.cpp
      - golf_fct_outputEdrData.cpp
      - golf_fct_spp.cpp
    - **进度**:
      - 学习UnitTest的创建测试工程时报错的解决方法
      - 解决golf_fct_opticHmi.cpp的测试errors
      - 编写测试用例

---

## 日期: 2025-05-15

### UnitTest 工作
  - **Code path**: \\bosch.com\dfsrb\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\Function-DA\Safety_Functions\Learning_Groups\Li Jiakuo\CA_G393_UT\CA_CN_TSR.zip
    - **Branch**:
      - **main**: release/CNFVE0100
      - **UT**: feature/MPCTECAOV-1734-g393-ut-tsr
    - **编译指令**：cpj_ca1vEvo\tools\build_g393evo_ec2_dev.bat  执行 这个脚本 选择all
    - **UT编译指令**：
      - ./fvg3.bat less_llvm -p cpj_ca1vEvo -v g393evo   -c Release -j 8 -cov on -m configure
      - cmake --build .\build\g393evo\less_llvm_cov\  --target COVERAGE_ov_tsa_test
    - **进度**:
      - 脚本编译
      - 申请license

  - **Code path**: \\bosch.com\dfsrb\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\Function-DA\Comfort_Functions\Training_Materials\CR\pros\pl-rearcorner.zip
    - **进度**:
      - 完成golf_fct_opticHmi.cpp的UT task
  
---

## 日期: 2025-05-16

### UnitTest 工作
  - **Project**: CA_CN_TSR
    - **Branch**:
      - **main**: release/CNFVE0100
      - **UT**: feature/MPCTECAOV-1734-g393-ut-tsr
    - **编译指令**：cpj_ca1vEvo\tools\build_g393evo_ec2_dev.bat  执行 这个脚本 选择all
    - **UT编译指令**：
      - ./fvg3.bat less_llvm -p cpj_ca1vEvo -v g393evo   -c Release -j 8 -cov on -m configure
      - cmake --build .\build\g393evo\less_llvm_cov\  --target COVERAGE_ov_tsa_test
    - **进度**:
      - 环境编译

  - **Project**: pl-rearcorner
    - **进度**:
      - 完成golf_fct_brakeHmi.cpp的UT task
      - 

---

## 日期: 2025-05-19

### UnitTest 工作
  - **Project**: CA_CN_TSR
    - **资料学习**:
      - \\bosch.com\dfsrb\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\CoC\ADAS_ASW\Training_Materials\Unit_test\Google_test
      - https://inside-docupedia.bosch.com/confluence/display/AdasCoC/Googletest
    - **进度**:
      - 完成sit_holding.cpp的UT task
### NM、E2E数据标注工作（紧急）
  - 下午培训NM、E2E任务前置知识
  

---

## 日期: 2025-05-21

### NM、E2E数据标注工作（紧急）
  - **文件路径**:\\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops
  - **进度管理**:https://bosch-my.sharepoint.com/:x:/r/personal/awm1wx_bosch_com/_layouts/15/Doc.aspx?sourcedoc=%7B4a611ffa-1037-4630-84fb-82a645d772f1%7D&action=edit&wdenableroaming=1&wdodb=1&wdlcid=en-US&wdorigin=AuthPrompt.Sharing.DirectLink.Copy&wdredirectionreason=Force_SingleStepBoot&wdinitialsession=b74cfad4-cada-9f2b-53bb-f73b54a34587&wdrldsc=2&wdrldc=1&wdrldr=InvalidExternalHyperlinkFailure
  - **培训视频**:
    - RE:\\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI RE Trainning
    - DEV:\\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI DEV Trainning 
  - **进度**:
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI_RE\标注数据_20250328\Chery_D01\FR 已完成
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI_RE\标注数据_20250328\DFL
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI_RE\标注数据_20250328\DFSK
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI_RE\标注数据_20250328\BYD

---

## 日期: 2025-05-22

### NM、E2E数据标注工作（紧急）
  - **文件路径**:\\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops
  - **进度管理**:https://bosch-my.sharepoint.com/:x:/r/personal/awm1wx_bosch_com/_layouts/15/Doc.aspx?sourcedoc=%7B4a611ffa-1037-4630-84fb-82a645d772f1%7D&action=edit&wdenableroaming=1&wdodb=1&wdlcid=en-US&wdorigin=AuthPrompt.Sharing.DirectLink.Copy&wdredirectionreason=Force_SingleStepBoot&wdinitialsession=b74cfad4-cada-9f2b-53bb-f73b54a34587&wdrldsc=2&wdrldc=1&wdrldr=InvalidExternalHyperlinkFailure
  - **培训视频**:
    - RE:\\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI RE Trainning
    - DEV:\\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI DEV Trainning 
  - **进度**:
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI_RE\标注数据_20250328\Chery_D01\FR 已完成
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI_RE\标注数据_20250328\DFL  已完成
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI_RE\标注数据_20250328\DFSK 已完成
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI_RE\标注数据_20250328\BYD  已完成

---

## 日期: 2025-05-23

### NM、E2E数据标注工作（紧急）
  - **文件路径**:\\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops
  - **进度管理**:https://bosch-my.sharepoint.com/:x:/r/personal/awm1wx_bosch_com/_layouts/15/Doc.aspx?sourcedoc=%7B4a611ffa-1037-4630-84fb-82a645d772f1%7D&action=edit&wdenableroaming=1&wdodb=1&wdlcid=en-US&wdorigin=AuthPrompt.Sharing.DirectLink.Copy&wdredirectionreason=Force_SingleStepBoot&wdinitialsession=b74cfad4-cada-9f2b-53bb-f73b54a34587&wdrldsc=2&wdrldc=1&wdrldr=InvalidExternalHyperlinkFailure
  - **培训视频**:
    - RE:\\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI RE Trainning
    - DEV:\\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI DEV Trainning 
  - **进度**:
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI_RE\标注数据_20250328\Chery_D01\FR 已完成
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI_RE\标注数据_20250328\DFL  已完成
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI_RE\标注数据_20250328\DFSK 已完成
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI_RE\标注数据_20250328\BYD  已完成
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Project\Devops\AI_RE\标注数据_20250328\CA\C318MPCEvo  已完成
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Dept\BCSC\01_Common\Devops\AI_RE\标注数据  20250328\BAIC C52X FR
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Dept\BCSC\01_Common\Devops\AI_RE\标注数据  20250328\FAWHQ E009\FR5CP
    - \\bosch.com\dfsRB\DfsCN\loc\Wx8\Dept\BCSC\01_Common\Devops\AI_RE\标注数据  20250328\SAIC_IM_S12LOversea\CR5CB

---

## 日期: 2025-05-26

### UnitTest 工作
  - **Project**: CA_CN_TSR
    - **资料学习**:
      - \\bosch.com\dfsrb\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\CoC\ADAS_ASW\Training_Materials\Unit_test\Google_test
      - https://inside-docupedia.bosch.com/confluence/display/AdasCoC/Googletest
    - **进度**:
      - sit_holding.cpp   Done
      - sit_val.cpp       Done
      - ov_rsf_main.cpp   Ongoing
      - sla_suppression.cpp

---

## 日期: 2025-05-27

### UnitTest 工作
  - **Project**: CA_CN_TSR
    - **资料学习**:
      - \\bosch.com\dfsrb\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\CoC\ADAS_ASW\Training_Materials\Unit_test\Google_test
      - https://inside-docupedia.bosch.com/confluence/display/AdasCoC/Googletest
    - **进度**:
      - sit_holding.cpp   Done
      - sit_val.cpp       Done
      - ov_rsf_main.cpp   Done
        - **问题**
          - line 116 分支覆盖不到
          - line 266 m_OverSpdWarnEnable_b无法设置为false
      - sla_suppression.cpp Ongoing

---

## 日期: 2025-05-28

### UnitTest 工作
  - **Project**: CA_CN_TSR
    - **资料学习**:
      - \\bosch.com\dfsrb\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\CoC\ADAS_ASW\Training_Materials\Unit_test\Google_test
      - https://inside-docupedia.bosch.com/confluence/display/AdasCoC/Googletest
    - **进度**:
      - sit_holding.cpp   Done
      - sit_val.cpp       Done
      - ov_rsf_main.cpp   Done
        - **问题**
          - line 116 分支覆盖不到
          - line 266 m_OverSpdWarnEnable_b无法设置为false
      - sla_suppression.cpp Ongoing

---

## 日期: 2025-05-29

### UnitTest 工作
  - **Project**: CA_CN_TSR
    - **资料学习**:
      - \\bosch.com\dfsrb\DfsCN\loc\Wx8\Dept\BCSC\01_Common\04_Competency\CoC\ADAS_ASW\Training_Materials\Unit_test\Google_test
      - https://inside-docupedia.bosch.com/confluence/display/AdasCoC/Googletest
    - **进度**:
      - sit_holding.cpp   Done
      - sit_val.cpp       Done
      - ov_rsf_main.cpp   Done
        - **问题**
          - line 116 死条件 状态分支覆盖不到
          - line 266 死条件 m_OverSpdWarnEnable_b->true
      - sla_suppression.cpp Done
      - - **问题**
          - line 570  死条件 line67:fSupprCASpecicalCondition 使得 满足if语句的fSupprNonExpired都被跳过
          - line 611  死条件 fIsTurnFiltPossible->true使得l_sumDegChange_f32 > 50
          - line 1354 死条件 fIsPlausible()->false
          - line 1428 死条件
          - line 1654 死条件 fHaveTrailer()->false
          - line 1760 死条件 fIsNight()->false
          - line 2234-2258 导航函数不要测

---

## 日期: 2025-05-30

### 脚本代码适配
  - **Project**: WIN 2.0
    - **进度**:
      - 可适配WIN 2.0仿真结果文件比对
      - 打包main.exe

### UnitTest 工作
  - **Project**: pl-rearcorner
    - **进度**:
      - golf_fct_brakeHmi.cpp       Done
      - golf_fct_opticHmi.cpp       Done
      - golf_fct_statesHmi.cpp      Ongoing
      - golf_fct_outputEdrData.cpp  
      - golf_fct_spp.cpp
---

## 日期: 2025-06-04

### UnitTest 工作
  - **Project**: pl-rearcorner
    - **进度**:
      - golf_fct_brakeHmi.cpp       Done
      - golf_fct_opticHmi.cpp       Done
      - golf_fct_statesHmi.cpp      Done
      - golf_fct_outputEdrData.cpp  Ongoing
      - golf_fct_spp.cpp            Ongoing
---

## 日期: 2025-06-05

### UnitTest 工作
  - **Project**: pl-rearcorner
    - **进度**:
      - golf_fct_brakeHmi.cpp       Done
      - golf_fct_opticHmi.cpp       Done
      - golf_fct_statesHmi.cpp      Done
      - golf_fct_outputEdrData.cpp  Done
      - golf_fct_spp.cpp            Done
  
  - **Project**: CA FVE0100
    - **Branch**:
      - fvg3.bat less_llvm -m full -p cpj_ca1vEvo -v g393evo -c Release -cov on --component apl_fct_aeb  -j 8
      - fvg3.bat less_llvm -m full -p cpj_ca1vEvo -v g393evo -c Release -cov on --component ov_postproc_commonRunnable -j 8
      - fvg3.bat less_llvm -m full -p cpj_ca1vEvo -v g393evo -c Release -cov on --component  postproc_aeb -j 8
    - **VS工程编译**:
      - fvg3.bat less_v140 -m configure -p cpj_ca1vEvo -v  g393evo -c Debug
    - **Branch**:
      - feature/MPCTECAOV-1844-ca1vevo-od_fct-less-setup-environment-for-unit-test_x(还要替换4个文件)。
---

## 日期: 2025-06-06

### UnitTest 工作
  - **Project**: CA FVE0100
    - **Branch**:
      - fvg3.bat less_llvm -m full -p cpj_ca1vEvo -v g393evo -c Release -cov on --component apl_fct_aeb  -j 8
      - fvg3.bat less_llvm -m full -p cpj_ca1vEvo -v g393evo -c Release -cov on --component ov_postproc_commonRunnable -j 8
      - fvg3.bat less_llvm -m full -p cpj_ca1vEvo -v g393evo -c Release -cov on --component  postproc_aeb -j 8
    - **VS工程编译**:
      - fvg3.bat less_v140 -m configure -p cpj_ca1vEvo -v  g393evo -c Debug
    - **Branch**:
      - feature/MPCTECAOV-1844-ca1vevo-od_fct-less-setup-environment-for-unit-test_x(还要替换4个文件)。

### pj-dc_stakeholder项目matlab二段刹功能
  - **问题**:
    - calculateTimeDurationForVelocityChangeDueToAccelerationForTwoStageVelReducLimit代码理解（vEndVelReducLimit）
---

## 日期: 2025-06-09

### pj-dc_stakeholder项目matlab二段刹功能
  - **问题**:
    - calculateTimeDurationForVelocityChangeDueToAccelerationForTwoStageVelReducLimit代码理解（vEndVelReducLimit）
  - **进度**:
    - rpm_calculateTimeEventsOfImmediateAsilVelocityReduction 完成
    - review 代码链路
      - rpm_calculateTimeEventsOfImmediateEmergencyBrake.m (line 72, 147)
---

## 日期: 2025-06-10

### pj-dc_stakeholder项目matlab二段刹功能
  - **Review讨论**:
    - 需要重新check函数入口，存在遗漏

### WIN2.0 Labeling工具适配
  - **适配需求**
    - 优化视频下载和处理速度
  - **学习代码**
  - **优化思路**
    - 重构video_links，dict实现视频路径和function触发区间的映射，并保证视频唯一性
      - video_links {"video_link_path": video_link, ...}
        - video_link {"tipllongfb": [time_points, ...], ...}
          - time_points [FuncActivation_time, FuncActivation_time + FuncActivation_Duration]
    - 记录video_links进入dict的先后顺序，按照顺序对视频切分并生成gif，保证labeling效率
---

## 日期: 2025-06-12

### WIN2.0 Labeling工具适配
  - **适配需求**
    - 优化视频下载和处理速度
  - **学习代码**
  - **优化思路**
    - 重构video_links，dict实现视频路径和function触发区间的映射，并保证视频唯一性
      - video_links {"video_link_path": video_link, ...}
        - video_link {"tipllongfb": [time_points, ...], ...}
          - time_points [FuncActivation_time, FuncActivation_time + FuncActivation_Duration]
    - 记录video_links进入dict的先后顺序，按照原有视频处理逻辑，按照.\Mytool\{output_path}\{video_name}\{function_name}\{activation_time}的路径格式存放切分好的image
    - 按照以下优先级处理：用户即将查询的下一条记录->在xlsx中最先出现的视频，同时保证处理效率和使用体验，在处理完一个视频的所有功能后删除
    - 静态存储处理后的视频所属image链接，下次打开相同xlsx文件时，不需要重复下载已经处理好的视频
    - 修复UI显示Bug：视频路径过长导致窗口超出屏幕范围

### UnitTest
- **Project**: WIN 2.0-pj_w3_mcu_chery
    - **Branch**: develop_OrinY
    - **环境编译**
      - 1.运行_init_TCC_IF_Tools.bat
      - 2.运行_init_WIN_TCC_ENV.bat
      - 3.打开build_OrianY.bat,将文件内容替换为：
          call apl_w3/tools/builder/generate_scom.bat -hw W3_E0X_EEA4_X_MASTER
          call cmake_build.bat -b W3_E0X_EEA4_X_MASTER -s safety -scom
      - 4.运行build_OrianY.bat

---

## 日期: 2025-06-17

### UnitTest
- **Project**: WIN 2.0-pj_w3_mcu_chery
    - **Branch**: feature/WINGII-6156-mcu-apl-per-gt-orinY
    - **进度 & Comment**
      - apl_w3\component\fal\per_x\per\modules\per_x_lib\src\per_x_lib_VideoObj.inl   81.915% (154/188)
      - apl_w3\component\fal\per_x\per\modules\per_x_lib\src\per_x_lib_VideoLine.inl  91.667% (110/120)
        - line 446 checkClothoidDataIsValid 无法覆盖
        - line 474 checkPolynomialDataIsValid()  调用代码被注释，无法覆盖
        - line 761 default分支无法覆盖，死代码

---

## 日期: 2025-06-18

### UnitTest
- **Project**: WIN 2.0-pj_w3_mcu_chery
    - **Branch**: feature/WINGII-6156-mcu-apl-per-gt-orinY
    - **进度 & Comment**
      - apl_w3\component\fal\per_x\per\modules\per_x_lib\src\per_x_lib_VideoObj.inl   82.447% (155/188)
        - line 153,156,158 default分支无法覆盖，死代码
        - line 308 getReferencePointMPC3() 分支无法覆盖，死代码
        - line 765 updateRfrncPntEdgeAglDxDyInvTTCSignal() 分支无法覆盖，死代码
      - apl_w3\component\fal\per_x\per\modules\per_x_lib\src\per_x_lib_VideoLine.inl  93.333% (112/120)
        - line 446 checkClothoidDataIsValid 无法覆盖
        - line 474 checkPolynomialDataIsValid()  调用代码被注释，无法覆盖
        - line 761 default分支无法覆盖，死代码
        - line 861,870 else分支无法覆盖，死代码
      - 
    - **新runnable适配**
      - 打开flux->show instance->code里搜索对应端口名定义->复制到mock类成员变量定义中
### pj-dc_stakeholder项目matlab二段刹功能
  - **Review**:
    - rpm_calculateEmergencyBrakeKinematics.cpp line 161 -> constructBrakeTrajectory

---

## 日期: 2025-06-19

### UnitTest
- **Project**: WIN 2.0-pj_w3_mcu_chery
    - **Branch**: feature/WINGII-6156-mcu-apl-per-gt-orinY
    - **进度 & Comment**
      - FCT 链路打通 && 调试加入用例后的bug
      - SIT sit_x_sppHmiInput_user.cpp 覆盖率effort 41% -> 66%

---

## 日期: 2025-06-20

### UnitTest
- **Project**: WIN 2.0-pj_w3_mcu_chery
    - **Branch**: feature/WINGII-6156-mcu-apl-per-gt-orinY
    - **进度 & Comment**
      - FCT preproc_runnable.cpp
      - FCT preproc_runnable_user.cpp
        - line 79 preproc_CalcRoadWheelAngleFront 代码被注释，导致if恒为False
        - line 88 if恒为False的连锁反应
      - SIT sit_x_sppHmiInput_user.cpp 覆盖率effort 66% -> 94%
        - line 163 dmsStatus恒等于active, else无法覆盖

---

## 日期: 2025-06-23

### UnitTest
- **Project**: WIN 2.0-pj_w3_mcu_chery
    - **Branch**: feature/WINGII-6156-mcu-apl-per-gt-orinY
    - **进度 & Comment**
      - apl_w3\component\fal\per_x\per\modules\per_x_lib\src\per_x_lib_VideoObj.inl   82.447% (155/188)
        - line 153,156,158 default分支无法覆盖，死代码
        - line 308 getReferencePointMPC3() 分支无法覆盖，死代码
        - line 765 updateRfrncPntEdgeAglDxDyInvTTCSignal() 分支无法覆盖，死代码
        - line 861,870 else分支无法覆盖，死代码
      - apl_w3\component\fal\per_x\per\modules\per_x_lib\src\per_x_lib_VideoLine.inl  93.333% (112/120)
        - line 474 checkPolynomialDataIsValid()  调用代码被注释，无法覆盖
        - line 761 default分支无法覆盖，死代码
        - line 446 怎么赋值nullptr
      - FCT preproc_runnable.cpp
      - FCT preproc_runnable_user.cpp
        - line 79 preproc_CalcRoadWheelAngleFront 代码被注释，导致if恒为False
        - line 88 if恒为False的连锁反应
      - SIT sit_x_sppHmiInput_user.cpp 覆盖率effort 94%
        - line 163 dmsStatus恒等于active, else无法覆盖

---

## 日期: 2025-06-24

### UnitTest
- **Project**: WIN 2.0-pj_w3_mcu_chery
    - **Branch**: feature/WINGII-6156-mcu-apl-per-gt-orinY
    - **进度 & Comment**
      - per_x_lib_VideoObj.inl   82.447% (155/188)
        - line 153,156,158 default分支无法覆盖，死代码
        - line 308 getReferencePointMPC3() 分支无法覆盖，死代码
        - line 765 updateRfrncPntEdgeAglDxDyInvTTCSignal() 分支无法覆盖，死代码
        - line 861,870 else分支无法覆盖，死代码
      - per_x_lib_VideoLine.inl  93.333% (112/120)
        - line 446 怎么赋值nullptr
        - line 474 checkPolynomialDataIsValid()  调用代码被注释，无法覆盖
        - line 761 default分支无法覆盖，死代码
      - FCT preproc_runnable.cpp          100.000% (11/11)
      - FCT preproc_runnable_user.cpp     96.813% (243/251)
        - line 79 preproc_CalcRoadWheelAngleFront 代码被注释，导致if恒为False
        - line 88 if恒为False的连锁反应
      - SIT sit_x_sppHmiInput.cpp         100.000% (3/3)
      - SIT sit_x_sppHmiInput_user.cpp    94.444% (34/36)
        - line 163 dmsStatus恒等于active, else无法覆盖

---

## 日期: 2025-06-25

### UnitTest
- **Project**: WIN 2.0-pj_w3_mcu_chery
    - **Branch**: feature/WINGII-6156-mcu-apl-gt-fw-develop-oriny-0625
    - **编译链路脚本路径适配**
      - per
        - add
          - "${RootPath}/apl_w3/fsi/rbdynet/cfg/"
  	      - "${RootPath}/apl_w3/fsi/rbdynet/common/inc"
        	- "${RootPath}/apl_w3/fsi/rbdynet/PubNet/CAN_Classic/eea5_1/Sins_Gen"
        	- "${RootPath}/apl_w3/fsi/rbdynet/PubNet/CAN_DA/eea5_1/Sins_Gen"
        	- "${RootPath}/apl_w3/fsi/rbdynet/PubNet/CAN_RE/eea5_1/Sins_Gen"
        	- "${RootPath}/apl_w3/fsi/rbdynet/PrvNet/RadarFront/Sins_Gen"
      	- replace
        	- /apl_w3/cfg								-> /apl_w3/asw/application/cfg
          - ${RootPath}/apl_w3/config/cubas/_out/e0x_eea5_1/gen6 			-> ${RootPath}/apl_w3/config/cubas/gen6/e0x_eea5_1
          - ${RootPath}/apl_w3/fsi/Cfg_Prj/rtaos/A18Y/out/ 				-> ${RootPath}/apl_w3/config/rtaos/A18Y/out/
          - ${RootPath}/apl_w3/config/cubas/pjif_dasy_cfg/jlr_my20_common		-> ${RootPath}/apl_w3/config/cubas/jlr_my20_common
          - ${RootPath}/apl_w3/fsi/Cfg_Prj/rtaos/Common/inc				-> ${RootPath}/apl_w3/config/rtaos/Common/inc
          - apl_w3/dep 								-> apl_w3/asw/application/dep
          - apl_w3/main								-> apl_w3/asw/application/main
          - Cfg_Prj/net/GW/Daddy_GW/PrvNet/RFC_FR5CP/inc				-> rbdynet/PrvNet/RadarFront/Block_Callout/inc
          - Cfg_Prj/net/GW/Daddy_GW/PrvNet/RFC_FR5CP/RFC_Status			-> rbdynet/PrvNet/RadarFront/Sins_Gen
          - Cfg_Prj/net/GW/Daddy_GW/PrvNet/common/inc				-> rbdynet/PrvNet/common/inc
          - Cfg_Prj/net/ETH_INC							-> rbdynet/rbdyNetGW/ETH_INC
          - apl_w3/pub_inc/								-> apl_w3/config/ipc_inf/pub_inc/
          - ${RootPath}/apl_w3/fsi/Component/custDiag/evmHandler/aswInteraction	-> ${RootPath}/apl_w3/fsi/custDiag/evmHandler/aswInteraction

---

## 日期: 2025-06-26 - 2025-06-27

### UnitTest
- **Project**: WIN 2.0-pj_w3_mcu_chery
    - **Branch**: feature/WINGII-6156-mcu-apl-gt-fw-develop-oriny-0625
    - **编译链路脚本路径适配**
      - 发现oriny PER代码bug导致compareByTypeAndDx覆盖率为0的问题，并经过compare同步了最新代码
      - per_x_sppEnvRunnable.cpp      100.000% (8/8)
      - per_x_sensorRadarFCLoc.cpp    100.000% (13/13)
      - per_x_sensorVideoFC.cpp       67.647% (23/34)
      - per_x_lib_General.inl         32.558% (14/43)

---

## 日期: 2025-06-30

### UnitTest
- **Project**: WIN 2.0-pj_w3_mcu_chery
    - **Branch**: feature/WINGII-6156-mcu-apl-gt-fw-develop-oriny-0625
    - **编译链路脚本路径适配**
      - per_x_sppEnvRunnable.cpp      100.000% (8/8)
      - per_x_sensorRadarFCLoc.cpp    100.000% (13/13)
      - per_x_sensorVideoFC.cpp       100.000% (29/29)
      - per_x_lib_RadarLoc.inl        100.000% (21/21)
      - per_x_lib_General.inl         32.558% (14/43)

---

## 日期: 2025-07-01

### UnitTest
- **Project**: WIN 2.0-pj_w3_mcu_chery
    - **Branch**: feature/WINGII-6156-mcu-apl-gt-fw-develop-oriny-0625
    - **Code Review Response**
      - UT报告低覆盖率文件过滤
      - 测试代码Comment编写
---

<!-- ## 学习资料笔记
记录学习过程中重要的知识点、参考资料和心得体会。

### 日期: YYYY-MM-DD
#### 学习主题
- **知识点**:
  - 知识点 1
  - 知识点 2
- **参考资料**:
  - [资料名称](链接)
- **心得体会**:
  - 简要总结学习收获。

---

## 其他备注
记录其他需要注意的事项或补充内容。 -->