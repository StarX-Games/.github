<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>IRON FLEET — 游戏策划文档 v1.0</title>
</head>
<body>
<div class="page">

<div class="cover">
  <div class="ctag">Game Design Document · IRON FLEET · v1.0 · AI-Readable Format</div>
  <h1>游戏策划文档</h1>
  <div class="csub">本文档按 AI 辅助开发规范编写——所有系统均包含精确数值、数据结构与逻辑公式，可直接作为 Claude Code 的上下文输入</div>
  <div class="cpills">
    <div class="cp">文档类型 <b>GDD v1.0</b></div>
    <div class="cp">游戏类型 <b>异步PVP策略建造</b></div>
    <div class="cp">平台 <b>iOS · Android · Steam</b></div>
    <div class="cp">引擎 <b>Unity 6</b></div>
    <div class="cp">编码风格 <b>C# · 数据驱动</b></div>
    <div class="cp">更新日期 <b>2026-04</b></div>
  </div>
</div>

<nav class="toc-bar">
  <div class="toc-title">目录 · 本文件共9章</div>
  <div class="toc-cols">
    <a href="#s1"><span>01</span>游戏概述与设计原则</a>
    <a href="#s2"><span>02</span>舰船建造系统</a>
    <a href="#s3"><span>03</span>电力管理系统</a>
    <a href="#s4"><span>04</span>AI芯片行为系统</a>
    <a href="#s5"><span>05</span>战斗系统</a>
    <a href="#s6"><span>06</span>船员系统</a>
    <a href="#s7"><span>07</span>舱室完整数据库</a>
    <a href="#s8"><span>08</span>经济与养成系统</a>
    <a href="#s9"><span>09</span>数据结构定义</a>
  </div>
</nav>


<!-- ═══ S1 ═══ -->
<section class="ch" id="s1">
  <div class="ch-hd"><span class="ch-n">01</span><h2 class="ch-t">游戏概述与设计原则</h2></div>

  <h3>核心定位</h3>
  <p>IRON FLEET 是一款<strong>异步 PVP 太空战舰建造策略游戏</strong>。玩家在 3D 等距截面视角下设计战舰布局、配置 AI 行为芯片，并以离线方式与全球玩家对战。核心体验来自三个维度的策略博弈：</p>
  <div class="g3" style="margin-top:10px">
    <div class="card"><h4>空间博弈</h4><p>有限格子内的最优舱室布局，走廊连通性约束，双甲板空间利用。</p></div>
    <div class="card"><h4>资源博弈</h4><p>电力上限始终不足，迫使玩家在进攻/防御/机动性之间持续取舍。</p></div>
    <div class="card"><h4>AI博弈</h4><p>通过芯片卡牌配置舱室行为逻辑，形成可与玩家手动操控竞争的自动化系统。</p></div>
  </div>

  <h3>设计原则（约束所有功能决策）</h3>
  <div class="tw"><table>
    <thead><tr><th>#</th><th>原则</th><th>含义</th><th>反例（禁止）</th></tr></thead>
    <tbody>
      <tr><td>P1</td><td><strong>电力永远不够</strong></td><td>任何舰船等级下，满载所有最优舱室的总电力需求必须超过反应堆上限20%以上</td><td>增加"超大反应堆"让玩家无需取舍</td></tr>
      <tr><td>P2</td><td><strong>付费不买数值</strong></td><td>所有武器/舱室/船员通过游戏内研究解锁，付费只加速或购买外观</td><td>付费专属高属性舱室</td></tr>
      <tr><td>P3</td><td><strong>防御靠设计而非付费</strong></td><td>防御力完全由布局+AI芯片决定，与付费无关</td><td>付费购买"防御加成"</td></tr>
      <tr><td>P4</td><td><strong>异步永不实时</strong></td><td>任何 PVP 战斗都是攻击方对防御方的 AI 快照作战，无需双方同时在线</td><td>实时 PVP 匹配</td></tr>
      <tr><td>P5</td><td><strong>日活15分钟设计</strong></td><td>每日核心游戏循环（收资源+看战报+攻击1-3次）不超过15分钟</td><td>需要长时间在线的玩法</td></tr>
    </tbody>
  </table></div>

  <h3>核心游戏循环（状态机描述）</h3>
  <pre><code><span class="cm">// 每日游戏状态流转（玩家视角）</span>
<span class="kw">enum</span> <span class="ty">DailyGameState</span> {
  COLLECT_OFFLINE_RESOURCES,    <span class="cm">// 收取离线矿物/气体产出（上限：离线16h）</span>
  VIEW_DEFENSE_REPLAYS,         <span class="cm">// 查看被攻击战斗回放（最多保留3条）</span>
  LAUNCH_PVP_ATTACKS,           <span class="cm">// 发起1-3次PVP进攻（由攻击令牌限制）</span>
  UPGRADE_OR_BUILD_ROOMS,       <span class="cm">// 升级或新建1个舱室（触发升级计时器）</span>
  COMPLETE_DAILY_MISSION,       <span class="cm">// 完成每日任务（奖励：矿物/气体/AI芯片碎片）</span>
  RESEARCH_TECH,                <span class="cm">// 排队研究（每次最多1项）</span>
  OPTIONAL_AD_WATCH             <span class="cm">// 选择性观看激励广告（每日上限5次）</span>
}

<span class="cm">// 预期日均游戏时长：10-20 分钟</span>
<span class="cm">// 攻击令牌恢复速度：1个/4小时，上限3个</span></code></pre>
</section>


<!-- ═══ S2 ═══ -->
<section class="ch" id="s2">
  <div class="ch-hd"><span class="ch-n">02</span><h2 class="ch-t">舰船建造系统</h2></div>

  <h3>2.1 网格系统规格</h3>
  <div class="box blue"><div class="box-lbl">系统约束</div><p>战舰视图为二维网格（2D Grid），单元格大小 = 1单位。网格坐标系：原点 (0,0) 在左下角，X轴向右，Y轴向上。战舰分<strong>上甲板（Deck A）</strong>和<strong>下甲板（Deck B）</strong>，各有独立网格，通过电梯间连通。</p></div>

  <div class="tw"><table>
    <thead><tr><th>舰船等级</th><th>Deck A 宽×高</th><th>Deck B 宽×高</th><th>总格子数</th><th>电力上限</th><th>最大舱室槽</th><th>解锁条件</th></tr></thead>
    <tbody>
      <tr><td>Lv.1 巡逻舰</td><td>8×4</td><td>8×3</td><td>56</td><td>6</td><td>8</td><td>初始</td></tr>
      <tr><td>Lv.2 护卫舰</td><td>10×5</td><td>10×4</td><td>90</td><td>10</td><td>12</td><td>指挥中心 Lv.3</td></tr>
      <tr><td>Lv.3 驱逐舰</td><td>12×6</td><td>12×5</td><td>132</td><td>14</td><td>16</td><td>指挥中心 Lv.6</td></tr>
      <tr><td>Lv.4 巡洋舰</td><td>14×7</td><td>14×6</td><td>182</td><td>18</td><td>20</td><td>指挥中心 Lv.9</td></tr>
      <tr><td>Lv.5 战列舰</td><td>16×8</td><td>16×7</td><td>240</td><td>22</td><td>25</td><td>指挥中心 Lv.12（上线后追加）</td></tr>
    </tbody>
  </table></div>

  <h3>2.2 舱室放置规则</h3>
  <pre><code><span class="cm">// 舱室放置验证逻辑（服务端+客户端双重验证）</span>
