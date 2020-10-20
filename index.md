<!DOCTYPE html>
<meta charset="utf-8">
<style>
#date-rect {
  position: fixed;
  top: 0;
  left: 50%;
  margin-top: 20px;

}
#date {
  font-size: 40px;
}

</style>

<!-- Load d3.js -->
<script src="https://d3js.org/d3.v4.js"></script>
<script src="//ajax.googleapis.com/ajax/libs/jquery/1.8.1/jquery.min.js"></script>

<!-- Create a div where the graph will take place -->
<div id="my_dataviz"></div>
<div id ="date-rect"><p id="date"></p></div>
<script>

// set the dimensions and margins of the graph
var margin = {top: 10, right: 30, bottom: 20, left: 40},
    width = window.innerWidth - margin.left - margin.right - 15,
    height = window.innerHeight - margin.top - margin.bottom - 25;

// append the svg object to the body of the page
var svg = d3.select("#my_dataviz")
  .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
  .append("g")
    .attr("transform",
          "translate(" + margin.left + "," + margin.top + ")");


// get the data
d3.csv("https://raw.githubusercontent.com/RebeccaFlach/Covid-deaths-animated/main/covid-deaths-reporting-d3-data.csv", function(data) {


  // List of subgroups, (column names). list of reporting dates
  var subgroups = data.columns.slice(1);


  // List of groups = row names. Date of death
  var groups = d3.map(data, function(d){return(d.date)}).keys();

  var parseDate = d3.timeParse("%Y-%m-%d"); //for making written date into js date object
  var formatDate = d3.timeFormat("%Y-%m-%d"); // for making date object into written date (for display)

//scales and adds x axis
  var x = d3.scaleTime()
    .domain(d3.extent(data, function(d) { return parseDate(d.date); }))
    .range([ 0, width]);
  svg.append("g")
    .attr("transform", "translate(0," + height + ")")
    .call(d3.axisBottom(x)
    .ticks(30));

  //scales and adds y axis
  var y = d3.scaleLinear()
    .domain([0, 120])
    .range([ height, 0 ]);
  svg.append("g")
    .call(d3.axisLeft(y))
    .attr("id", "left-axis");


  var stackedData = [];
  var stackedDataMain = d3.stack()
    .keys(subgroups.reverse())
    (data);
    stackedData.push(stackedDataMain[0]);

var graph = function(){
    svg.append("g")
        .selectAll("g")
        // Enter in the stack data = loop key per key = group per group
        .data(stackedData)
        .enter().append("g")
          .attr("fill", function(d) { return color(d.key); })
          .attr("stroke", "black")
          .attr("stroke-width", 1)
          .selectAll("rect")
          // enter a second time = loop subgroup per subgroup to add all rectangles
          .data(function(d) { return d; })
          .enter().append("rect")
            .attr("x", function(d) {return x(parseDate(d.data.date)); })
            .attr("y", function(d) {return y(d[1]); })
            .attr("height", function(d) {return y(d[0]) - y(d[1]); })
            .attr("width", (d, i) => {return x(parseDate(data[1].date)) -  x(parseDate(data[0].date))})
            .attr("class", "bar");

    }
    var colors = ["black", "#ffbf00"];
    var colorStorage = ["#780800", "#c70d00", "#fc1303", "#ff6a00"]

    var color = d3.scaleOrdinal()
    .domain(subgroups)
    .range(colors);

    var i = 1;

    graph();

    var delayGraph = function(){
      if(i > 5){
        colors.unshift("black")
      }
      else if (i > 1 && i <= 5){
        colors.splice(1, 0, colorStorage[5 - i])
      }
      color = d3.scaleOrdinal()
      .domain(subgroups)
      .range(colors);

      if(i < stackedDataMain.length){
        stackedData.push(stackedDataMain[i]);
        var displayDate = stackedDataMain[i].key;
        $("#date").text(displayDate.replace("/", "-"));

        graph();
        i += 1;

      }
      else {
        clearInterval(interval)
      }
    } ;

    var interval = setInterval(delayGraph, 500)

    var loopEntireThing = function(){
      svg.selectAll(".bar").remove();
      colors = ["black", "#ffbf00"]; //resets colors
      stackedData = []; //resets data array
      stackedData.push(stackedDataMain[0]);
      graph(); //graphs first data set
      i = 1;
      clearInterval(interval);
      interval = setInterval(delayGraph, 500);
    }

    //reset graph on doubleclick,
                svg
                  .append('rect')
                  .style("fill", "none")
                  .style("pointer-events", "all")
                  .attr('width', width)
                  .attr('height', height)
                  .on('dblclick', loopEntireThing)

});




</script>
