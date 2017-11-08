# data-visualization

<!DOCTYPE html>
<meta charset="utf-8">
<style>

body {
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
  position: relative;
  width: 960px;
  padding: 50px;
}


.axis text {
  font: 10px sans-serif;
}

.axis path,
.axis line {
  fill: none;
  stroke: #000;
  shape-rendering: crispEdges;
}


.x.axis path {
  display: none;
}

.sortbox {
  position: relative;
  top: 10px;
  right: 10px;
  padding: 10px;
}

</style>
<body>
<h2>10 sports ranked across 3 skill categories by ESPN panel</h2>
<p><a href="http://www.espn.com/espn/page2/sportSkills">ESPN</a>,Degree of Difficulty: Sport Rankings.</p>
    <div id="form">
      <label><input type="radio" name="mode" value = "power" checked>Power</label>
      <label><input type="radio" name="mode" value = "speed">Speed</label>
      <label><input type="radio" name="mode" value = "analytic">Analytic</label>
    </div>
<label class="sortbox"><input type="checkbox" id="sort">Sort values</label>
<script type="text/javascript" src="d3.min.js"></script>
<script>

//set up chart
var margin = {top: 50, right: 20, bottom: 30, left: 40},
    width = 960 - margin.left - margin.right,
    height = 500 - margin.top - margin.bottom;

var x = d3.scaleBand().rangeRound([0,width]).padding(0.1);

var y = d3.scaleLinear()
    .rangeRound([height, 0]);

var xAxis = d3.axisBottom(x);
var yAxis = d3.axisLeft(y);

var svg = d3.select("body").append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", 520)
  .append("g")
    .attr("transform", "translate(" + margin.left + "," + margin.top + ")");


  //Load Data
  d3.csv("toughestsport.csv", function(error, data) {
    data = data.slice(0,10);
    data.forEach(function(d) {
      d.analytic = +d.ANA;
      d.sport = d.SPORT;
      d.power = +d.PWR;
      d.speed = +d.SPD;
    });

    x.domain(data.map(function(d) { return d.sport; }));
    y.domain([0, d3.max(data, function(d) { return d.power; })]);

    //x axis and labels
    svg.append("g")
      .attr("class", "x_axis")
      .attr("transform", "translate(0," + height + ")")
      .call(xAxis)
      .selectAll("text")
        .attr("dy",".5em")
        .attr("transform","rotate(-30)")
        .style("text-anchor","end");

    //y axis and labels
    svg.append("g")
      .attr("class", "y_axis")
      .call(yAxis);
    svg.append("text")
      .attr("class","y_label")
      .attr("transform", "rotate(-90)")
      .attr("y", 0 - margin.left)
      .attr("x",0 - (height/2))
      .attr("dy", ".71em")
      .style("text-anchor", "middle")
      .text("Difficulty Score");

    //Create Barchart 
    svg.selectAll('.bar')
      .data(data)
    .enter().append("rect")
      .attr("class", "bar")
      .attr("x", function(d) { return x(d.sport); })
      .attr("width", x.bandwidth())
      .attr("y", function(d) { return y(d.power); })
      .attr("height", function(d) { return height - y(d.power);})
      .attr("fill","red")
      .attr("fill-opacity",.9); 

    d3.selectAll("input:not(#sort)").on("change", handleFormClick);
    var value = "power";
    var dat = makeData(value,data);
    function handleFormClick() {
      d3.select("#sort").property("checked", false);
      if (this.value === "power") {
        transitionPower();
      } else if(this.value === "speed"){
        transitionSpeed();
      } else{
        transitionAnalytic();
      }
    }
    function transitionPower(){
      value = "power";
      dat = makeData(value,data);
      transitionRects(dat,value);
    }
    function transitionSpeed(){
      value = "speed";
      dat = makeData(value,data);
      transitionRects(dat,value);
    }
    function transitionAnalytic(){
      value = "analytic";
      dat = makeData(value,data);
      transitionRects(dat,value);
    }

    function makeData(val,data){
      if (val === "power"){
        return data.map(function(d){
          return {skill:d.power,sport:d.sport}
        })
      } else if (val === "speed"){
          return data.map(function(d){
            return {skill:d.speed,sport:d.sport}
        })
      } else {
          return data.map(function(d){
            return {skill:d.analytic,sport:d.sport}
        })
      }
    }

    function transitionRects(skill,val){
      //set domain
      x.domain(skill.map(function(d) { return d.sport; }));
      y.domain([0, d3.max(skill, function(d) { return d.skill })]);
      //remove old data 
      var bars = svg.selectAll(".bar").remove().exit().data(skill);
      //enter new data, change bars
      bars
      .enter().append("rect")
      .attr("class", "bar")
      .attr("x", function(d) { return x(d.sport); })
      .attr("width", x.bandwidth())
      .attr("y", function(d) { return y(d.skill); })
      .attr("height", function(d) { return height - y(d.skill);})
      .attr("fill",  function(){  
        if(val === "power"){
          return "red";
        } else if (val === "speed"){
          return "orange";
        } else {
          return "steelblue";
        }})
      .attr("fill-opacity",.9); 
   }

   //Perform Sorting
    d3.select("#sort").on("change", change);
    var sortTimeout = setTimeout(function() {
      d3.select("#sort").property("checked", true).each(change);
    }, 2000);
   function change() {
      clearTimeout(sortTimeout);
      var x0 = x.domain(dat.sort(this.checked 
        ? function(a, b) { return b.skill - a.skill;}
        : function(a, b) { return d3.ascending(a.sport,b.sport);})
        .map(function(d) { return d.sport;}))
        .copy();

      //sorting bars
      svg.selectAll(".bar")
          .sort(function(a, b) { return x0(a.sport) - x0(b.sport);});

      var transition = svg.transition().duration(750),
          delay = function(d, i) { return i * 50; };

      transition.selectAll(".bar")
          .delay(delay)
          .attr("x", function(d) { return x0(d.sport); });

      transition.select(".x_axis")
          .call(xAxis)
        .selectAll("g")
          .delay(delay);
  }

});
</script>
</body>