<span class="kw">bool</span> <span class="fn">CanPlaceRoom</span>(<span class="ty">RoomData</span> room, <span class="ty">Vector2Int</span> position, <span class="ty">DeckType</span> deck) {
    <span class="cm">// 规则1：舱室不能超出网格边界</span>
    <span class="kw">if</span> (!IsWithinBounds(position, room.Size, deck)) <span class="kw">return false</span>;

    <span class="cm">// 规则2：不能与已放置舱室重叠</span>
    <span class="kw">if</span> (HasOverlap(position, room.Size, deck)) <span class="kw">return false</span>;

    <span class="cm">// 规则3：跨甲板舱室（如反应堆）需要上下甲板同一列空间均空</span>
    <span class="kw">if</span> (room.SpansBothDecks &amp;&amp; !IsClearOnBothDecks(position, room.Size)) <span class="kw">return false</span>;

    <span class="cm">// 规则4：非走廊舱室必须与船员宿舍连通（BFS路径检查）</span>
    <span class="kw">if</span> (!room.IsCorridor &amp;&amp; !IsConnectedToCrewQuarters(position, deck)) <span class="kw">return false</span>;

    <span class="cm">// 规则5：每艘舰船最多放置舱室数 ≤ ShipLevel.MaxRoomSlots</span>
    <span class="kw">if</span> (currentRoomCount &gt;= GetMaxRoomSlots()) <span class="kw">return false</span>;

    <span class="kw">return true</span>;
}

<span class="cm">// 连通性检查：BFS从所有船员宿舍出发，检查目标位置是否可达</span>
<span class="kw">bool</span> <span class="fn">IsConnectedToCrewQuarters</span>(<span class="ty">Vector2Int</span> targetPos, <span class="ty">DeckType</span> deck) {
    <span class="cm">// 遍历所有走廊和电梯间，BFS路径查找</span>
    <span class="cm">// 走廊格子：可通行</span>
    <span class="cm">// 电梯间：可跨甲板通行</span>
    <span class="cm">// 气密舱门（关闭状态）：阻断路径</span>
}</code></pre>

  <h3>2.3 外部武器挂载点</h3>
  <p>每个舰船等级在船体外壳有固定数量的<strong>外部挂载点（HardPoint）</strong>。外部武器（无人机舱、外置激光炮）安装在挂载点上，<strong>不占用内部格子</strong>，但需要内部对应格子有电力连接线接口。</p>
  <div class="tw"><table>
    <thead><tr><th>舰船等级</th><th>外部挂载点数量</th><th>挂载点位置</th></tr></thead>
    <tbody>
      <tr><td>Lv.1</td><td>2</td><td>左侧1 + 右侧1</td></tr>
      <tr><td>Lv.2</td><td>4</td><td>左侧2 + 右侧2</td></tr>
      <tr><td>Lv.3</td><td>6</td><td>左侧2 + 右侧2 + 顶部2</td></tr>
      <tr><td>Lv.4</td><td>8</td><td>左侧3 + 右侧3 + 顶部2</td></tr>
    </tbody>
  </table></div>

  <h3>2.4 舱室协同效果矩阵</h3>
  <div class="box amber"><div class="box-lbl">协同触发条件</div><p>两个舱室在同一甲板上，且在8方向相邻（含对角线）时触发协同效果。协同效果仅当两个舱室均处于激活状态（有电力供应）时生效。</p></div>
  <div class="tw"><table>
    <thead><tr><th>舱室A</th><th>相邻舱室B</th><th>协同效果</th><th>数值</th></tr></thead>
    <tbody>
      <tr><td>激光炮台</td><td>工程舱</td><td>激光炮装填速度+</td><td>+10%</td></tr>
      <tr><td>导弹发射器</td><td>工程舱</td><td>导弹装填速度+</td><td>+8%</td></tr>
      <tr><td>医疗舱</td><td>船员宿舍</td><td>船员HP恢复速度+</td><td>+25%</td></tr>
      <tr><td>医疗舱</td><td>训练场</td><td>训练时间-</td><td>-15%</td></tr>
      <tr><td>护盾发生器</td><td>护盾发生器</td><td>护盾恢复速度+</td><td>+15%（仅双护盾触发）</td></tr>
      <tr><td>传送室</td><td>气密舱门控制</td><td>防反传送能力+</td><td>+20%（敌方登船检测范围扩大）</td></tr>
      <tr><td>研究实验室</td><td>研究实验室</td><td>研究速度+</td><td>+20%（仅双实验室触发）</td></tr>
      <tr><td>情报室</td><td>指挥中心</td><td>战前侦察范围+</td><td>额外显示2个舱室（默认显示1个）</td></tr>
      <tr><td>装甲强化舱</td><td>任意非走廊舱室</td><td>相邻舱室护甲值+</td><td>+30HP（被动）</td></tr>
      <tr><td>核反应堆</td><td>工程舱</td><td>反应堆HP恢复速度+</td><td>+20%（战斗中）</td></tr>
    </tbody>
  </table></div>
</section>


<!-- ═══ S3 ═══ -->
<section class="ch" id="s3">
  <div class="ch-hd"><span class="ch-n">03</span><h2 class="ch-t">电力管理系统</h2></div>

  <h3>3.1 电力规格</h3>
  <div class="formula">
    <span class="var">TotalPower</span> <span class="eq">=</span> <span class="var">ReactorBaseOutput</span> <span class="eq">×</span> <span class="num">ReactorLevel</span><br>
    <span class="cm">// ReactorBaseOutput = 2（每级核反应堆提供2点电力）</span><br>
    <span class="cm">// 初始核反应堆 Lv.1 = 2点；Lv.2 = 4点；最高 Lv.5 = 10点</span><br>
    <span class="var">AvailablePower</span> <span class="eq">=</span> <span class="var">TotalPower</span> <span class="eq">-</span> <span class="var">ActiveRoomsConsumption</span>
  </div>

  <h3>3.2 电力状态机</h3>
  <pre><code><span class="cm">// 每个功能舱室的电力状态</span>
<span class="kw">enum</span> <span class="ty">PowerState</span> {
  UNPOWERED = 0,   <span class="cm">// 无电力，舱室完全不工作</span>
  POWERED   = 1,   <span class="cm">// 正常运行（基础功能）</span>
  OVERLOADED = 2   <span class="cm">// 超载运行（需额外+1电力，效果+50%，持续30s后自动降回POWERED）</span>
}

<span class="cm">// 超载的代价（P1原则的具体实现）</span>
<span class="kw">struct</span> <span class="ty">OverloadCost</span> {
  <span class="ty">int</span> extraPowerRequired = <span class="num">1</span>;            <span class="cm">// 额外消耗1点电力</span>
  <span class="ty">float</span> performanceBonus = <span class="num">0.5f</span>;         <span class="cm">// 性能+50%</span>
  <span class="ty">float</span> heatDamagePerSecond = <span class="num">2.0f</span>;       <span class="cm">// 持续受热伤害2HP/秒</span>
  <span class="ty">float</span> maxDuration = <span class="num">30.0f</span>;             <span class="cm">// 最多维持30秒</span>
}</code></pre>

  <h3>3.3 战斗中手动电力调配</h3>
  <p>玩家（进攻方）在战斗中可实时点击任意舱室旁的电力按钮（+/-）进行电力重分配。每次调整有 <strong>0.5秒 冷却时间</strong>，防止快速刷电。AI芯片可设置自动电力转移动作。</p>
  <div class="box teal"><div class="box-lbl">电力调配规则</div>
    <p>① 增加电力：点击舱室+按钮，从可用电力池分配1点给该舱室。② 撤回电力：点击-按钮，从该舱室收回1点电力到可用池。③ 超载操作：当舱室已满载时，再次点击+执行超载（消耗额外电力）。④ 批量转移：长按舱室可打开"立即转移所有电力至护盾/武器"快捷按钮。</p>
  </div>
