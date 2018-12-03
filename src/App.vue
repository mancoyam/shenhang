<template>
  <div id="app">
    <div id="backup"></div>
    <div class="head">
        <img class="logo" src="@/assets/img/logo.png" />
        <span class="title">全渠道销售数据</span>
        <p class="time">{{time|moment}} <span>星期{{day}}</span></p>
    </div>
    <div class="ticket-left">
        <p class="ticket-title">机票总销售金额：</p>
        <p class="ticket-content"><span>{{totalAmonut}}</span>元</p>
    </div>
    <div class="ticket-right">
        <p class="ticket-title">机票总订单量：</p>
        <p class="ticket-content"><span>{{totalNum}}</span>单</p>
    </div>
    <div class="pie-box">
        <div class="chart-title-box">
            <p class="chart-title">订单占比</p>
        </div>
        <div class="pie" id="pie"></div>
    </div>
    <div class="bottom">
        <div class="bottom-left">
            <div class="chart-title-box">
                <p class="chart-title">机票销售数据的来源渠道</p>
            </div>
            <div class="data-origin">
                <div class="data-origin-left">
                    <div class="data-item data-head">
                        <div class="num">排名</div>
                        <div class="origin">渠道</div>
                        <div class="extra">金额</div>
                        <div class="order-num">订单量</div>
                    </div>
                    <div class="data-item" v-for="item in leftData" :key="item.num">
                        <div class="num">{{item.num}}</div>
                        <div class="origin">{{item.origin}}</div>
                        <div class="extra">￥{{item.extra}}</div>
                        <div class="order-num">{{item.order}}</div>
                    </div>
                </div>
                <div class="data-origin-right">
                    <div class="data-item data-head">
                        <div class="num">排名</div>
                        <div class="origin">渠道</div>
                        <div class="extra">金额</div>
                        <div class="order-num">订单量</div>
                    </div>
                    <div class="data-item" v-for="item in rightData" :key="item.num">
                        <div class="num">{{item.num}}</div>
                        <div class="origin">{{item.origin}}</div>
                        <div class="extra">￥{{item.extra}}</div>
                        <div class="order-num">{{item.order}}</div>
                    </div>
                </div>
            </div>
        </div>
        <div class="bottom-center">
            <div class="chart-title-box">
                <p class="chart-title">当日机票订单量</p>
            </div>
            <div class="line" id="line"></div>
        </div>
        <div class="bottom-right">
            <div class="chart-title-box">
                <p class="chart-title">订单</p>
            </div>
            <div class="bar" id="bar"></div>
        </div>
    </div>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld'
//引入echarts
import echarts from 'echarts'
import 'echarts-gl'

