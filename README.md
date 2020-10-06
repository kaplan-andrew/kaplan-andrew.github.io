# kaplan-andrew.github.io
Recreation of the John Snow cholera map

<!DOCTYPE html>
<html lang="en">
	<head>
        <meta charset = "UTF-8">
        <script src="https://d3js.org/d3.v3.min.js"></script>
        <script src="http://labratrevenge.com/d3-tip/javascripts/d3.tip.v0.6.3.js"></script>
        <title>Project 1</title>
        <style>
            .deathdates {
                fill:#d55e00;
                }
            
            .deathdates:hover {
                        fill: black ;
                        }
            
            .agecount {
                fill:#0072b2;
                }
            
            .agecount:hover {
                        fill: black ;
                        }
            
            .gencount {
                fill:#009e73;
                }
            
            .gencount:hover {
                        fill: black ;
                        }

            .d3-tip {
                line-height: 1;
                font-weight: bold;
                padding: 12px;
                background: rgba(0, 0, 0, 0.8);
                color: #fff;
                border-radius: 2px;
            }

            /* Creates a small triangle extender for the tooltip */
            .d3-tip:after {
                box-sizing: border-box;
                display: inline;
                font-size: 10px;
                width: 100%;
                line-height: 1;
                color: rgba(0, 0, 0, 0.8);
                content: "\25BC";
                position: absolute;
                text-align: center;
            }	

            /* Style northward tooltips differently */
            .d3-tip.n:after {
            margin: -1px 0 0 0;
            top: 100%;
            left: 0;
            }

        </style>
    </head>
	<body>
        <script type = "text/javascript">
            
            //create svg1 to append attributes to
            var margin = {top: 50, right: 50, bottom: 50, left: 50},
                width = 800 - margin.left - margin.right,
                height = 800 - margin.top - margin.bottom;

            var svg1 = d3.select("body")
                        .append("svg")
                        .attr("width", width + margin.left + margin.right)
                        .attr("height", height + margin.top + margin.bottom)
                        .attr("transform", "translate(" + margin.left + ")")
                        .style('background-color', 'white');
            
            //add streets
            var streets;    // declare global variable
            d3.json("/data/streets.json", function(json){
                streets = json; //copy data from json to streets
                console.log(json); // output json to console

                var lineFunction = d3.svg.line()
                                    .x(function(d, i) {return d.x*40})      
                                    .y(function(d, i) {return d.y*40})
                                    .interpolate("linear")

                svg1.selectAll(".line")     // I changed this from .link to .line
                                .data(streets)
                                .enter()
                                .append("path")
                                .attr("class", "link")
                                .style('stroke','grey')
                                .style("stroke-width", 2)
                                .style('fill', 'none')
                                .attr("d", lineFunction)
                        });


            var blueColor = d3.scale.linear()
                .domain([1,6])
                .range(["powderblue", "midnightblue"]);
            
            var redColor = d3.scale.linear()
                .domain([1,6])
                .range(["pink", "darkred"]);

            var tip0 = d3.tip()
                        .attr('class', 'd3-tip')
                        .offset([-10, 0])
                        .html(function(d) {
                            return "<strong>This victim was </strong><span style='color:red'>" + d.agetxt + "</span><strong> years old and </strong><span style='color:red'>" + d.gendertxt + "</span>";
                        })
                    
            svg1.call(tip0);

            //add deaths
           var deaths;
           d3.csv("/data/deaths_age_sex2.csv", function(csv){
                deaths = csv; //copy data from csv to deaths
                console.log(csv); // output csv to console
                svg1.selectAll(".deaths")
                            .data(deaths)
                            .enter()
                            .append("circle")
                            .attr('class', 'deaths')
                            .style('fill',function(d){
                                if (d.gender == 1) {return blueColor(d.age)}
                                else {return redColor(d.age)}
                            ;})
                            .attr('cx', function(d) { return (d.x)*40 })
                            .attr('cy', function(d) { return (d.y)*40 })
                            .attr('r', 3)
                            .style('stroke', "black")
                            .style('stroke-width', 1)
                            .on('mouseover', tip0.show)
                            .on('mouseout', tip0.hide);
                        });

            var tip = d3.tip()
                        .attr('class', 'd3-tip')
                        .offset([-10, 0])
                        .html(function(d) {
                            return "<strong>This is a pump </strong>";
                        })
                    
            svg1.call(tip);

            ///add pumps
           var pumps;
           d3.csv("/data/pumps.csv", function(csv){
                pumps = csv; //copy data from csv to deaths
                console.log(csv); // output csv to console
                svg1.selectAll(".pumps")
                            .data(pumps)
                            .enter()
                            .append("rect")
                            .attr('class', 'pumps')
                            .style('fill','#924900')
                            .attr('x', function(d) { return (d.x*40) })
                            .attr('y', function(d) { return (d.y*40) })
                            .attr('width', 10)
                            .attr('height', 10)
                            .style('stroke-width', 1)
                            .style('stroke', "orange")
                            .on('mouseover', tip.show)
                            .on('mouseout', tip.hide);
                
                svg1.append("text")
                    .text('John Snow Cholera Map')
                    .attr("text-anchor", "middle")
                    .attr("class", "graph-title")
                    .attr("font-size", "30px")
                    .attr("y", height - 100)
                    .attr("x", (width+100) / 1.5);
            
                        });
            

            ///create deaths by date graph
            var margin = {top: 50, right: 50, bottom: 50, left: 50},
                width = 700 - margin.left - margin.right,
                height = 300 - margin.top - margin.bottom;

                var svg2 = d3.select("body")
                            .append("svg")
                            .attr("width", width + margin.left + margin.right)
                            .attr("height", height + margin.top + margin.bottom)
                            .attr("transform", "translate(" + margin.left + ")")
                            .style('background-color', 'white');

            d3.csv("/data/deathdays.csv", function(csv)
            {
                deathdays = csv; //copy data from csv to deaths
                console.log(deathdays); // output csv to console

                deathdays.forEach(function(d) {
                        d.deaths = +d.deaths;
                    });
                var maxValue = d3.max(csv, function(d) { return d.deaths; });

                //CREATE SCALES
                var x = d3.scale.ordinal()
                                .domain(deathdays.map(function(d) { return d['date']; }))
                                .rangeRoundBands([0, width+100], .1);

                var y = d3.scale.linear()
                                .domain([0, maxValue])
                                .range([height, 0]);
                
                //CREATE AXIS
                var xAxis = d3.svg.axis()
                            .scale(x)
                            .orient("bottom");

                var yAxis = d3.svg.axis()
                    .scale(y)
                    .orient("right");
                
                //APPEND AXIS TO SVG
                svg2.append("g")
                    .attr("class", "x axis")
                    .style("font", "8px times")
                    .attr("transform", "translate(0," + (height+50) + ")")
                    .call(xAxis)
                    .selectAll("text")	
                        .style("text-anchor", "end")
                        .attr("dx", "-.8em")
                        .attr("dy", ".05em")
                        .attr("transform", function(d) {
                            return "rotate(-90)" 
                        });;

                svg2.append("g")
                    .attr("class", "y axis")
                    .style("font", "10px times")
                    //.attr("transform", "translate(0," + width + ")")
                    .attr("transform", "translate(594,50)")
                    .call(yAxis);
                
                var tip1 = d3.tip()
                        .attr('class', 'd3-tip')
                        .offset([-10, 0])
                        .html(function(d) {
                            return "<strong>Deaths on </strong>" + d.date + "<strong>: </strong><span style='color:red'>" + d.deaths + "</span>";
                        })
                    
                svg2.call(tip1);

                //Loop through all the dates
                var bar2 = svg2.selectAll(".deathdates")
                            .data(deathdays)
                            .enter()
                            .append("rect")
                            .attr('class', 'deathdates')
                            //.style('fill','#d55e00')
                            .attr("x", function(d) { return x(d['date'])+8; })
                            .attr("y", function(d) { return y(d['deaths'])+50; })
                            .attr("height", function(d) { return height - y(d['deaths']); })
                            .attr("width", x.rangeBand())
                            .on('mouseover', tip1.show)
                            .on('mouseout', tip1.hide);

                svg2.append("text")
                    .text('Deaths by Date')
                    .attr("text-anchor", "middle")
                    .attr("class", "graph-title")
                    .attr("y", height - 180)
                    .attr("x", (width+100) / 2.0);

            });

            ///create deaths by age graph
            var margin = {top: 50, right: 50, bottom: 50, left: 50},
                width = 600 - margin.left - margin.right,
                height = 300 - margin.top - margin.bottom;

            var svg3 = d3.select("body")
                        .append("svg")
                        .attr("width", width + margin.left + margin.right)
                        .attr("height", height + margin.top + margin.bottom)
                        .attr("transform", "translate(" + margin.left + ")")
                        .style('background-color', 'white');

            d3.csv("/data/count_by_age.csv", function(csv)
            {
                ageCount = csv; //copy data from csv to deaths
                console.log(ageCount); // output csv to console

                ageCount.forEach(function(d) {
                        d.age_count = +d.age_count;
                    });
                var maxValue = d3.max(csv, function(d) { return d.age_count; });
                
                //CREATE SCALES
                var x = d3.scale.ordinal()
                                .domain(ageCount.map(function(d) { return d['age']; }))
                                .rangeRoundBands([0, width], .1);

                var y = d3.scale.linear()
                                .domain([0, maxValue])
                                .range([height, 0]);
                
                //CREATE AXIS
                var xAxis = d3.svg.axis()
                            .scale(x)
                            .orient("bottom");

                var yAxis = d3.svg.axis()
                                .scale(y)
                                .orient("right");
                
                //APPEND AXIS TO SVG
                svg3.append("g")
                    .attr("class", "x axis")
                    .style("font", "14px times")
                    .attr("transform", "translate(0," + (height+50) + ")")
                    .call(xAxis);

                svg3.append("g")
                    .attr("class", "y axis")
                    .style("font", "10px times")
                    //.attr("transform", "translate(0," + width + ")")
                    .attr("transform", "translate(494,50)")
                    .call(yAxis);
                
                var tip2 = d3.tip()
                        .attr('class', 'd3-tip')
                        .offset([-10, 0])
                        .html(function(d) {
                            return "<strong>Deaths for age </strong>" + d.age + "<strong>: </strong><span style='color:red'>" + d.age_count + "</span>";
                        })
                    
                svg3.call(tip2);

                //Loop through all the dates
                var bar3 = svg3.selectAll(".agecount")
                            .data(ageCount)
                            .enter()
                            .append("rect")
                            .attr('class', 'agecount')
                            //.style('fill','#0072b2')
                            .attr("x", function(d) { return x(d['age']); })
                            .attr("y", function(d) { return y(d['age_count'])+50; })
                            .attr("width", x.rangeBand())
                            .attr("height", function(d) { return height - y(d['age_count']); })
                            .on('mouseover', tip2.show)
                            .on('mouseout', tip2.hide);
                
                svg3.append("text")
                    .text('Deaths by Age Group')
                    .attr("text-anchor", "middle")
                    .attr("class", "graph-title")
                    .attr("y", height-180)
                    .attr("x", width / 2.0);

            });


            ///create deathes by gender graph
            var margin = {top: 50, right: 50, bottom: 50, left: 50},
                width = 600 - margin.left - margin.right,
                height = 300 - margin.top - margin.bottom;

            var svg4 = d3.select("body")
                        .append("svg")
                        .attr("width", width + margin.left + margin.right)
                        .attr("height", height + margin.top + margin.bottom)
                        .attr("transform", "translate(" + margin.left + ")")
                        .style('background-color', 'white');

            d3.csv("/data/count_by_gen.csv", function(csv)
            {
                genCount = csv; //copy data from csv to deaths
                console.log(genCount); // output csv to console

                genCount.forEach(function(d) {
                        d.gen_count = +d.gen_count;
                    });
                
                var maxValue = d3.max(csv, function(d) { return d.gen_count; });
                console.log(maxValue)

                //CREATE SCALES
                var x = d3.scale.ordinal()
                                .domain(genCount.map(function(d) { return d['gender']; }))
                                .rangeRoundBands([0, width], .1);

                var y = d3.scale.linear()
                                .domain([0, maxValue])
                                .range([height, 0]);
                
                //CREATE AXIS
                var xAxis = d3.svg.axis()
                            .scale(x)
                            .orient("bottom");

                var yAxis = d3.svg.axis()
                            .scale(y)
                            .orient("left");
                
                //APPEND AXIS TO SVG
                svg4.append("g")
                    .attr("class", "x axis")
                    .style("font", "14px times")
                    .attr("transform", "translate(24," + (height+50) + ")")
                    .call(xAxis);

                svg4.append("g")
                    .attr("class", "y axis")
                    .style("font", "10px times")
                    .attr("transform", "translate(30,50)")
                    .call(yAxis);

                var tip3 = d3.tip()
                        .attr('class', 'd3-tip')
                        .offset([-10, 0])
                        .html(function(d) {
                            return "<strong>Deaths for </strong>" + d.gender + "<strong>: </strong><span style='color:red'>" + d.gen_count + "</span>";
                        })
                    
                svg4.call(tip3);

                //Loop through all the dates
                var bar4 = svg4.selectAll(".gencount")
                            .data(genCount)
                            .enter()
                            .append("rect")
                            .attr('class', 'gencount')
                            //.style('fill','#009e73')
                            .attr("x", function(d) { return x(d['gender'])+30; })
                            .attr("width", x.rangeBand())
                            .attr("y", function(d) { return y(d['gen_count'])+50; })
                            .attr("height", function(d) { return height - y(d['gen_count']); })
                            .on('mouseover', tip3.show)
                            .on('mouseout', tip3.hide);

                svg4.append("text")
                    .text('Deaths by Gender')
                    .attr("text-anchor", "middle")
                    .attr("class", "graph-title")
                    .attr("y", height-180)
                    .attr("x", width / 2.0);						
            });

            
  
        </script>
        <p>Youtube video tutorial: https://www.youtube.com/watch?v=zuNQ1FcXI8Y&ab_channel=Kapalappa</p>
        <p>Written description of the visual: </p>
    </body>
</html>