</section>


<!-- ═══ S4 ═══ -->
<section class="ch" id="s4">
  <div class="ch-hd"><span class="ch-n">04</span><h2 class="ch-t">AI 芯片行为系统</h2></div>

  <h3>4.1 系统概述</h3>
  <p>AI 芯片系统是本游戏的核心差异化功能。每个功能舱室可插入最多 <strong>2 张 AI 芯片</strong>，每张芯片由一个<strong>触发条件</strong>和一个<strong>执行动作</strong>组成。舱室按芯片插槽顺序（Slot 0 优先）执行第一个满足触发条件的芯片。</p>

  <h3>4.2 AI 芯片完整规格表</h3>
  <h4>触发条件芯片（Trigger Chips）— 上线版共12张</h4>
  <div class="tw"><table>
    <thead><tr><th>芯片ID</th><th>名称</th><th>触发条件（精确定义）</th><th>稀有度</th><th>获取方式</th></tr></thead>
    <tbody>
      <tr><td><code>TRIG_001</code></td><td>持续</td><td>始终满足（Always true）</td><td><span class="tag tg">普通</span></td><td>初始拥有</td></tr>
      <tr><td><code>TRIG_002</code></td><td>开局</td><td>战斗开始后 0-5秒内</td><td><span class="tag tg">普通</span></td><td>初始拥有</td></tr>
      <tr><td><code>TRIG_003</code></td><td>危急</td><td>己方舰船总HP ≤ 30%</td><td><span class="tag tg">普通</span></td><td>研究解锁（实验室Lv.2）</td></tr>
      <tr><td><code>TRIG_004</code></td><td>受击</td><td>本舱室在过去2秒内受到伤害</td><td><span class="tag tg">普通</span></td><td>研究解锁（实验室Lv.2）</td></tr>
      <tr><td><code>TRIG_005</code></td><td>护盾破裂</td><td>己方所有护盾HP = 0</td><td><span class="tag tb">稀有</span></td><td>研究解锁（实验室Lv.3）</td></tr>
      <tr><td><code>TRIG_006</code></td><td>优势</td><td>敌方舰船总HP ≤ 50%</td><td><span class="tag tb">稀有</span></td><td>研究解锁（实验室Lv.3）</td></tr>
      <tr><td><code>TRIG_007</code></td><td>船员离场</td><td>本舱室内无船员</td><td><span class="tag tb">稀有</span></td><td>研究解锁（实验室Lv.4）</td></tr>
      <tr><td><code>TRIG_008</code></td><td>登船警报</td><td>敌方船员进入己方舰船</td><td><span class="tag tb">稀有</span></td><td>研究解锁（实验室Lv.4）</td></tr>
      <tr><td><code>TRIG_009</code></td><td>低电量</td><td>己方可用电力 ≤ 1</td><td><span class="tag tp">史诗</span></td><td>赛季战令奖励</td></tr>
      <tr><td><code>TRIG_010</code></td><td>关键受损</td><td>指挥中心 HP ≤ 50%</td><td><span class="tag tp">史诗</span></td><td>锦标赛奖励</td></tr>
      <tr><td><code>TRIG_011</code></td><td>敌武器沉默</td><td>敌方全部武器舱室被离子炮断电</td><td><span class="tag tp">史诗</span></td><td>研究解锁（实验室Lv.5）</td></tr>
      <tr><td><code>TRIG_012</code></td><td>精英时机</td><td>己方传说级船员HP &gt; 80%</td><td><span class="tag ta">传说</span></td><td>创始玩家专属</td></tr>
    </tbody>
  </table></div>

  <h4>执行动作芯片（Action Chips）— 上线版共14张</h4>
  <div class="tw"><table>
    <thead><tr><th>芯片ID</th><th>名称</th><th>执行动作（精确定义）</th><th>适用舱室</th><th>稀有度</th></tr></thead>
    <tbody>
      <tr><td><code>ACT_001</code></td><td>瞄准护盾</td><td>将武器目标设为敌方最高HP护盾舱室</td><td>所有武器</td><td><span class="tag tg">普通</span></td></tr>
      <tr><td><code>ACT_002</code></td><td>瞄准武器</td><td>将武器目标设为敌方当前激活的武器舱室（HP最低者）</td><td>所有武器</td><td><span class="tag tg">普通</span></td></tr>
      <tr><td><code>ACT_003</code></td><td>增加电力</td><td>从可用电力池分配1点给本舱室（若可用电力≥1）</td><td>所有舱室</td><td><span class="tag tg">普通</span></td></tr>
      <tr><td><code>ACT_004</code></td><td>撤回电力</td><td>将本舱室1点电力归还可用池</td><td>所有舱室</td><td><span class="tag tg">普通</span></td></tr>
      <tr><td><code>ACT_005</code></td><td>呼叫维修</td><td>派遣最近的空闲船员前往本舱室执行维修</td><td>所有舱室</td><td><span class="tag tb">稀有</span></td></tr>
      <tr><td><code>ACT_006</code></td><td>超载激活</td><td>激活本舱室超载模式（需额外1点电力，效果+50%，持续30s）</td><td>武器/护盾</td><td><span class="tag tb">稀有</span></td></tr>
      <tr><td><code>ACT_007</code></td><td>精准瞄准</td><td>将武器目标设为敌方最低HP非护盾舱室</td><td>所有武器</td><td><span class="tag tb">稀有</span></td></tr>
      <tr><td><code>ACT_008</code></td><td>转移电力至护盾</td><td>将本舱室所有电力转移给护盾舱室（优先最近的护盾）</td><td>武器/辅助</td><td><span class="tag tb">稀有</span></td></tr>
      <tr><td><code>ACT_009</code></td><td>关闭气密门</td><td>关闭本舱室周围所有气密门（阻断走廊通道）</td><td>气密舱门控制</td><td><span class="tag tb">稀有</span></td></tr>
      <tr><td><code>ACT_010</code></td><td>紧急撤退</td><td>命令本舱室所有船员移动至指挥中心</td><td>所有舱室</td><td><span class="tag tp">史诗</span></td></tr>
      <tr><td><code>ACT_011</code></td><td>瞄准指挥中心</td><td>将武器目标设为敌方指挥中心（无视HP大小）</td><td>所有武器</td><td><span class="tag tp">史诗</span></td></tr>
      <tr><td><code>ACT_012</code></td><td>集火命令</td><td>将己方所有武器目标统一设为同一个舱室（最脆弱者）</td><td>指挥中心</td><td><span class="tag tp">史诗</span></td></tr>
      <tr><td><code>ACT_013</code></td><td>传送突袭</td><td>立即将最强船员传送至敌方指挥中心区域</td><td>传送室</td><td><span class="tag ta">传说</span></td></tr>
      <tr><td><code>ACT_014</code></td><td>全舰超载</td><td>同时激活所有武器舱室超载模式（消耗全部可用电力）</td><td>指挥中心</td><td><span class="tag ta">传说</span></td></tr>
    </tbody>
  </table></div>

  <h3>4.3 AI执行逻辑</h3>
  <pre><code><span class="cm">// 每个舱室每帧（0.5s tick）执行一次 AI 评估</span>
