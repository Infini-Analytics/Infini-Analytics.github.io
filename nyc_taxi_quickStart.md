# Infini Analytics Quickstart

#### 登录
![登录](./assets/Login.png)

#### 配置数据库
输入合理的数据库配置信息，包括：服务器地址，用户名，密码，数据库名，端口，
也可点击右下角默认参数一键填充。
![数据库配置](./assets/MegaConfig.png)
配置成功后自动跳转到仪表盘列表界面。
![数据库配置成功](./assets/MegaSuccessConfig.png)
![新的名称](./assets/DashBoardList.png)

#### 新建仪表盘
点击‘新增仪表盘’，创建新的仪表盘
![默认名称](./assets/NewDashBoard.png)
点击默认名称进行修改，名称会自动保存
![新的名称](./assets/DashBoardNameChange.png)

#### 添加数图
选择要展示的数据表。鼠标悬浮在选项上即可查看相关数据表信息，包括整体数据量，字段数，字段名称，字段类型等。
![查看数据表信息](./assets/ViewColumnLists.png)
选择要查看的字段名称。
![选择计量](./assets/NumberChartSelectMeasure.png)
选择要展示的计量类型，当前支持平均值、最大值、最小值、总和、唯一值、标准差。
![选择计量类型](./assets/NumberChartSelectAggType.png)
数据显示后可选择数据的展示格式，当前支持整数、浮点数、科学计数法等展示类型。
![选择格式](./assets/NumberChartSelectFormat.png)
点击应用即可保存图表到仪表盘。
![保存数表](./assets/SaveNumberChart.png)

#### 添加饼图
按照相同的方式，添加饼图
![添加饼图](./assets/AddPieChart.png)

#### 添加折线图
折线图的维度支持时间类型和数据类型。
时间类型可按照不同的时间段长度进行分组
![添加折线图](./assets/AddLineChart.png)
![选择分组](./assets/LineChartSelectTimeBin.png)
时间类型还支持按照特定时间进行抽取
![选择抽取](./assets/LineChartSelectExtract.png)
![选择抽取单位](./assets/LineChartSelectExtractBin.png)
图表没展示前，会提示你还有那些选项必选，选中后即可展示图表。如下图，我们还需选取计量字段。
![选好Bin](./assets/LineChartAfterSelectDimension.png)
图表展示后，可根据右侧边栏对图表的展示进行配置，包括图表样式，颜色，展示范围等等。
![SHOW](./assets/LineChartSelectAggType.png)

#### 添加点地图
![AddPoitMap](./assets/AddPointMap.png)

#### 一键布局
Infini当前支持4种一键布局方式，可选择自己需要的方式，或直接对图表拖拽设置自己自定义布局
![BeforLayout](./assets/DashBoardBeforeAutoLayouts.png)
![AfterLayout](./assets/DashBoardAutoLayout.png)


