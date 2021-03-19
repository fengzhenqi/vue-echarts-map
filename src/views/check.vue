<template>
  <div class="about">
    <div class="area-map">
        <div id="my-map"></div>
    </div>
  </div>
</template>

<script>
import * as echarts from 'echarts';
import XingTai from './xt.json';
function filterMap(geoJson = {}, cityArr = []) {
    let arr = [];
    let geo = geoJson.features;
    cityArr.map(name => {
      let cityConfig = geo.filter(item => item.properties.name == name);
      arr = arr.concat(cityConfig)
    })
    geoJson.features = arr;
    return {...geoJson};
}
export default {
  mounted() {
    // console.log(XingTai);
    let geoJson = filterMap(XingTai, ['云南省', '贵州省', '广西壮族自治区', '广东省', '海南省']) 
  //   :onlyShowArea='["云南", "贵州", "广西", "广东", "海南"]'
    console.log(geoJson);
    let myChart = echarts.init(document.getElementById('my-map'));
    myChart.showLoading();
    myChart.hideLoading();

    echarts.registerMap('HK', geoJson);

    myChart.setOption({
        tooltip: {
            trigger: 'item',
            formatter: '{b}<br/>{c} (p / km2)'
        },
        toolbox: {
            show: true,
            orient: 'vertical',
            left: 'right',
            top: 'center',
            feature: {
                dataView: {readOnly: false},
                restore: {},
                saveAsImage: {}
            }
        },
        visualMap: {
            min: 800,
            max: 50000,
            text: ['High', 'Low'],
            realtime: false,
            calculable: true,
            inRange: {
                color: ['lightskyblue', 'yellow', 'orangered']
            }
        },
        series: [
            {
                name: '香港18区人口密度',
                type: 'map',
                mapType: 'HK', // 自定义扩展图表类型
                label: {
                    show: true
                },
                data: [
                    {name: '广东省', value: 20057.34},
                    {name: '海南省', value: 15477.48}
                ]
            }
        ]
    });
  
}
}
</script>

<style lang="less" scoped>
.area-map {
    width: 100%;
    height: 500px;
    // background: url('../cockpit/img/map-bag@2x.png') center bottom no-repeat;
    background-size: 100% 27%;
}
#my-map {
    width: 100%;
    height: 100%;
    // background: url('../cockpit/img/mapbac.png') center center no-repeat;
    background-size: 90% 100%;
}
</style>
