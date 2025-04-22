#
## 键盘按键显示工具  
http://code52.org/carnac/  
https://github.com/Code52/carnac/releases/download/2.3.13/Setup.exe

## json可视化
https://json4u.cn/  
支持：  
- JSON 校验——有修改时会自动做校验。错误行会有标注，并提供充足的上下文帮助定位错误；
- JSON 格式化——拖拽文件或粘贴后会自动格式化，即使是无效的 JSON 数据也能格式化；
- JSON 最小化——去除所有的空白字符，将 JSON 数据压缩成一行；
- JSON 转义——对 JSON 数据进行转义，增加 \ 字符使其成为一个合法的字符串；
- JSON 反转义——对 JSON 数据进行反转义，删除 \ 字符使其成为合法的 JSON；
- JSON 排序——递归的对 key 做排序，但不会对数组做排序，排序前后的 JSON 在语义上是相等的。常用于排序后比较，让 diff 集中在一起，方便人眼查看；
- 显示 JSON 路径——鼠标点击任意处，会展示对应 token 的 JSON Pointer。和 minimap 搭配使用，能帮助快速理解 JSON 的结构；
- CSV 转 JSON——将 CSV 文件转换成 JSON，支持带表头和不带表头两种情况；
- JSON 转 CSV——将 JSON 数组转换成 CSV 文件；
- URL 转 JSON——递归的解析 URL 转成一个 JSON。如果需要对比两个 URL 的 diff，先转成 JSON 再使用 diff 就非常方便；
- BigInt/int64 比较——在业务开发中 ID 通常会定义为 int64，而 js 的 number 只能精确表达 52bit 整数，此类字段如果前后端不做特殊处理，会丢失精度。为了快速找出这类差异，所以支持了 Bigint 比较的功能；
- 逐词比较——有时候两个 JSON 数据只有很少的差别，比如：两个很长的字符串，但只有一两个词不一样，此时肉眼很难发现具体哪个地方不一致。所以支持了逐词比较，让你能一眼看出具体 diff；
- 数组差分比较——如果两个 JSON 数组有差异，一般情况下我们其实不需要查看每一个元素的 diff，因为很可能会有比较多的噪音，并不方便我们肉眼查看。所以做了差分比较，提供与 git diff 一样简洁、人类可读的差异；
- 文本比较——因为文本比较也是很常见的需求，所以支持了将无效 JSON 降级为文本比较（text diff），体验与 git diff 接近。


CREATE OR REPLACE PROCEDURE `your_project.your_dataset.generate_suspend_records`()
BEGIN
  -- 创建临时表存储中间结果
  CREATE TEMP TABLE temp_suspend_data AS
  WITH 
  -- 第一步：生成基础关联数据
  base_data AS (
    SELECT
      b.RIC_SOURCE,
      b.companyName,
      b.suspensionType,
      b.suspensionDate,
      b.resumptionDate,
      b.suspensionstarttime,
      b.resumptiontime,
      a.tradingDate,
      -- 计算总交易日数
      COUNT(*) OVER(PARTITION BY b.RIC_SOURCE) AS total_days,
      -- 标记是否是最后交易日
      ROW_NUMBER() OVER(PARTITION BY b.RIC_SOURCE ORDER BY a.tradingDate DESC) AS rn
    FROM 
      `your_project.your_dataset.b` b
    INNER JOIN 
      `your_project.your_dataset.a` a
    ON 
      a.tradingDate BETWEEN b.suspensionDate AND DATE_SUB(b.resumptionDate, INTERVAL 1 DAY)
  ),
  
  -- 第二步：处理日期合并逻辑
  processed_data AS (
    SELECT
      *,
      -- 判断是否需要合并最后两行
      CASE 
        WHEN total_days > 1 AND rn = total_days - 1 THEN 1
        ELSE 0
      END AS merge_flag,
      -- 生成合并日期（当需要合并时使用前一个交易日）
      IF(rn = total_days, 
         ARRAY_AGG(tradingDate ORDER BY tradingDate LIMIT 2)[SAFE_OFFSET(0)],
         tradingDate) AS display_date
    FROM 
      base_data
  )

  -- 第三步：生成最终输出
  INSERT INTO `your_project.your_dataset.c`
  SELECT
    RIC_SOURCE,
    companyName,
    suspensionType,
    CASE 
      WHEN total_days = 1 THEN suspensionDate
      WHEN merge_flag = 1 THEN display_date
      ELSE DATE_SUB(display_date, INTERVAL 1 DAY)
    END AS suspensionDate,
    CASE 
      WHEN total_days = 1 THEN resumptionDate
      WHEN merge_flag = 1 THEN NULL
      ELSE DATE_SUB(display_date, INTERVAL 1 DAY)
    END AS resumptionDate,
    CASE 
      WHEN rn = total_days THEN resumptiontime
      ELSE suspensionstarttime
    END AS suspensionstarttime,
    CASE 
      WHEN rn = total_days THEN NULL
      ELSE resumptiontime
    END AS resumptiontime,
    MAX(_timestamp) OVER(PARTITION BY RIC_SOURCE, display_date) AS _timestamp
  FROM 
    processed_data
  QUALIFY
    -- 合并行逻辑
    (rn = 1 OR NOT merge_flag)
  ORDER BY 
    RIC_SOURCE, display_date;

  -- 清理临时表
  DROP TABLE temp_suspend_data;
END;