<span class="kw">void</span> <span class="fn">EvaluateAIChips</span>(<span class="ty">Room</span> room, <span class="ty">BattleState</span> state) {
    <span class="kw">foreach</span> (<span class="ty">AIChip</span> chip <span class="kw">in</span> room.InstalledChips) {  <span class="cm">// 最多2张，Slot0优先</span>
        <span class="kw">if</span> (chip.Trigger.<span class="fn">IsSatisfied</span>(state)) {
            chip.Action.<span class="fn">Execute</span>(room, state);
            <span class="kw">break</span>;  <span class="cm">// 执行第一个满足条件的芯片后停止</span>
        }
    }
}

<span class="cm">// 进攻方手动操控可以实时覆盖AI芯片的目标选择</span>
<span class="cm">// 覆盖持续时间：5秒，之后恢复AI控制</span>
<span class="cm">// 防御方：完全由AI芯片控制，无手动输入</span></code></pre>

  <h3>4.4 芯片配置策略范例（供玩家教程参考）</h3>
  <div class="g2">
    <div class="card">
      <h4>激光炮 — 标准进攻配置</h4>
      <pre style="margin:0"><code>Slot 0: TRIG_002(开局) + ACT_001(瞄准护盾)
Slot 1: TRIG_005(护盾破裂) + ACT_007(精准瞄准)
<span class="cm">// 开局先打护盾，护盾破后打关键舱室</span></code></pre>
    </div>
    <div class="card">
      <h4>护盾 — 紧急防御配置</h4>
      <pre style="margin:0"><code>Slot 0: TRIG_003(危急) + ACT_006(超载激活)
Slot 1: TRIG_001(持续) + ACT_003(增加电力)
<span class="cm">// 危急时护盾超载，平时尽量抢电力</span></code></pre>
    </div>
  </div>
</section>


<!-- ═══ S5 ═══ -->
<section class="ch" id="s5">
  <div class="ch-hd"><span class="ch-n">05</span><h2 class="ch-t">战斗系统</h2></div>

  <h3>5.1 战斗流程与时序</h3>
  <pre><code><span class="cm">// 战斗总时长上限：180秒</span>
<span class="cm">// 战斗 Tick 频率：服务端 20Hz（50ms/tick），客户端插值渲染</span>

<span class="kw">enum</span> <span class="ty">BattlePhase</span> {
  PRE_BATTLE_SCAN    = 0,   <span class="cm">// 0-3秒：情报室侦察，显示敌方部分舱室信息</span>
  COMBAT_ACTIVE      = 1,   <span class="cm">// 3-180秒：正式战斗</span>
  RESULT_CALCULATION = 2,   <span class="cm">// 180秒：超时结算</span>
}

<span class="cm">// 胜负判定规则</span>
<span class="kw">enum</span> <span class="ty">BattleResult</span> {
  ATTACKER_WIN,    <span class="cm">// 敌方指挥中心HP = 0</span>
  DEFENDER_WIN,    <span class="cm">// 攻击方时间耗尽（180s），己方指挥中心HP &gt; 0</span>
  DRAW,            <span class="cm">// 双方指挥中心HP相等时超时（极少发生）</span>
}

<span class="cm">// 超时判定：HP% 较低一方判负（HP%相同则攻击方判负）</span></code></pre>

  <h3>5.2 伤害计算公式</h3>
  <div class="formula">
    <span class="cm">// 基础伤害计算</span><br>
    <span class="var">RawDamage</span> <span class="eq">=</span> <span class="var">WeaponBaseDamage</span> <span class="eq">×</span> (1 + <span class="var">CrewAttackBonus</span>) <span class="eq">×</span> <span class="var">OverloadMultiplier</span><br><br>
    <span class="cm">// 护盾伤害（护盾先吸收，再对舱室造成穿透伤害）</span><br>
    <span class="var">ShieldDamage</span> <span class="eq">=</span> <span class="var">RawDamage</span> <span class="eq">×</span> <span class="var">WeaponShieldEfficiency</span>  <span class="cm">// 激光=0.5，导弹=1.0，离子=0.0</span><br>
    <span class="var">PenetrationDamage</span> <span class="eq">=</span> max(0, <span class="var">RawDamage</span> <span class="eq">-</span> <span class="var">ShieldDamage</span> <span class="eq">×</span> 2)  <span class="cm">// 护盾吸收后剩余穿透</span><br><br>
    <span class="cm">// 舱室伤害（扣减护甲后）</span><br>
    <span class="var">ActualRoomDamage</span> <span class="eq">=</span> max(1, <span class="var">PenetrationDamage</span> <span class="eq">-</span> <span class="var">RoomArmor</span>)  <span class="cm">// 最低1点伤害</span><br><br>
    <span class="cm">// 超载加成</span><br>
    <span class="var">OverloadMultiplier</span> <span class="eq">=</span> 1.5 (超载中) / 1.0 (正常)
  </div>

  <h3>5.3 武器详细参数</h3>
  <div class="tw"><table>
    <thead><tr><th>武器</th><th>基础伤害/发</th><th>攻击间隔(s)</th><th>DPS</th><th>对护盾效率</th><th>穿透率</th><th>射程（格）</th><th>特殊效果</th></tr></thead>
    <tbody>
      <tr><td>激光炮台 Lv.1</td><td>12</td><td>1.5</td><td>8</td><td>50%</td><td>0%</td><td>无限</td><td>—</td></tr>
      <tr><td>激光炮台 Lv.5</td><td>28</td><td>1.2</td><td>23</td><td>50%</td><td>0%</td><td>无限</td><td>—</td></tr>
      <tr><td>导弹发射器 Lv.1</td><td>45</td><td>6.0</td><td>7.5</td><td>100%</td><td>30%</td><td>无限</td><td>范围伤害1格内50%</td></tr>
      <tr><td>导弹发射器 Lv.5</td><td>90</td><td>5.0</td><td>18</td><td>100%</td><td>40%</td><td>无限</td><td>范围伤害1格内60%</td></tr>
      <tr><td>离子炮 Lv.1</td><td>0</td><td>4.0</td><td>0</td><td>0%</td><td>100%</td><td>无限</td><td>命中断电目标舱室2秒</td></tr>
      <tr><td>离子炮 Lv.5</td><td>0</td><td>3.5</td><td>0</td><td>0%</td><td>100%</td><td>无限</td><td>命中断电目标舱室3.5秒</td></tr>
      <tr><td>无人机舱 Lv.1</td><td>8/无人机</td><td>3s（无人机持续攻击）</td><td>16（2架）</td><td>70%</td><td>10%</td><td>全图</td><td>同时释放2架无人机，各自寻找目标</td></tr>
      <tr><td>无人机舱 Lv.5</td><td>15/无人机</td><td>2.5s</td><td>36（3架）</td><td>70%</td><td>20%</td><td>全图</td><td>同时释放3架无人机</td></tr>
    </tbody>
  </table></div>

  <h3>5.4 战斗回放系统规格</h3>
  <div class="box green"><div class="box-lbl">技术规格（服务端实现要求）</div>
    <p>战斗回放以<strong>指令序列（Command Sequence）</strong>方式存储，非视频流。服务端记录战斗中所有输入事件及 Tick 时间戳，客户端下载后本地重新演算并渲染。单场战斗数据大小上限：<strong>64KB</strong>。保留时长：<strong>72小时</strong>。</p>
  </div>
  <pre><code><span class="cm">// 战斗回放数据结构</span>
