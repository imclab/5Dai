

<h1 align="center">Live. Open. Intellgence.</h1>
<h1 align="center">Actionable Insights</h>


<div align="center">
<a href="https://ossinsight.io">
  <img src="/web/static/img/screenshots/homepage.gif"
</a>



 <div id="container" style="height: 100%"></div>

  <script type="text/javascript" src="https://fastly.jsdelivr.net/npm/jquery@3.7.1/dist/jquery.min.js"></script>
  <script type="text/javascript" src="https://fastly.jsdelivr.net/npm/echarts@5.5.0/dist/echarts.min.js"></script>
  
  <!-- Uncomment this line if you want to dataTool extension
  <script type="text/javascript" src="https://fastly.jsdelivr.net/npm/echarts@5.5.0/dist/extension/dataTool.min.js"></script>
  -->
  <!-- Uncomment this line if you want to use gl extension
  <script type="text/javascript" src="https://fastly.jsdelivr.net/npm/echarts-gl@2/dist/echarts-gl.min.js"></script>
  -->
  <!-- Uncomment this line if you want to echarts-stat extension
  <script type="text/javascript" src="https://fastly.jsdelivr.net/npm/echarts-stat@latest/dist/ecStat.min.js"></script>
  -->
  <!-- Uncomment this line if you want to use map
  <script type="text/javascript" src="https://fastly.jsdelivr.net/npm/echarts@4.9.0/map/js/china.js"></script>
  <script type="text/javascript" src="https://fastly.jsdelivr.net/npm/echarts@4.9.0/map/js/world.js"></script>
  -->
  <!-- Uncomment these two lines if you want to use bmap extension
  <script type="text/javascript" src="https://api.map.baidu.com/api?v=3.0&ak=YOUR_API_KEY"></script>
  <script type="text/javascript" src="https://fastly.jsdelivr.net/npm/echarts@5.5.0/dist/extension/bmap.min.js"></script>
  -->

  <script type="text/javascript">
    var dom = document.getElementById('container');
    var myChart = echarts.init(dom, null, {
      renderer: 'canvas',
      useDirtyRect: false
    });
    var app = {};
    var ROOT_PATH = 'https://echarts.apache.org/examples';
    var option;

    var config = {
  color: '#c0101a',
  levels: 1,
  intensity: 4,
  threshold: 0.01
};
var canvas = document.createElement('canvas');
canvas.width = 4096;
canvas.height = 2048;
context = canvas.getContext('2d');
context.lineWidth = 0.5;
context.strokeStyle = config.color;
context.fillStyle = config.color;
context.shadowColor = config.color;
$.when(
  $.getScript(
    'https://fastly.jsdelivr.net/npm/d3-array@2.8.0/dist/d3-array.js'
  ),
  $.getScript(
    'https://fastly.jsdelivr.net/npm/d3-contour@2.0.0/dist/d3-contour.js'
  ),
  $.getScript('https://fastly.jsdelivr.net/npm/d3-geo@2.0.1/dist/d3-geo.js'),
  $.getScript('https://fastly.jsdelivr.net/npm/d3-timer@2.0.0/dist/d3-timer.js')
).done(function () {
  image(ROOT_PATH + '/data-gl/asset/bathymetry_bw_composite_4k.jpg').then(
    function (image) {
      var m = image.height,
        n = image.width,
        values = new Array(n * m),
        contours = d3.contours().size([n, m]).smooth(true),
        projection = d3.geoIdentity().scale(canvas.width / n),
        path = d3.geoPath(projection, context);
      //   StackBlur.R(image, 5);
      for (var j = 0, k = 0; j < m; ++j) {
        for (var i = 0; i < n; ++i, ++k) {
          values[k] = image.data[k << 2] / 255;
        }
      }
      var opt = {
        image: canvas
      };
      var results = [];
      function update(threshold, levels) {
        context.clearRect(0, 0, canvas.width, canvas.height);
        var thresholds = [];
        for (var i = 0; i < levels; i++) {
          thresholds.push((threshold + (1 / levels) * i) % 1);
        }
        results = contours.thresholds(thresholds)(values);
        redraw();
      }
      function redraw() {
        results.forEach(function (d, idx) {
          context.beginPath();
          path(d);
          context.globalAlpha = 1;
          context.stroke();
          if (idx > (config.levels / 5) * 3) {
            context.globalAlpha = 0.01;
            context.fill();
          }
        });
        opt.onupdate();
      }
      d3.timer(function (t) {
        var threshold = (t % 10000) / 10000;
        update(threshold, 1);
      });
      initCharts(opt);
      update(config.threshold, config.levels);
    }
  );
  function image(url) {
    return new Promise(function (resolve) {
      var image = new Image();
      image.src = url;
      image.crossOrigin = 'Anonymous';
      image.onload = function () {
        var canvas = document.createElement('canvas');
        canvas.width = image.width / 8;
        canvas.height = image.height / 8;
        var context = canvas.getContext('2d');
        context.drawImage(image, 0, 0, canvas.width, canvas.height);
        resolve(context.getImageData(0, 0, canvas.width, canvas.height));
      };
    });
  }
  function initCharts(opt) {
    var contourChart = echarts.init(document.createElement('canvas'), null, {
      width: 4096,
      height: 2048
    });
    var img = new echarts.graphic.Image({
      style: {
        image: opt.image,
        x: -1,
        y: -1,
        width: opt.image.width + 2,
        height: opt.image.height + 2
      }
    });
    contourChart.getZr().add(img);
    opt.onupdate = function () {
      img.dirty();
    };
    myChart.setOption({
      backgroundColor: '#000',
      globe: {
        environment: ROOT_PATH + '/data-gl/asset/starfield.jpg',
        heightTexture:
          ROOT_PATH + '/data-gl/asset/bathymetry_bw_composite_4k.jpg',
        displacementScale: 0.05,
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
            // enable: true
          }
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
            texture: ROOT_PATH + '/data-gl/asset/lake.hdr',
            exposure: 1,
            diffuseIntensity: 0.5,
            specularIntensity: 2
          }
        },
        viewControl: {
          autoRotate: false
        },
        layers: [
          {
            type: 'blend',
            blendTo: 'emission',
            texture: contourChart,
            intensity: config.intensity
          }
        ]
      }
    });
  }
});

    if (option && typeof option === 'object') {
      myChart.setOption(option);
    }

    window.addEventListener('resize', myChart.resize);
  </script>










