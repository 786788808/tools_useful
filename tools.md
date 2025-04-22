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
