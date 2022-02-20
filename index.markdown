<html>
<head>
    <script type="text/javascript" src="https://unpkg.com/vis-network/standalone/umd/vis-network.min.js"></script>

    <style type="text/css">
.container-lg{
	max-width: 2000px
}
#mynetwork {
  width: 100%;
  height: 900px;
  border: 1px solid lightgray;
}
#loadingBar {
  position: absolute;
  top: 0px;
  left: 0px;
  width: 100%;
  height: 902px;
  background-color: rgba(200, 200, 200, 0.8);
  -webkit-transition: all 0.5s ease;
  -moz-transition: all 0.5s ease;
  -ms-transition: all 0.5s ease;
  -o-transition: all 0.5s ease;
  transition: all 0.5s ease;
  opacity: 1;
}
#wrapper {
  position: relative;
  width: 100%;
  height: 900px;
}

#text {
  position: absolute;
  top: 8px;
  left: 530px;
  width: 30px;
  height: 50px;
  margin: auto auto auto auto;
  font-size: 15px;
  color: #000000;
}

div.outerBorder {
  position: relative;
  top: 400px;
  width: 600px;
  height: 44px;
  margin: auto auto auto auto;
  border: 8px solid rgba(0, 0, 0, 0.1);
  background: rgb(252, 252, 252); /* Old browsers */
  background: -moz-linear-gradient(
    top,
    rgba(252, 252, 252, 1) 0%,
    rgba(237, 237, 237, 1) 100%
  ); /* FF3.6+ */
  background: -webkit-gradient(
    linear,
    left top,
    left bottom,
    color-stop(0%, rgba(252, 252, 252, 1)),
    color-stop(100%, rgba(237, 237, 237, 1))
  ); /* Chrome,Safari4+ */
  background: -webkit-linear-gradient(
    top,
    rgba(252, 252, 252, 1) 0%,
    rgba(237, 237, 237, 1) 100%
  ); /* Chrome10+,Safari5.1+ */
  background: -o-linear-gradient(
    top,
    rgba(252, 252, 252, 1) 0%,
    rgba(237, 237, 237, 1) 100%
  ); /* Opera 11.10+ */
  background: -ms-linear-gradient(
    top,
    rgba(252, 252, 252, 1) 0%,
    rgba(237, 237, 237, 1) 100%
  ); /* IE10+ */
  background: linear-gradient(
    to bottom,
    rgba(252, 252, 252, 1) 0%,
    rgba(237, 237, 237, 1) 100%
  ); /* W3C */
  filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#fcfcfc', endColorstr='#ededed',GradientType=0 ); /* IE6-9 */
  border-radius: 72px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.2);
}

#border {
  position: absolute;
  top: 10px;
  left: 10px;
  width: 500px;
  height: 23px;
  margin: auto auto auto auto;
  box-shadow: 0px 0px 4px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
}

#bar {
  position: absolute;
  top: 0px;
  left: 0px;
  width: 20px;
  height: 20px;
  margin: auto auto auto auto;
  border-radius: 11px;
  border: 2px solid rgba(30, 30, 30, 0.05);
  background: rgb(0, 173, 246); /* Old browsers */
  box-shadow: 2px 0px 4px rgba(0, 0, 0, 0.4);
}

    </style>
</head>
<body>

<div id="wrapper">
  <div id="mynetwork"></div>
  <div id="loadingBar">
    <div class="outerBorder">
      <div id="text">0%</div>
      <div id="border">
        <div id="bar"></div>
      </div>
    </div>
  </div>
</div>

<script  type="text/javascript">
var tools = [  
	{% for tool in site.data %}
	{% assign tool_selected = tool[1] %}
	{% assign tool_filename = tool[0] %}	
    {
      id: "{{tool_filename}}",
	  label: "{{tool_selected.name}}",
	  title: "{{tool_selected.name}}",
	  group: "{{tool_selected.type}}"
    }{% unless forloop.last %},{% endunless %}
  {% endfor %}
];
var sources = [  
	{% for tool in site.data %}
	{% assign tool_selected = tool[1] %}
	{% assign tool_filename = tool[0] %}
		{% for source in tool_selected.sources %}
		{
		  from: "{{source.code}}",
		  to: "{{tool_filename}}"
		},
		{% endfor %}
	{% endfor %}
];
var destinations = [  
	{% for tool in site.data %}
	{% assign tool_selected = tool[1] %}
	{% assign tool_filename = tool[0] %}
		{% for destination in tool_selected.destinations %}
		{
		  from: "{{tool_filename}}",
		  to: "{{destination.code}}"
		},
		{% endfor %}
	{% endfor %}
];



</script>
{% comment %}
{{ site.data | jsonify}}}
{% endcomment %}