<a href="https://next.ossinsight.io/widgets/official/compose-user-dashboard-stats?user_id=391299" target="_blank" style="display: block" align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://next.ossinsight.io/widgets/official/compose-user-dashboard-stats/thumbnail.png?user_id=391299&image_size=auto&color_scheme=dark" width="771" height="auto">
    <img alt="Dashboard stats of @JT5D" src="https://next.ossinsight.io/widgets/official/compose-user-dashboard-stats/thumbnail.png?user_id=391299&image_size=auto&color_scheme=light" width="771" height="auto">
  </picture>
</a>
</div>

<h4 align="center">
  <b><a href="https://ossinsight.io/explore/">Data Explorer</a></b>
  •
  <b><a href="https://ossinsight.io/collections/open-source-database">Repo Rankings</a></b>
  •
  <b><a href="https://ossinsight.io/analyze/Ovilia">Developer Analytics</a></b>
  •
  <a href="https://ossinsight.io/analyze/pingcap/tidb">Repo Analytics</a>
  •
  <a href="https://ossinsight.io/collections/open-source-database">Collections</a>
  •
  <a href="https://ossinsight.io/docs/workshop">Workshop</a>
  •
  <a href="https://ossinsight.io/blog">Blog</a>
  •
  <a href="https://ossinsight.io/docs">API</a>
  •
  <a href="https://twitter.com/OSSInsight">Twitter</a>
</h3>

## Introduction

OSS Insight is a powerful tool that provides comprehensive, valuable, and trending insights into the open source world by analyzing 5+ billion rows of GitHub events data.

OSS Insight's <a href="https://ossinsight.io/explore/">Data Explorer</a> provides a new way to explore GitHub data. Simply ask your question in natural language and Data Explorer will generate SQL, query the data, and present the results visually.

OSS Insight also provides in-depth analysis of individual GitHub repositories and developers, as well as the ability to compare two repositories using the same metrics.
  
