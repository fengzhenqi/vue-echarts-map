<template>
    <div class="area-map">
        <div id="my-map"></div>
    </div>
</template>

<script>

function filterMap(geoJson = {}, cityArr = [], name, cp) {
    let arr = [];
    let obj = {};
    cityArr.map(item1 => {
        geoJson.features.map((item, index) => {
            if(item1 === item.properties.name) {
                arr = arr.concat(item.geometry.coordinates[0][0]);
                obj = item;
                obj.properties.name = name || item.properties.name;
                obj.properties.cp = cp || item.properties.cp;
                geoJson.features.splice(index, 1);
            }
        });
    })
    arr = arr.map(item => JSON.stringify(item));
    let newArr = [...arr];
    arr.map(item => {
        let num = newArr.filter(item1 => item1 == item).length;
        num > 1 ? newArr = newArr.filter(item1 => item1 !== item) : '';
        
    })
    newArr = newArr.map(item => JSON.parse(item));
    obj.geometry.coordinates[0][0] = newArr;
    geoJson.features = geoJson.features.concat(obj);
    return {...geoJson};
}

import * as echarts from 'echarts';
import yel from '../../assets/yels.png';
import org from '../../assets/orgs.png';
import red from '../../assets/reds.png';
import XingTai from './xt.json';

let xt = filterMap(XingTai, ['桥东区', '桥西区'], '城区', [114.485687, 37.068009]);
let timeOut = null;