<span class="kw">struct</span> <span class="ty">BattleReplay</span> {
  <span class="ty">string</span> battleId;
  <span class="ty">long</span> startTimestamp;
  <span class="ty">string</span> attackerShipSnapshot;  <span class="cm">// 攻击方舰船JSON快照（含AI芯片配置）</span>
  <span class="ty">string</span> defenderShipSnapshot;  <span class="cm">// 防御方舰船JSON快照</span>
  <span class="ty">List</span>&lt;<span class="ty">CommandEvent</span>&gt; commands;  <span class="cm">// 攻击方手动操控事件序列</span>
  <span class="ty">BattleResult</span> result;
  <span class="ty">int</span> attackerFinalHpPercent;
  <span class="ty">int</span> defenderFinalHpPercent;
}

<span class="kw">struct</span> <span class="ty">CommandEvent</span> {
  <span class="ty">float</span> tick;              <span class="cm">// 事件发生时刻（秒，精度0.05s）</span>
  <span class="ty">CommandType</span> type;        <span class="cm">// TARGET_CHANGE | POWER_ADJUST | OVERLOAD | MANUAL_TELEPORT</span>
  <span class="ty">string</span> targetRoomId;     <span class="cm">// 操作目标舱室ID</span>
  <span class="ty">int</span> value;               <span class="cm">// 附加参数（如电力增减量）</span>
}</code></pre>

  <h3>5.5 PVP 匹配规则</h3>
  <div class="tw"><table>
    <thead><tr><th>条件</th><th>规则</th></tr></thead>
    <tbody>
      <tr><td>匹配基准</td><td>优先同舰船等级（±0），无对手时扩展至±1级，再无则±2级</td></tr>
      <tr><td>战力分</td><td>综合舱室总HP × 0.3 + 武器DPS总和 × 0.5 + 船员总属性 × 0.2</td></tr>
      <tr><td>匹配范围</td><td>战力分差值 ≤ 15%（扩展匹配时逐步放宽至30%/50%）</td></tr>
      <tr><td>同一对手间隔</td><td>同一玩家在24小时内只能被攻击3次，超出后不进入匹配池</td></tr>
      <tr><td>新手保护</td><td>注册7天内：只匹配其他7天内新玩家（新手池独立）</td></tr>
      <tr><td>匹配超时</td><td>30秒内无匹配对手则推荐NPC战舰（不计积分，有资源奖励）</td></tr>
    </tbody>
  </table></div>
</section>


<!-- ═══ S6 ═══ -->
<section class="ch" id="s6">
  <div class="ch-hd"><span class="ch-n">06</span><h2 class="ch-t">船员系统</h2></div>

  <h3>6.1 船员属性定义</h3>
  <pre><code><span class="kw">struct</span> <span class="ty">CrewStats</span> {
  <span class="ty">int</span> MaxHP;          <span class="cm">// 血量上限（100-200）</span>
  <span class="ty">int</span> Attack;         <span class="cm">// 近战攻击力（60-180），决定登船肉搏DPS</span>
  <span class="ty">float</span> MoveSpeed;    <span class="cm">// 移动速度（0.8-1.3格/秒），影响到达目标速度</span>
  <span class="ty">float</span> RepairSpeed;  <span class="cm">// 维修速度（0.5-1.5HP/秒），与工程舱协同叠加</span>
  <span class="ty">string</span> SpecialAbility; <span class="cm">// 特殊能力描述（详见6.2）</span>
}

<span class="cm">// 训练加成计算（每次训练提升基础属性）</span>
<span class="var">TrainedStat</span> <span class="eq">=</span> <span class="var">BaseStat</span> <span class="eq">×</span> (1 + <span class="var">TrainingLevel</span> <span class="eq">×</span> <span class="num">0.03</span>)
<span class="cm">// TrainingLevel 上限 = 20，最高训练加成 = +60%</span>
<span class="cm">// 训练时间（小时）= BaseStat × 0.1 × (1 + TrainingLevel × 0.2)</span></code></pre>

  <h3>6.2 上线版12名船员完整数据</h3>
  <div class="tw"><table>
    <thead><tr><th>ID</th><th>名称</th><th>稀有度</th><th>MaxHP</th><th>Attack</th><th>Speed</th><th>Repair</th><th>特殊能力（精确描述）</th><th>获取方式</th></tr></thead>
    <tbody>
      <tr><td>CREW_001</td><td>铁甲工程师</td><td><span class="tag tg">普通</span></td><td>120</td><td>80</td><td>1.0</td><td>1.5</td><td>维修速度额外+20%，不受其他加成影响</td><td>初始拥有1名</td></tr>
      <tr><td>CREW_002</td><td>联邦士兵</td><td><span class="tag tg">普通</span></td><td>150</td><td>120</td><td>0.9</td><td>0.5</td><td>近战攻击附加10%无视护甲</td><td>新手任务奖励</td></tr>
      <tr><td>CREW_003</td><td>救援医官</td><td><span class="tag tg">普通</span></td><td>100</td><td>60</td><td>1.1</td><td>1.2</td><td>自身HP &lt; 50%时移动速度+30%</td><td>招募卡初级抽取</td></tr>
      <tr><td>CREW_004</td><td>精英狙击手</td><td><span class="tag tb">稀有</span></td><td>110</td><td>180</td><td>0.85</td><td>0.4</td><td>每场战斗第一次近战攻击必暴击（×2伤害）</td><td>招募卡中级抽取</td></tr>
      <tr><td>CREW_005</td><td>护盾技师</td><td><span class="tag tb">稀有</span></td><td>130</td><td>90</td><td>0.95</td><td>0.8</td><td>所在舱室为护盾舱时，护盾恢复速度+25%</td><td>招募卡中级抽取</td></tr>
      <tr><td>CREW_006</td><td>爆破专家</td><td><span class="tag tb">稀有</span></td><td>140</td><td>150</td><td>0.8</td><td>0.3</td><td>驻守武器舱时该武器对舱室伤害+20%</td><td>完成第5章主线任务</td></tr>
      <tr><td>CREW_007</td><td>科学家</td><td><span class="tag tb">稀有</span></td><td>90</td><td>70</td><td>1.0</td><td>0.6</td><td>驻守研究实验室时研究速度+30%</td><td>招募卡中级抽取</td></tr>
      <tr><td>CREW_008</td><td>电力专家</td><td><span class="tag tp">史诗</span></td><td>115</td><td>85</td><td>1.05</td><td>1.0</td><td>驻守任意舱室时该舱室电力消耗-1（最低1）</td><td>赛季战令Lv.30</td></tr>
      <tr><td>CREW_009</td><td>离子武者</td><td><span class="tag tp">史诗</span></td><td>125</td><td>160</td><td>1.0</td><td>0.5</td><td>近战攻击附加2秒断电效果（与离子炮效果叠加）</td><td>锦标赛前32名奖励</td></tr>
      <tr><td>CREW_010</td><td>传送突击队</td><td><span class="tag tp">史诗</span></td><td>160</td><td>140</td><td>1.2</td><td>0.3</td><td>传送室冷却时间-40%；传送时携带1名最近的普通船员同行</td><td>招募卡高级抽取</td></tr>
      <tr><td>CREW_011</td><td>舰队指挥官</td><td><span class="tag ta">传说</span></td><td>170</td><td>110</td><td>0.95</td><td>0.7</td><td>驻守指挥中心时：全体船员所有属性+10%；指挥中心HP+20%</td><td>传说级招募（低概率）</td></tr>
      <tr><td>CREW_012</td><td>自主作战AI</td><td><span class="tag ta">传说</span></td><td>200</td><td>130</td><td>1.3</td><td>1.2</td><td>AI芯片执行效率+50%（Tick频率从0.5s降至0.33s）；无需走廊路径即可瞬移至目标舱室</td><td>赛季战令Lv.50</td></tr>
    </tbody>
  </table></div>

  <h3>6.3 船员行为逻辑</h3>
  <pre><code><span class="cm">// 船员默认行为优先级（无AI芯片指令时）</span>