<script  type="text/javascript">
let tool_ids = tools.map(a => a.id);
var tools2 = tools;
sources.forEach( source => {
	if(! tool_ids.includes(source.from)){
		tools2.push({id: source.from, label:source.from, size:4,title:source.from, group: "unknown"});
		tool_ids.push(source.from)
	}		
})
destinations.forEach( destination => {
	if(! tool_ids.includes(destination.to)){
		tools2.push({id: destination.to, label:destination.to, size:4,title:destination.to, group: "unknown"});
		tool_ids.push(destination.to)
	}		
})



var nodes = tools2;
var edges = sources.concat(destinations);
var network;
var allNodes;
var highlightActive = false;

var nodesDataset = new vis.DataSet(nodes); 
var edgesDataset = new vis.DataSet(edges); 

function draw() {
  var container = document.getElementById("mynetwork");
  var options = {
    nodes: {
      shape: "box",
	  labelHighlightBold: true,
      font: {
        size: 12,
        face: "Tahoma",
		align: "left" 
      },
    },
    edges: {
      width: 0.15,
      color: { inherit: "to" },
      smooth: {
        type: "continuous",
      },
	  arrows: "to",
	  hoverWidth: 3
    },
	layout: {
		improvedLayout: false
	},
    physics: {
		solver:'repulsion'
	},
    interaction: {
      tooltipDelay: 200,
      hideEdgesOnDrag: false,
      hideEdgesOnZoom: true,
	  hover: true,
    },
  };
  var data = { nodes: nodesDataset, edges: edgesDataset }; 

  network = new vis.Network(container, data, options);

  network.on("stabilizationProgress", function (params) {
    var maxWidth = 496;
    var minWidth = 20;
    var widthFactor = params.iterations / params.total;
    var width = Math.max(minWidth, maxWidth * widthFactor);

    document.getElementById("bar").style.width = width + "px";
    document.getElementById("text").innerText =
      Math.round(widthFactor * 100) + "%";
  });
  network.once("stabilizationIterationsDone", function () {
    document.getElementById("text").innerText = "100%";
    document.getElementById("bar").style.width = "496px";
    document.getElementById("loadingBar").style.opacity = 0;
    // really clean the dom element
    setTimeout(function () {
      document.getElementById("loadingBar").style.display = "none";
    }, 500);
  });
  
  // get a JSON object
  allNodes = nodesDataset.get({ returnType: "Object" });
  
  network.on("click", neighbourhoodHighlight);
}

function neighbourhoodHighlight(params) {
  // if something is selected:
  if (params.nodes.length > 0) {
    highlightActive = true;
    var i, j;
    var selectedNode = params.nodes[0];
    var degrees = 2;

    // mark all nodes as hard to read.
    for (var nodeId in allNodes) {
      allNodes[nodeId].color = "rgba(200,200,200,0.5)";
      if (allNodes[nodeId].hiddenLabel === undefined) {
        allNodes[nodeId].hiddenLabel = allNodes[nodeId].label;
        allNodes[nodeId].label = undefined;
      }
    }
    var connectedNodes = network.getConnectedNodes(selectedNode);
    var allConnectedNodes = [];

    // get the second degree nodes
    for (i = 1; i < degrees; i++) {
      for (j = 0; j < connectedNodes.length; j++) {
        allConnectedNodes = allConnectedNodes.concat(
          network.getConnectedNodes(connectedNodes[j])
        );
      }
    }

    

    // all first degree nodes get their own color and their label back
    for (i = 0; i < connectedNodes.length; i++) {
      allNodes[connectedNodes[i]].color = undefined;
      if (allNodes[connectedNodes[i]].hiddenLabel !== undefined) {
        allNodes[connectedNodes[i]].label =
          allNodes[connectedNodes[i]].hiddenLabel;
        allNodes[connectedNodes[i]].hiddenLabel = undefined;
      }
    }

    // the main node gets its own color and its label back.
    allNodes[selectedNode].color = undefined;
    if (allNodes[selectedNode].hiddenLabel !== undefined) {
      allNodes[selectedNode].label = allNodes[selectedNode].hiddenLabel;
      allNodes[selectedNode].hiddenLabel = undefined;
    }
  } else if (highlightActive === true) {
    // reset all nodes
    for (var nodeId in allNodes) {
      allNodes[nodeId].color = undefined;
      if (allNodes[nodeId].hiddenLabel !== undefined) {
        allNodes[nodeId].label = allNodes[nodeId].hiddenLabel;
        allNodes[nodeId].hiddenLabel = undefined;
      }
    }
    highlightActive = false;
  }

  // transform the object into an array
  var updateArray = [];
  for (nodeId in allNodes) {
    if (allNodes.hasOwnProperty(nodeId)) {
      updateArray.push(allNodes[nodeId]);
    }
  }
  nodesDataset.update(updateArray);
}

window.addEventListener("load", () => {
  draw();
});

</script>




</body>
</html>