export default {
    props: {
        mapData: {
            type: Array,
            default: () => []
        }
    },
    data () {
        return {
            dataUrl: red,
            myMap: null,
            cityNum: 0,
            geolight: null,
            highlightArr: [
                ['城区', '隆尧县', '广宗县'],
                ['临城县', '任县', '南宫市'],
                ['内邱县', '宁晋县', '威县'],
                ['邢台县', '柏乡县', '清河县'],
                ['沙河市', '平乡县', '新河县'],
                ['巨鹿县', '南和县', '临西县']
            ],
            option: {}
        }
    },
    mounted () {
        this.myMap = echarts.init(document.getElementById('my-map'));
        this.init();
    },
    watch: {
        'mapData': {
            handler(newVal, oldVal) {
                clearTimeout(timeOut);
                this.init();
            },
            deep: true,
        }
    },
    methods: {
        init() {
            let _this = this;
            this.myMap.showLoading();
            echarts.registerMap('xt', xt);
            this.option = {
                aria: {
                    show: true
                },
                geo: {
                    map: 'xt',
                    show: true,
                    roam: true,
                    zoom: 1.2,
                    center: [114.812507, 37.299255],//当前视角的中心点
                    roam: true, //是否开启平游或缩放
                    scaleLimit: { //滚轮缩放的极限控制
                        min: 1.2,
                        max: 1.2
                    },
                    label: {
                        normal: {
                            show: false
                        },
                        emphasis: {
                            show: false,
                        }
                    },
                    itemStyle: {
                        normal: {
                            areaColor: '#3a7fd5',
                            borderColor: '#0a53e9', //线
                            // shadowColor: '#29CCFE', //外发光
                            shadowBlur: 30,
                            // borderWidth: 8,
                        },
                        emphasis: {
                            areaColor: '#0a2dae', //悬浮区背景
                        }
                    }
                },
                series: [
                    {
                        map: 'xt',
                        name: '邢台市',
                        type: 'map',
                        mapType: 'xt', // 自定义扩展图表类型
                        zoom: 1.2,
                        // aspectScale: 0.75,
                        itemStyle: {
                            normal: {
                                label: {
                                    show: true,//默认是否显示省份名称  
                                    textStyle: {
                                        color: '#29CDFF'
                                    }  
                                },
                                borderWidth: 1,//边框大小
                                borderColor: '#24B2E2',
                                // areaColor: '#00f',//背景颜色
                                areaColor: {
                                    type: 'linear',
                                    x: 0,
                                    y: 0,
                                    x2: 0,
                                    y2: 1,
                                    colorStops: [{
                                    offset: 0, color: '#0468A6' // 0% 处的颜色
                                    }, {
                                    offset: 1, color: '#044083' // 100% 处的颜色
                                    }],
                                    globalCoord: false // 缺省为 false
                                },
                            },
                            emphasis: {
                                label: {
                                    show: true,//选中状态是否显示省份名称
                                    textStyle: {
                                        color: '#29CDFF'
                                    }
                                },
                                areaColor: {
                                    type: 'linear',
                                    x: 0,
                                    y: 0,
                                    x2: 0,
                                    y2: 1,
                                    colorStops: [{
                                    offset: 0, color: '#1875B3' // 0% 处的颜色
                                    }, {
                                    offset: 1, color: '#3686B3' // 100% 处的颜色
                                    }],
                                    globalCoord: false // 缺省为 false
                                },
                            },
                        },
                    },
                    {
                        name: 'Top',
                        type: 'scatter',
                        coordinateSystem: 'geo',
                        symbol: (val, params) => {
                            // console.log(params);
                            let color = 'image://' + red;
                            this.highlightArr.map(item => {
                                if(item[0] == params.name) {
                                    color = 'image://' + red;
                                } else if(item[1] == params.name) {
                                    color = 'image://' + yel;
                                } else if(item[2] == params.name) {
                                    color = 'image://' + org;
                                }
                            });
                            return color;
                        },
                        silent: true,
                        symbolSize: [210, 110],
                        symbolOffset: [80, -70],
                        label: {
                            normal: {
                                show: false,
                            },
                            emphasis: {
                                show: true,
                                textStyle: {
                                    color: '#fff',
                                    fontSize: 9,
                                },
                                formatter: (params) => {
                                    let arr = [
                                        `{a| ${params.name}}\n{hr|}\n`,
                                        `{b| · 销售量：${params.value[2]['salNum'] || 0} 条}\n`,
                                        `{b| · 销售额：${params.value[2]['salAmt'] || 0} 元}\n`,
                                        `{b| · 省产烟销量：${params.value[2]['provInv'] || 0} 条}`
                                    ];
                                    return arr.join('');
                                },
                                offset: [20, 4],
                                borderRadius: 4,
                                width: 164,
                                height: 100,
                                rich: {
                                    a: {
                                        padding: [0, 0, 0, 10],
                                        fontFamily: 'SourceHanSansCN-Regular, SourceHanSansCN',
                                        borderRadius: [5, 0, 0, 0],
                                        width: 154,
                                        height: 21,
                                        fontWeight: 400,
                                        color: '#FFFFFF',
                                        fontSize: 12,
                                        lineHeight: 21,
                                        align: 'left'
                                    },
                                    b: {
                                        height: 21,
                                        fontFamily: 'SourceHanSansCN-Regular, SourceHanSansCN',
                                        fontWeight: 400,
                                        color: '#FFFFFF',
                                        fontSize: 12,
                                        lineHeight: 21,
                                        align: 'left'
                                    }
                                }
                            }
                        },
                        itemStyle: {
                            normal: {
                                opacity: 0
                            },
                            emphasis: {
                                opacity: 1
                            }
                        },
                        data: this.getCenter(xt, this.mapData),
                        showEffectOn: 'render',
                        rippleEffect: {
                            brushType: 'stroke'
                        },
                        hoverAnimation: true,
                        legendHoverLink: true,
                        zlevel: 1000
                    }
                ]
            }
            this.myMap.setOption(this.option);
            this.myMap.hideLoading();
            this.setBanner();
            this.myMap.on('mousemove', params => {
                clearTimeout(timeOut);
                this.setBanner(true);
                this.setBlack(this.geolight);
                this.setLight(params.name);
                this.geolight = params.name;
                // console.log(params);
            });
            this.myMap.on('mouseout', params => {
                clearTimeout(timeOut);
                this.setBlack(this.geolight);
                this.setBanner();
            });
            window.addEventListener('resize', ()=> {
                _this.myMap.resize();
            })
        },
        getCenter(city, valArr) {
            let arr = [];
            city.features.forEach((item, index) => {
                arr[index] = {
                    name: item.properties.name,
                    value: this.gcj02tobd09(item.properties.center[0], item.properties.center[1]).concat({})
                }
                valArr.forEach(ele => {
                    if(item.properties.name == ele.name) arr[index]['value'][2] = ele;
                })
            })
            return arr;
        },
        gcj02tobd09(lng, lat) {
            let x_PI = 3.14159265358979324 * 3000.0 / 180.0;
            let z = Math.sqrt(lng * lng + lat * lat) + 0.00002 * Math.sin(lat * x_PI);
            let theta = Math.atan2(lat, lng) + 0.000003 * Math.cos(lng * x_PI);
            let bd_lng = z * Math.cos(theta) + 0.0065;
            let bd_lat = z * Math.sin(theta) + 0.006;
            return [bd_lng, bd_lat]
        },
        setBanner(type = false, timer = 5000) {
            if(type && !timeOut) return;
            let len = this.highlightArr.length - 1;
            this.myMap.dispatchAction({
                type: 'downplay',
                seriesIndex: 0,
                name: this.highlightArr[this.cityNum - 1 < 0 ? len : this.cityNum - 1]
            })
            this.myMap.dispatchAction({
                type: 'downplay',
                seriesIndex: 1,
                name: this.highlightArr[this.cityNum - 1 < 0 ? len : this.cityNum - 1]
            })
            if(type) {
                clearTimeout(timeOut);
                timeOut = null;
                return;
            }
            this.myMap.dispatchAction({
                type: 'highlight',
                seriesIndex: 0,
                name: this.highlightArr[this.cityNum]
            })
            this.myMap.dispatchAction({
                type: 'highlight',
                seriesIndex: 1,
                name: this.highlightArr[this.cityNum]
            })
            this.highlightArr[this.cityNum].map(item => {
                this.myMap.dispatchAction({
                    type: 'showTip',
                    seriesIndex: 0,
                    name: item,
                })
            })
            this.cityNum++ >= len ? this.cityNum = 0 : '';
            timeOut = setTimeout(() => {
                this.setBanner(false ,timer);
            }, timer)
        },
        // 点击设置高亮
        setLight(name) {
            this.myMap.dispatchAction({
                type: 'highlight',
                seriesIndex: 1,
                name: name
            })
        },
        // 取消高亮
        setBlack(name) {
            this.myMap.dispatchAction({
                type: 'downplay',
                seriesIndex: 1,
                name: name
            })
        }
    },
};
</script>

<style lang="less" scoped>
.area-map {
    width: 100%;
    height: 100%;
    background: url('../../assets/map-bag@2x.png') center bottom no-repeat;
    background-size: 100% 27%;
}
#my-map {
    width: 100%;
    height: 100%;
    background: url('../../assets/mapbac.png') center center no-repeat;
    background-size: 90% 100%;
}
</style>