<span class="kw">enum</span> <span class="ty">CrewDefaultBehavior</span> {
  PRIORITY_1_REPAIR_CRITICAL,  <span class="cm">// 修复HP &lt; 20% 的舱室（最优先）</span>
  PRIORITY_2_FIGHT_BOARDERS,   <span class="cm">// 攻击己方舰内的敌方船员</span>
  PRIORITY_3_REPAIR_DAMAGED,   <span class="cm">// 修复HP &lt; 60% 的舱室</span>
  PRIORITY_4_STAY_IN_ROOM,     <span class="cm">// 驻守最近的功能舱室</span>
}

<span class="cm">// 船员移动：A*寻路，走廊格为可通行节点，关闭气密门为不可通行节点</span>
<span class="cm">// 电梯间：连通上下甲板，穿越耗时2秒</span>
<span class="cm">// 肉搏战：双方都在同一舱室时自动进入近战，每0.5s双方互相造成Attack/2伤害</span></code></pre>
</section>


<!-- ═══ S7 ═══ -->
<section class="ch" id="s7">
  <div class="ch-hd"><span class="ch-n">07</span><h2 class="ch-t">舱室完整数据库</h2></div>

  <div class="box amber"><div class="box-lbl">数据库说明</div><p>下表为上线版全部18个舱室的精确数值。所有数值按 Lv.1 基础值列出；每级升级乘以 <strong>1.15</strong>（HP/输出类属性）或 <strong>0.95</strong>（时间类属性）。护甲值（Armor）为固定值不随等级变化。</p></div>

  <div class="tw"><table>
    <thead><tr><th>ID</th><th>名称</th><th>尺寸(宽×高)</th><th>所在甲板</th><th>电力消耗</th><th>基础HP</th><th>护甲值</th><th>升级上限</th><th>升级耗时(s) Lv.1→2</th><th>核心功能数值</th></tr></thead>
    <tbody>
      <tr><td>ROOM_001</td><td>指挥中心</td><td>2×2</td><td>双甲板</td><td>0（自供）</td><td>400</td><td>20</td><td>12</td><td>3600</td><td>每级+50HP，每级+1格子解锁</td></tr>
      <tr><td>ROOM_002</td><td>核反应堆</td><td>2×2</td><td>双甲板</td><td>0（自供）</td><td>300</td><td>15</td><td>10</td><td>7200</td><td>每级+2电力上限</td></tr>
      <tr><td>ROOM_003</td><td>船员宿舍</td><td>1×2</td><td>任意</td><td>0</td><td>150</td><td>5</td><td>5</td><td>1800</td><td>每级+1船员上限（基础2）</td></tr>
      <tr><td>ROOM_004</td><td>电梯间</td><td>1×2</td><td>双甲板（跨层）</td><td>1</td><td>200</td><td>10</td><td>5</td><td>3600</td><td>每级-0.3s穿越耗时（基础2s）</td></tr>
      <tr><td>ROOM_005</td><td>走廊</td><td>1×1</td><td>任意</td><td>0</td><td>80</td><td>0</td><td>3</td><td>600</td><td>每级+20HP，无功能加成</td></tr>
      <tr><td>ROOM_006</td><td>护盾发生器</td><td>2×1</td><td>任意</td><td>2</td><td>250</td><td>10</td><td>10</td><td>5400</td><td>每级：护盾HP+80，恢复速率+3HP/s（基础5HP/s）</td></tr>
      <tr><td>ROOM_007</td><td>医疗舱</td><td>2×1</td><td>任意</td><td>1</td><td>180</td><td>5</td><td>8</td><td>2400</td><td>每级：船员HP恢复+0.5HP/s（基础1HP/s）</td></tr>
      <tr><td>ROOM_008</td><td>气密舱门控制</td><td>1×1</td><td>任意</td><td>1</td><td>120</td><td>5</td><td>5</td><td>1800</td><td>可控制半径2格内所有走廊门；每级+0.5格控制范围</td></tr>
      <tr><td>ROOM_009</td><td>装甲强化舱</td><td>1×2</td><td>任意</td><td>0</td><td>350</td><td>30</td><td>8</td><td>3600</td><td>相邻舱室护甲+30（基础），每级+5；自身HP每级+50</td></tr>
      <tr><td>ROOM_010</td><td>激光炮台</td><td>2×1</td><td>任意</td><td>2</td><td>200</td><td>5</td><td>10</td><td>4800</td><td>见5.3武器参数表</td></tr>
      <tr><td>ROOM_011</td><td>导弹发射器</td><td>2×2</td><td>任意</td><td>2</td><td>250</td><td>8</td><td>10</td><td>7200</td><td>见5.3武器参数表</td></tr>
      <tr><td>ROOM_012</td><td>离子炮</td><td>2×1</td><td>任意</td><td>3</td><td>200</td><td>5</td><td>10</td><td>7200</td><td>见5.3武器参数表</td></tr>
      <tr><td>ROOM_013</td><td>传送室</td><td>2×2</td><td>任意</td><td>3</td><td>220</td><td>8</td><td>8</td><td>9000</td><td>传送冷却时间：基础45s，每级-3s；每次可传送1名船员</td></tr>
      <tr><td>ROOM_014</td><td>无人机舱</td><td>2×2</td><td>任意（需外部挂载点）</td><td>3</td><td>180</td><td>5</td><td>8</td><td>9000</td><td>见5.3武器参数表；无人机HP=40，被击中即销毁</td></tr>
      <tr><td>ROOM_015</td><td>矿物开采仓</td><td>2×1</td><td>任意</td><td>1</td><td>160</td><td>3</td><td>10</td><td>3600</td><td>矿物产出：基础100/h，每级+30/h；上限8h产出</td></tr>
      <tr><td>ROOM_016</td><td>研究实验室</td><td>2×2</td><td>任意</td><td>1</td><td>200</td><td>5</td><td>8</td><td>7200</td><td>解锁研究树节点；每级-10%研究时间；科学家协同+30%</td></tr>
      <tr><td>ROOM_017</td><td>训练场</td><td>2×2</td><td>任意</td><td>1</td><td>200</td><td>5</td><td>8</td><td>7200</td><td>训练队列上限：基础1，每级+0；同时只能训练1名船员</td></tr>
      <tr><td>ROOM_018</td><td>情报室</td><td>2×1</td><td>任意</td><td>1</td><td>160</td><td>3</td><td>6</td><td>3600</td><td>战前侦察：基础显示1个随机敌方舱室，每级+0.5个（取整）</td></tr>
    </tbody>
  </table></div>