[🎦 Video - OSS Insight: Easiest New Way to Analyze Open Source Software](https://www.youtube.com/watch?v=6ofDBgXh4So&t=1s)

### **Feature 0: Shareable Insight Widgets**

| Repository Activity Trends | Collaborative Productivity - Last 28 days |
| ----------- | ----------- |
|<img src="https://next.ossinsight.io/widgets/official/compose-activity-trends/thumbnail.png?repo_id=41986369&image_size=auto" />|<img src="https://next.ossinsight.io/widgets/official/compose-last-28-days-collaborative-productivity/thumbnail.png?repo_id=41986369&image_size=auto" />|

| Repository Performance Stats - Last 28 days | Active Contributors - Last 28 days |
| ----------- | ----------- |
|<img src="https://next.ossinsight.io/widgets/official/compose-last-28-days-stats/thumbnail.png?repo_id=41986369&image_size=auto" />|<img src="https://next.ossinsight.io/widgets/official/compose-recent-active-contributors/thumbnail.png?repo_id=41986369&limit=100&image_size=auto"/>|

| Star Geographic Distribution | Star History |
| ----------- | ----------- |
|<img src="https://next.ossinsight.io/widgets/official/analyze-repo-stars-map/thumbnail.png?activity=stars&repo_id=41986369&image_size=auto" />|<img src="https://next.ossinsight.io/widgets/official/analyze-repo-stars-history/thumbnail.png?repo_id=41986369&image_size=auto" />|

For more charming widgets, please [Check it out 👉](https://next.ossinsight.io/widgets?utm_source=github&utm_medium=referral)

### **Feature 1: GPT-Powered Data Exploration**
  
Data Explorer provides a new way to discover trends and insights into 5+ billion rows of GitHub data.

Simply ask your question in natural language and Data Explorer will generate SQL, query the data, and present the results visually. It's built with <a href="https://tidbcloud.com/channel?utm_source=ossinsight&utm_medium=referral&utm_campaign=chat2query_202301&utm_content=github_readme">Chat2Query</a>, a GPT-powered SQL generator in TiDB Cloud. 

Examples:

* [Projects similar to @facebook/react](https://ossinsight.io/explore?id=ba186a53-b2ab-4cad-a46f-e2c36566cacd)
* [The most interesting Web3 projects](https://ossinsight.io/explore?id=f829026d-491c-44e0-937a-287f97a3cba7)
* [Where are @kubernetes/kubernetes contributors from?](https://ossinsight.io/explore?id=754a681e-913f-4333-b55d-dbd8598bd84d)
* [More popular questions](https://ossinsight.io/explore/)

[🎦 Video - Data Explorer: Discover insights in GitHub data with GPT-generated SQL](https://www.youtube.com/watch?v=rZZfgOJ-quI)

### **Feature 2: Technical Fields Analytics 👁️**

* **GitHub Collections Analysis**
  
  Find insights about the monthly or historical rankings and trends in technical fields with curated repository lists.

<div align="center">
  <a href="https://en.pingcap.com/tidb-cloud/?utm_source=ossinsight&utm_medium=referral">
     <img src="/web/static/img/screenshots/homepage-collection.png" alt="GitHub Collections Analytics" height="500" />
  </a>
</div>


  **Examples**:
  
  * [Collection: Web Framework](https://ossinsight.io/collections/web-framework)
  * [Collection: Artificial Intelligence](https://ossinsight.io/collections/artificial-intelligence)
  * [Collection: Web3](https://ossinsight.io/collections/web3)
  * [More](https://ossinsight.io/collections/open-source-database) ...

  **Welcome to add collections**

  👏 We welcome your contributions here! You can add a collection on our website by submitting PRs. Please create a `.yml` file under [the collections file path]( https://github.com/pingcap/ossinsight/tree/main/etl/meta/collections).

  [Here](https://github.com/pingcap/ossinsight/blob/main/CONTRIBUTING.md#add-a-collection) is a file template that describes what you need to include. We look forward to your PRs！

* **Deep Insight into some popular fields of technology**
  
  Share with you many deep insights into some popular fields of technology, such as open source Databases, JavaScript Framework, Low-code Development Tools and so on.

   **Examples**:
  * [Deep Insight Into Open Source Databases](https://ossinsight.io/blog/deep-insight-into-open-source-databases)
  * [JavaScript Framework Repos Landscape 2021](https://ossinsight.io/blog/deep-insight-into-js-framework-2021)
  * [Web Framework Repos Landscape 2021](https://ossinsight.io/blog/deep-insight-into-web-framework-2021)
  * [More](https://ossinsight.io/blog) ...
  
  We’ll also share the SQL commands that generate all these analytical results above each chart, so you can use them on your own on TiDB Cloud following this [10-minute tutorial](https://ossinsight.io/blog/try-it-yourself/).

### **Feature 3: Developer Analytics**

Insights about **developer productivity**, **work cadence**, and **collaboration** from developers' contribution behavior.

* **Basic**:
  * Stars, behavior, most used languages，and contribution trends
  * Code (commits, pull requests, pull request size and code line changes), code reviews, and issues
* **Advanced**:
  * Contribution time distribution for all kind of contribution activities
  * Monthly stats about contribution activities in all public repositories

<div align="center">
    <img src="/web/static/img/screenshots/homepage-developer.png" alt="Developer Analytics" height="500" />
</div>

### **Feature 4: Repository Analytics**

Insights about the **code update frequency & degree of popularity** from repositories' status.

* **Basic**:
  * Stars, forks, issues, commits, pull requests, contributors, programming languages, and lines of code modified
  * Historical trends of these metrics
  * Time cost of issues, pull requests
* **Advanced**:
  * Geographical distribution of stargazers, issue creators, and pull request creators
  * Company distribution of stargazers, issue creators, and pull request creators

<div align="center">
    <img src="/web/static/img/screenshots/homepage-repository.png" alt="Repository Analytics" height="500" />
</div>

**Examples**:
* [React](https://ossinsight.io/analyze/facebook/react)
* [TiDB](https://ossinsight.io/analyze/pingcap/tidb)
* [web3.js](https://ossinsight.io/analyze/web3/web3.js)
* [Ant Design](https://ossinsight.io/analyze/ant-design/ant-design)
* [Chaos Mesh](https://ossinsight.io/analyze/chaos-mesh/chaos-mesh)

### **Feature 5: Compare Projects 🔨**

Compare two projects using the repo metrics mentioned in **Repository Analytics**.

**Examples**:
* [Compare Vue and React](https://ossinsight.io/analyze/vuejs/vue?vs=facebook/react)
* [Compare CockroachDB and TiDB](https://ossinsight.io/analyze/pingcap/tidb?vs=cockroachdb/cockroach)
* [Compare PyTorch and TensorFlow](https://ossinsight.io/analyze/pytorch/pytorch?vs=tensorflow/tensorflow)
  
## Contribution

We've released OSS Insight because it can do more insights about GitHub.We hope that others can benefit from the project. You are more than welcome to participate in capacity building. We are thankful for any [contributions](https://github.com/pingcap/ossinsight/blob/main/CONTRIBUTING.md) from the community.

* [GitHub Discussion](https://github.com/pingcap/ossinsight/discussions). Best for: help with building, discussion about OSS Insight best practices.
* [GitHub Issues](https://github.com/pingcap/ossinsight/issues). Best for: bugs and errors you encounter using OSS Insight.
* [GitHub PR](https://github.com/pingcap/ossinsight/pulls). Best for: pull request the features you wish for OSS Insight.
  * [collection](https://github.com/pingcap/ossinsight/blob/main/CONTRIBUTING.md#add-a-collection) You can add a collection on our website.
  * [blog](https://github.com/pingcap/ossinsight/blob/main/CONTRIBUTING.md#add-a-blog) You are welcome to share blogs about using OSS Insight.
  * [fix](https://github.com/pingcap/ossinsight/blob/main/CONTRIBUTING.md#add-a-collection) You can make fixes to current issues.
  * [feat](https://github.com/pingcap/ossinsight/blob/main/CONTRIBUTING.md#feature-requests) You are welcome to contribute if you have new feature ideas.

## Contact

We have a few channels for contact:

* [GitHub Discussions](https://github.com/pingcap/ossinsight/discussions): You can ask a question or discuss here.
* [@OSS Insight](https://twitter.com/OSSInsight) on Twitter
* [mail](mailto:ossinsight@pingcap.com):If you want to analyze more, please [contact us](mailto:ossinsight@pingcap.com) ✉️

## Development

* [⚙️ Setup](https://github.com/pingcap/ossinsight/blob/main/CONTRIBUTING.md#development)
* [💡 How to build your own insight tool](https://ossinsight.io/docs/workshop/mini-ossinsight/introduction)

## Sponsors

<div align="center">
  <a href="https://en.pingcap.com/tidb-cloud/?utm_source=ossinsight&utm_medium=referral">
    <img src="/web/static/img/tidb-cloud-logo-w.png" height=50 />
  </a>
</div>
