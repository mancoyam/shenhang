<template>
  <div id="app">
    <div id="backup"></div>
    <div class="head">
        <img class="logo" src="@/assets/img/logo.png" />
        <span class="title">全渠道销售数据</span>
        <p class="time">{{time|moment}} <span>星期{{day}}</span></p>
    </div>
    .pie    
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
        day: null //星期的某一天的数字
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
    var backup = echarts.init(document.getElementById('backup'));
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
                  autoRotate: false
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
    })
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
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
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
    font-family: '黑体';
    font-size: 40px;
    color: #fff;
    vertical-align: middle;
}
.time {
    position: absolute;
    right: 24px;
    top: 20px;
    width: 365px;
    height: 30px;
    line-height: 30px;
    font-family: 'ldr';
    font-size: 36px;
    color: #fff;
}
.time span {
    font-family: '黑体';
    vertical-align: top;
    font-size: 25px;
}
.ticket_left {
    position: absolute;
    left: 25px;
    top: 130px;
    width: 565px;
}
.ticket_right {
    position: absolute;
    right: 25px;
    top: 130px;
    width: 475px;
}
.pie {
    position: absolute;
    left: 25px;
    bottpm: 300px;
    width: 320px;
    height: 240px;
    background: rgba(255, 255, 255, .5)
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
    justify-content: space-around;
    box-sizing: border-box;
}
.bottom-left {
    flex: 2;
    height: 100%;
    background: rgba(255, 255, 255, .5);
}
.bottom-center {
    flex: 1;
    height: 100%;
    background: rgba(255, 255, 255, .5);
}
.bottom-right {
    flex: 1;
    height: 100%;
    background: rgba(255, 255, 255, .5);
}
</style>