</section>


<!-- ═══ S8 ═══ -->
<section class="ch" id="s8">
  <div class="ch-hd"><span class="ch-n">08</span><h2 class="ch-t">经济与养成系统</h2></div>

  <h3>8.1 三种资源规格</h3>
  <div class="tw"><table>
    <thead><tr><th>资源</th><th>代码名</th><th>用途</th><th>主要获取</th><th>初始储量上限</th><th>上限扩展方式</th></tr></thead>
    <tbody>
      <tr><td>铁矿石</td><td><code>IRON</code></td><td>建造/升级舱室</td><td>矿物开采仓（离线）+ PVP奖励</td><td>5,000</td><td>每升1级舰船+2,000</td></tr>
      <tr><td>量子晶体</td><td><code>CRYSTAL</code></td><td>研究技术/训练船员</td><td>PVP胜利 + 每日任务 + 广告奖励</td><td>2,000</td><td>升级仓库（专用舱室，上线后追加）</td></tr>
      <tr><td>星际钻</td><td><code>STARDIAMOND</code></td><td>加速/招募/付费购买</td><td>完成成就 + 付费购买</td><td>无上限</td><td>—</td></tr>
    </tbody>
  </table></div>

  <h3>8.2 舱室升级费用曲线</h3>
  <div class="formula">
    <span class="cm">// 升级费用（铁矿石）</span><br>
    <span class="var">IronCost</span>(<span class="var">level</span>) <span class="eq">=</span> <span class="var">BaseIronCost</span> <span class="eq">×</span> pow(1.5, <span class="var">level</span> - 1)<br><br>
    <span class="cm">// 升级时间（秒）</span><br>
    <span class="var">UpgradeTime</span>(<span class="var">level</span>) <span class="eq">=</span> <span class="var">BaseUpgradeTime</span> <span class="eq">×</span> pow(1.8, <span class="var">level</span> - 1)<br><br>
    <span class="cm">// 加速费用（星际钻）= ceil(剩余秒数 / 600)  → 每10分钟1颗钻</span><br>
    <span class="var">SpeedupCost</span>(<span class="var">remainingSeconds</span>) <span class="eq">=</span> ceil(<span class="var">remainingSeconds</span> / <span class="num">600</span>)
  </div>

  <h3>8.3 离线产出计算</h3>
  <pre><code><span class="cm">// 服务端登录时计算（防止客户端篡改）</span>
<span class="ty">long</span> <span class="fn">CalculateOfflineProduction</span>(<span class="ty">PlayerData</span> player, <span class="ty">long</span> currentTimestamp) {
    <span class="ty">long</span> offlineSeconds = currentTimestamp - player.lastLoginTimestamp;

    <span class="cm">// 分段产出计算</span>
    <span class="kw">if</span> (offlineSeconds &lt;= <span class="num">28800</span>) {       <span class="cm">// 0-8小时：100%产出</span>
        <span class="kw">return</span> player.ironPerHour <span class="eq">×</span> (offlineSeconds / <span class="num">3600.0</span>);
    } <span class="kw">else if</span> (offlineSeconds &lt;= <span class="num">57600</span>) { <span class="cm">// 8-16小时：60%产出</span>
        <span class="ty">long</span> fullProd = player.ironPerHour <span class="eq">×</span> <span class="num">8</span>;
        <span class="ty">long</span> reducedProd = player.ironPerHour <span class="eq">×</span> <span class="num">0.6</span> <span class="eq">×</span> ((offlineSeconds - <span class="num">28800</span>) / <span class="num">3600.0</span>);
        <span class="kw">return</span> fullProd + reducedProd;
    } <span class="kw">else</span> {                               <span class="cm">// 16小时以上：停止产出</span>
        <span class="kw">return</span> player.ironPerHour <span class="eq">×</span> (<span class="num">8</span> + <span class="num">8</span> <span class="eq">×</span> <span class="num">0.6</span>);  <span class="cm">// 12.8小时等效产出</span>
    }
}</code></pre>

  <h3>8.4 PVP 积分与奖励</h3>
  <div class="tw"><table>
    <thead><tr><th>条件</th><th>积分变化</th><th>资源奖励</th></tr></thead>
    <tbody>
      <tr><td>进攻胜利</td><td>+20 ~ +35（根据对手等级差）</td><td>IRON×200 + CRYSTAL×50</td></tr>
      <tr><td>进攻失败</td><td>-5 ~ -10</td><td>IRON×20（参与奖励）</td></tr>
      <tr><td>防御胜利</td><td>+5</td><td>IRON×100（下次登录领取）</td></tr>
      <tr><td>防御失败</td><td>-15 ~ -25</td><td>无</td></tr>
      <tr><td>平局</td><td>0</td><td>IRON×100 + CRYSTAL×25</td></tr>
    </tbody>
  </table></div>

  <h3>8.5 研究树结构</h3>
  <pre><code><span class="cm">// 研究树节点（上线版共32个节点，分4个分支）</span>
<span class="kw">enum</span> <span class="ty">ResearchBranch</span> {
  WEAPONS,      <span class="cm">// 12个节点：解锁新武器、提升武器属性</span>
  DEFENSE,      <span class="cm">// 8个节点：护盾强化、装甲技术</span>
  CREW,         <span class="cm">// 7个节点：船员训练速度、招募池扩展</span>
  AI_CHIPS,     <span class="cm">// 5个节点：解锁高级AI芯片（TRIG_003~012, ACT_005~014）</span>
}

<span class="cm">// 研究时间范围：1小时（基础节点）~ 72小时（高级节点）</span>
<span class="cm">// 前置条件：每个节点有0-2个前置研究</span>
<span class="cm">// 消耗：CRYSTAL（主要）+ IRON（部分节点）</span></code></pre>
</section>


