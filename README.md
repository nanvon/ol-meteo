# ol-meteo

OpenLayers 仿 Windy 气象可视化。Fork from [ccmostmosthandsome/olMeteo](https://github.com/ccmostmosthandsome/olMeteo)

## 思路

1. 使用 Python 将气象格点数据转换成灰度图
   - 首先获取安徽省范围的四个顶点的经纬度，确定数据与渲染范围；
   - 从地图左上角开始，每隔 11km 取一个点，取完一行就换行，获取该点的气象数据；
   - 根据本项目的`[111.5, 29.3, 123.8, 34.8]`范围，可以得到 55\*29 的矩阵数据；
   - 使用 Python 对数据进行超分辨率处理，得到一个 440\*232 的矩阵数据
2. 前端处理矩阵数据，渲染上色
   - 将矩阵 txt 数据转成 Float32Array 数组；
   - 使用 OpenLayers 的 canvasFunction 方法，将视图上点的位置坐标转成经纬度；
   - 考虑到视图的范围`[111.5, 29.3, 123.8, 34.8]`，将经纬度转换成栅格坐标；
   - 获取周围四个点的坐标，利用双线性插值算法计算出当前经纬度的气象要素值；
   - 将灰度值映射到设定好的颜色数组中，并绘制到地图图层上

## 运行

```sh
pnpm install

pnpm dev
```

## 参考

- [ccmostmosthandsome/olMeteo](https://github.com/ccmostmosthandsome/olMeteo)
- [chroma.js](https://github.com/gka/chroma.js)
- [气象可视化之windy方案的技术分析](https://mp.weixin.qq.com/s/4oCJQU69tsdnPSSLQdiEMg)
- [格点数据应用之二 WebGIS等值面可视化篇](https://zhuanlan.zhihu.com/p/347293918)
- [GRIB学习笔记：GRIB2要素场转换为图片](https://blog.perillaroc.wang/post/2019/2019-05-18-grib-notebook-convert-grib2-to-png/)