export default {
  name: 'App',
  data() {
    return {
        time: new Date(), //当前时间
        day: null, //星期的某一天的数字
        leftData: [{num: '1', origin: 'B2B', extra: '50045986', order: '5168728'}, {num: '2', origin: '工贸', extra: '8328469', order: '168728'}, {num: '3', origin: '电商', extra: '650085', order: '68728'}], //销售渠道前三名
        rightData: [{num: '4', origin: 'SME', extra: '904538', order: '63128'}, {num: '5', origin: '呼叫中心', extra: '89045', order: '8728'}, {num: '6', origin: '其他', extra: '1045983', order: '8583'}], //销售渠道四-六名
        totalAmonut: null, //总销售金额
        totalNum: null //总订单量
    }
  },
  mounted() {
    const weekday = new Array(7);
    weekday[0] = '日';
    weekday[1] = '一';
    weekday[2] = '二';
    weekday[3] = '三';
    weekday[4] = '四';
    weekday[5] = '五';
    weekday[6] = '六';
    let day = new Date().getDay();
    this.day = weekday[day];
    this.totalAmonut = 7654321999
    this.totalNum = 19850962
    var backup = echarts.init(document.getElementById('backup'));
    var pie = echarts.init(document.getElementById('pie'));
    var line = echarts.init(document.getElementById('line'));
    var bar = echarts.init(document.getElementById('bar'));
    //设置航线图参数
    this.$http.get('./static/flights.json').then(res => {
      var data = res.data;
      var airports = data.airports.map(function (item) {
          return {
              coord: [item[3], item[4]]
          }
      });
      function getAirportCoord(idx) {
          return [data.airports[idx][3], data.airports[idx][4]];
      }

      // Route: [airlineIndex, sourceAirportIndex, destinationAirportIndex]
      var routesGroupByAirline = {};
      data.routes.forEach(function (route) {
          var airline = data.airlines[route[0]];
          var airlineName = airline[0];
          if (!routesGroupByAirline[airlineName]) {
              routesGroupByAirline[airlineName] = [];
          }
          routesGroupByAirline[airlineName].push(route);
      });

      var pointsData = [];
      data.routes.forEach(function (airline) {
          pointsData.push(getAirportCoord(airline[1]));
          pointsData.push(getAirportCoord(airline[2]));
      });

      var series = data.airlines.map(function (airline) {
          var airlineName = airline[0];
          var routes = routesGroupByAirline[airlineName];

          if (!routes) {
              return null;
          }
          return {
              type: 'lines3D',
              name: airlineName,

              effect: {
                  show: true,
                  trailWidth: 2,
                  trailLength: 0.15,
                  trailOpacity: 1,
                  trailColor: 'rgb(30, 30, 60)'
              },

              lineStyle: {
                  width: 1,
                  color: 'rgb(50, 50, 150)',
                  // color: 'rgb(118, 233, 241)',
                  opacity: 0.1
              },
              blendMode: 'lighter',

              data: routes.map(function (item) {
                  return [airports[item[1]].coord, airports[item[2]].coord];
              })
          };
      }).filter(function (series) {
          return !!series;
      });
      series.push({
          type: 'scatter3D',
          coordinateSystem: 'globe',
          blendMode: 'lighter',
          symbolSize: 2,
          itemStyle: {
              color: 'rgb(50, 50, 150)',
              opacity: 0.2
          },
          data: pointsData
      });

      backup.setOption({
          legend: {
              selectedMode: 'single',
              left: 'center',
              top: 500,
              data: Object.keys(routesGroupByAirline),
              orient: 'horizontal',
              textStyle: {
                  color: '#fff'
              }
          },
          globe: {

              environment: './static/img/starfield.jpg',

              heightTexture: './static/img/bathymetry_bw_composite_4k.jpg',

              displacementScale: 0.1,
              displacementQuality: 'high',

              baseColor: '#000',

              shading: 'realistic',
              realisticMaterial: {
                  roughness: 0.2,
                  metalness: 0
              },

              postEffect: {
                  enable: true,
                  depthOfField: {
                      enable: false,
                      focalDistance: 150
                  }
              },
              temporalSuperSampling: {
                  enable: true
              },
              light: {
                  ambient: {
                      intensity: 0
                  },
                  main: {
                      intensity: 0.1,
                      shadow: false
                  },
                  ambientCubemap: {
                      texture: './static/lake.hdr',
                      exposure: 1,
                      diffuseIntensity: 0.5,
                      specularIntensity: 2
                  }
              },
              viewControl: {
                  autoRotate: false,
                  distance: 300
              },
              silent: true
          },
          series: series
      });
      window.addEventListener('keydown', function () {
          series.forEach(function (series, idx) {
              backup.dispatchAction({
                  type: 'lines3DToggleEffect',
                  seriesIndex: idx
              });
          })
      });
    }).catch(err => {
      console.log(err)
    });
    //设置饼图数据
    let pieOption = {
        series : [
            {
                name: '订单',
                type: 'pie',
                radius : '55%',
                center: ['50%', '50%'],
                hoverAnimation: false,
                data: [{
                    // 数据项的名称
                    name: 'B2B ' + Math.round(24*100/82) + '%',
                    // 数据项值
                    value: 24
                }, {
                    name: '工贸 ' + Math.round(14*100/82) + '%',
                    value: 14
                }, {
                    name: '电商 ' + Math.round(11*100/82) + '%',
                    value: 11
                }, {
                    name: 'SME ' + Math.round(5*100/82) + '%',
                    value: 5
                }, {
                    name: '',
                    value: 5
                }, {
                    name: '呼叫中心 ' + Math.round(18*100/82) + '%',
                    value: 18
                }, {
                    name: '其他 ' + Math.round(10*100/82) + '%',
                    value: 10
                }],
                itemStyle: {
                    color: function(params) {
		    	        //自定义颜色
		    	        var colorList = ['#1e8dff', '#e5ae2f', '#f9743f', '#5822e8', '#55a9c3', '#f42469','#b58ec5'];
		    	        return colorList[params.dataIndex]
		    	      }
                }
            }
        ]
    };
    pie.setOption(pieOption);

    //设置柱状图数据
    let barOption = {
        grid: {
            left: 60,
            top: 20,
            right: 0,
            bottom: 0
        },
        xAxis: {
            type: 'value',
            boundaryGap: [0, 0.01],
            splitNumber: 1,
            show:false
        },
        yAxis: {
            type: 'category',
            data: ['其他','呼叫中心','SME','电商','工贸','B2B'],
            splitNumber: 6,
            axisTick:{
               show:false
            },
            axisLine:{
                show:false,         
            },  
            splitLine:{
                show:false,        
            },
            axisLabel:{
                color: '#fff'
            }
        },
        series: [
            {
                name: '2011年',
                type: 'bar',
                data: [145300, 428310, 568128, 895186, 1650289, 2160478],
                itemStyle: {
                    normal: {
                        color: function(params) {
                            //自定义颜色
                            var colorList = ['#1e8dff', '#e5ae2f', '#f9743f', '#5822e8', '#55a9c3', '#f42469','#b58ec5'];
                            return colorList[params.dataIndex]
                        },
                        barBorderRadius: 8,
                        label : {
                            show: true,
                            position: 'right',
                            formatter:function(params){
                               return params.value;
                            },
                            color: '#fff'
                       }
                    }
                },
                barWidth: 12
            }
        ]
    };
    bar.setOption(barOption);

    //设置折线图数据
    let lineOption = {
        grid: {
            left: 40,
            top: 20,
            right: 0,
            bottom: 20
        },
        xAxis: {
            type: 'category',
            data: ['1月', '2月', '3月', '4月', '5月', '6月', '7月'],
            axisLine: {
                lineStyle: {
                    color: '#fff'
                }
            }
        },
        yAxis: {
            type: 'value',
            splitNumber: 3,
            axisLine: {
                lineStyle: {
                    color: '#fff'
                }
            }
        },
        series: [{
            data: [820, 932, 901, 934, 1290, 1330, 1320],
            type: 'line',
            itemStyle: {
                color: '#53bef5'
            },
            lineStyle: {
                color: '#53bef5'
            }
        }]
    };
    line.setOption(lineOption);
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
html,body {
  width: 100%;
  min-width: 1200px;
  height: 100%;
  min-height: 800px;
  background: #000;
}
#app {
  position: relative;
  width: 100%;
  height: 100%;
  font-family: '黑体', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
#backup {
  width: 100%;
  height: 100%;
}
.head {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 82px;
    line-height: 82px;
    border-bottom: 1px solid #fff;
}
.logo {
    width: 215px;
    height: 68px;
    vertical-align: middle;
}
.title {
    font-size: 40px;
    color: #fff;
    vertical-align: middle;
}
.time {
    position: absolute;
    right: 24px;
    top: 20px;
    width: 376px;
    height: 30px;
    line-height: 30px;
    font-family: 'ldr';
    font-size: 36px;
    color: #fff;
}
.time span {
    vertical-align: top;
    font-size: 25px;
}
.ticket-left {
    position: absolute;
    left: 25px;
    top: 130px;
    width: 565px;
}
.ticket-right {
    position: absolute;
    right: 25px;
    top: 130px;
    width: 320px;
}
.ticket-title {
    font-size: 24px;
    color: #fff;
    text-align: left;
}
.ticket-content {
    font-size: 48px;
    color: #f7dc0f;
    text-align: left;
    letter-spacing: 5px;
}
.ticket-content span {
    font-family: 'ldr';
    font-size: 60px;
    vertical-align: top;
}
.pie-box {
    position: absolute;
    left: 25px;
    bottom: 300px;
    width: 24%;
    max-width: 320px;
    min-width: 280px;
    height: 240px;
    padding: 20px;
    background: rgba(6, 10, 34, 0.7);
    box-sizing: border-box;
}
.pie {
    width: 100%;
    height: 130px;
    margin-top: 24px;
}
.bottom {
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 270px;
    padding: 0 25px 30px;
    color: #fff;
    display: flex;
    justify-content: space-between;
    box-sizing: border-box;
}
.bottom-left {
    flex: 2;
    height: 100%;
    margin-right: 20px;
    padding: 30px 20px;
    background: rgba(6, 10, 34, 0.7);
    box-sizing: border-box;
}
.bottom-center {
    flex: 1;
    height: 100%;
    margin-right: 20px;
    padding: 30px 20px;
    background: rgba(6, 10, 34, 0.7);
    box-sizing: border-box;
}
.bottom-right {
    flex: 1;
    height: 100%;
    padding: 30px 20px 30px 10px;
    background: rgba(6, 10, 34, 0.7);
    box-sizing: border-box;
}
.chart-title-box {
    position: relative;
    width: 100%;
    height: 44px;
    border-bottom: 2px solid #135b94;
}
.chart-title {
    position: absolute;
    left: 0;
    top: 0;
    height: 42px;
    font-size: 28px;
    color: #fff;
    border-bottom: 6px solid #fff;
}
.line {
    width: 100%;
    height: 134px;
}
.bar {
    width: 100%;
    height: 134px;
}
.data-origin {
    width: 100%;
    height: 144px;
}
.data-origin-left, .data-origin-right {
    float: left;
    width: 49%;
    height: 100%;
}
.data-origin-left {
    margin-right: 2%;
}
.data-item {
    width: 100%;
    height: 36px;
    overflow: hidden;
}
.data-item div {
    float: left;
    text-align: left;
    font-size: 17px;
}
.num {
    width: 15%;
    height: 36px;
    line-height: 36px;
}
.origin {
    width: 15%;
    height: 36px;
    line-height: 36px;
}
.extra {
    width: 35%;
    height: 36px;
    line-height: 36px;
    color: #138e2e;
}
.order-num {
    width: 35%;
    height: 36px;
    line-height: 36px;
}
.data-item.data-head div {
    font-weight: bolder;
    font-size: 16px;
    color: #fff;
}
</style>