<!-- ═══ S9 ═══ -->
<section class="ch" id="s9">
  <div class="ch-hd"><span class="ch-n">09</span><h2 class="ch-t">数据结构定义（JSON Schema）</h2></div>

  <div class="box blue"><div class="box-lbl">用途说明</div><p>以下JSON结构定义用于：① 服务端 Orleans Grain 的持久化状态；② 客户端本地缓存格式；③ Claude Code 生成相关 C# 类时的参考依据。所有字段均为必填，除非标注 <code>optional</code>。</p></div>

  <h3>9.1 玩家数据</h3>
  <pre><code>{
  <span class="str">"userId"</span>: <span class="str">"string (Firebase UID)"</span>,
  <span class="str">"username"</span>: <span class="str">"string (3-16 chars)"</span>,
  <span class="str">"faction"</span>: <span class="str">"FEDERATION | PIRATE | TECH"</span>,
  <span class="str">"shipLevel"</span>: <span class="str">"int (1-5)"</span>,
  <span class="str">"trophies"</span>: <span class="str">"int"</span>,
  <span class="str">"resources"</span>: {
    <span class="str">"iron"</span>: <span class="str">"long"</span>,
    <span class="str">"crystal"</span>: <span class="str">"long"</span>,
    <span class="str">"starDiamond"</span>: <span class="str">"int"</span>
  },
  <span class="str">"attackTokens"</span>: <span class="str">"int (0-3)"</span>,
  <span class="str">"lastLoginTimestamp"</span>: <span class="str">"long (Unix ms)"</span>,
  <span class="str">"lastAttackTokenRechargeTimestamp"</span>: <span class="str">"long (Unix ms)"</span>,
  <span class="str">"shipSnapshot"</span>: <span class="str">"ShipData (see below)"</span>,
  <span class="str">"crew"</span>: <span class="str">"List&lt;OwnedCrew&gt;"</span>,
  <span class="str">"researchCompleted"</span>: <span class="str">"List&lt;string&gt; (research node IDs)"</span>,
  <span class="str">"aiChipsOwned"</span>: <span class="str">"Dict&lt;string, int&gt; (chipId -&gt; count)"</span>,
  <span class="str">"purchaseHistory"</span>: <span class="str">"optional List&lt;PurchaseRecord&gt;"</span>,
  <span class="str">"seasonPassLevel"</span>: <span class="str">"int (0-50)"</span>,
  <span class="str">"hasActiveSubscription"</span>: <span class="str">"bool"</span>
}</code></pre>

  <h3>9.2 舰船布局数据（ShipData）</h3>
  <pre><code>{
  <span class="str">"shipId"</span>: <span class="str">"string"</span>,
  <span class="str">"deckA"</span>: {
    <span class="str">"width"</span>: <span class="str">"int"</span>,
    <span class="str">"height"</span>: <span class="str">"int"</span>,
    <span class="str">"rooms"</span>: [
      {
        <span class="str">"instanceId"</span>: <span class="str">"string (uuid)"</span>,
        <span class="str">"roomTypeId"</span>: <span class="str">"string (ROOM_001...ROOM_018)"</span>,
        <span class="str">"position"</span>: { <span class="str">"x"</span>: <span class="str">"int"</span>, <span class="str">"y"</span>: <span class="str">"int"</span> },
        <span class="str">"level"</span>: <span class="str">"int"</span>,
        <span class="str">"aiChips"</span>: [
          { <span class="str">"slot"</span>: <span class="num">0</span>, <span class="str">"triggerId"</span>: <span class="str">"TRIG_001"</span>, <span class="str">"actionId"</span>: <span class="str">"ACT_001"</span> },
          { <span class="str">"slot"</span>: <span class="num">1</span>, <span class="str">"triggerId"</span>: <span class="str">"TRIG_003"</span>, <span class="str">"actionId"</span>: <span class="str">"ACT_006"</span> }
        ],
        <span class="str">"assignedCrewId"</span>: <span class="str">"optional string"</span>
      }
    ]
  },
  <span class="str">"deckB"</span>: <span class="str">"same structure as deckA"</span>,
  <span class="str">"hardPoints"</span>: [
    { <span class="str">"pointId"</span>: <span class="str">"string"</span>, <span class="str">"roomInstanceId"</span>: <span class="str">"optional string (installed weapon)"</span> }
  ],
  <span class="str">"totalPowerConsumption"</span>: <span class="str">"int (server-calculated, read-only)"</span>,
  <span class="str">"combatPowerScore"</span>: <span class="str">"float (server-calculated, read-only)"</span>
}</code></pre>

  <h3>9.3 战斗状态快照（BattleState）</h3>
  <pre><code>{
  <span class="str">"battleId"</span>: <span class="str">"string"</span>,
  <span class="str">"attackerUserId"</span>: <span class="str">"string"</span>,
  <span class="str">"defenderUserId"</span>: <span class="str">"string"</span>,
  <span class="str">"startTimestamp"</span>: <span class="str">"long"</span>,
  <span class="str">"elapsedSeconds"</span>: <span class="str">"float"</span>,
  <span class="str">"phase"</span>: <span class="str">"PRE_BATTLE_SCAN | COMBAT_ACTIVE | RESULT_CALCULATION"</span>,
  <span class="str">"attackerShip"</span>: { <span class="str">"snapshot"</span>: <span class="str">"ShipData"</span>, <span class="str">"runtimeState"</span>: <span class="str">"ShipRuntimeState"</span> },
  <span class="str">"defenderShip"</span>: { <span class="str">"snapshot"</span>: <span class="str">"ShipData"</span>, <span class="str">"runtimeState"</span>: <span class="str">"ShipRuntimeState"</span> },
  <span class="str">"commandLog"</span>: <span class="str">"List&lt;CommandEvent&gt;"</span>
}

<span class="cm">// ShipRuntimeState：战斗中变化的动态状态（不持久化，只用于计算）</span>
{
  <span class="str">"roomHpMap"</span>: <span class="str">"Dict&lt;instanceId, int&gt;"</span>,
  <span class="str">"roomPowerMap"</span>: <span class="str">"Dict&lt;instanceId, int&gt;"</span>,
  <span class="str">"shieldHpMap"</span>: <span class="str">"Dict&lt;instanceId, int&gt;"</span>,
  <span class="str">"crewPositions"</span>: <span class="str">"Dict&lt;crewId, {deckType, x, y}>"</span>,
  <span class="str">"crewHpMap"</span>: <span class="str">"Dict&lt;crewId, int&gt;"</span>,
  <span class="str">"weaponReloadTimers"</span>: <span class="str">"Dict&lt;instanceId, float&gt;"</span>,
  <span class="str">"ionDisableTimers"</span>: <span class="str">"Dict&lt;instanceId, float&gt;"</span>
}</code></pre>

  <h3>9.4 Orleans Grain 键值对照</h3>
  <pre><code><span class="cm">// Orleans Grain 设计（服务端开发参考）</span>
<span class="kw">interface</span> <span class="ty">IPlayerGrain</span> : <span class="ty">IGrainWithStringKey</span>       <span class="cm">// Key: userId</span>
<span class="kw">interface</span> <span class="ty">IMatchmakingGrain</span> : <span class="ty">IGrainWithIntegerKey</span>   <span class="cm">// Key: shipLevel (1-5)</span>
<span class="kw">interface</span> <span class="ty">IBattleGrain</span> : <span class="ty">IGrainWithStringKey</span>         <span class="cm">// Key: battleId</span>
<span class="kw">interface</span> <span class="ty">ILeaderboardGrain</span> : <span class="ty">IGrainWithStringKey</span>    <span class="cm">// Key: "season_{seasonId}"</span>
<span class="kw">interface</span> <span class="ty">IFleetGrain</span> : <span class="ty">IGrainWithStringKey</span>          <span class="cm">// Key: fleetId</span>
<span class="kw">interface</span> <span class="ty">INotificationGrain</span> : <span class="ty">IGrainWithStringKey</span>   <span class="cm">// Key: userId</span></code></pre>
</section>

<footer class="footer">
  <span>IRON FLEET — 游戏策划文档 GDD v1.0</span>
  <span>AI可读格式 · Claude Code上下文优化</span>
  <span>2026-04</span>
</footer>

</div>
</body>
</html>


<!--

**Here are some ideas to get you started:**

🙋‍♀️ A short introduction - what is your organization all about?
🌈 Contribution guidelines - how can the community get involved?
👩‍💻 Useful resources - where can the community find your docs? Is there anything else the community should know?
🍿 Fun facts - what does your team eat for breakfast?
🧙 Remember, you can do mighty things with the power of [Markdown](https://docs.github.com/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
-->
