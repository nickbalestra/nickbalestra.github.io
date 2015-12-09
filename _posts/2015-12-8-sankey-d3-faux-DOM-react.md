---
title: "D3 vs. React DOMination"
date: 2015-12-8
description: Relying on fake DOM to use D3 outside the browser and React for rendering its result.
---

If you try pairing up [D3](http://d3js.org/) with [React](https://facebook.github.io/react/) to create interactive data visualizations, you'll soon discover that both libraries try to solve the same problem and they both want to be in control of the DOM. While React does this with its DOM diffing approach, D3 does it with its enter-update-exit pattern.

How to avoid them fighting over DOMination and what approach is best? [Shirley Wu](http://slides.com/shirleywu/deck#/) does a great job dissecting the problem into three possible approaches. In this post I explore a variation of her first approach, by not only using D3 for data calculation but also to build the DOM elements that React will then render. How? By tricking D3 into believing he's working on the DOM while is not...

We'll build and use as a reference a simple Sankey diagram builder app:
![index page](https://raw.githubusercontent.com/nickbalestra/nickbalestra.github.io/master/assets/images/sankeyApp.png)

Check the [live demo](http://nick.balestra.ch/sankey/).

***

### Fake it till you make it

Say hello to [react-faux-dom](https://github.com/Olical/react-faux-dom): A DOM-like data structure to be mutated by D3 then rendered to React elements.The idea is rather simple: By tricking D3 into thinking it's acting on the DOM we can rely on many of its methods to manipulate such structure. React-faux-dom supports selector engine and comes with a partial support for addEventListener, perfect if we want to add element interaction.

To use it is a matter of requiring it to create out fake DOM structures, in the example we create a 'div' element.

{% highlight javascript linenos %}
import ReactFauxDOM from 'react-faux-dom';

// Create a faux-DOM 'div' element
var divNode = ReactFauxDOM.createElement('div');
{% endhighlight %}

We can now feed this to D3:

{% highlight javascript linenos %}
// Set units, margin, sizes
var margin = { top: 10, right: 0, bottom: 10, left: 0 };
var width = 690 - margin.left - margin.right;
var height = 400 - margin.top - margin.bottom;

// Let D3 create and append an svg to our faux-DOM
var svg = d3.select(divNode).append("svg")
  .attr("width", width + margin.left + margin.right)
  .attr("height", height + margin.top + margin.bottom)
 .append("g")
 .attr("transform", "translate(" + margin.left + "," + margin.top + ")");
{% endhighlight %}

If we would have relied on JSX instead, our code would looked something like:


{% highlight javascript linenos %}
// Set units, margin, sizes
var margin = { top: 10, right: 0, bottom: 10, left: 0 };
var width = 690 - margin.left - margin.right;
var height = 400 - margin.top - margin.bottom;

// Create our React element structure via JSX
<div>
  <svg width={width + margin.left + margin.right} height={height + margin.top + margin.bottom}>
    <g transform={"translate(" + margin.left + "," + margin.top + ")"}></g>
  </svg>
</div>
{% endhighlight %}

Once you finish faking it, and it's time to making it, just return it as a react element to be rendered:

{% highlight javascript linenos %}
return divNode.toReact();
{% endhighlight %}

*** 

### Sankey Diagram, a real example

To put all the pieces together and see this approach in a more concrete example, we'll be building an interactive App that allows us to construct and edit [Sankey's diagrams](http://bost.ocks.org/mike/sankey/).

The approach will be:

- React for structure
- D3 for data calculation 
- D3/Faux-DOM for React DOM elements construction
- React for rendering

<br>As [Shirley Wu](https://twitter.com/shirleyxywu) mention, this approach may be good for a react app with a small and simple visualization. It comes with the pro of being Clean and easy and the added benefit of being able to port quickly D3 visualizations to react as we don't have to refactor them into JSX. 

#### 1) React for structure

We are going to use a wrapping React Component to hold the state and methods we may need to pass down to other components that need to be able to update the visualization.


{% highlight javascript linenos %}
class App extends React.Component {
  constructor() {
    super()

    this.state = {
      nodes: [],
      links: [],
      modalIsOpen: false
    };

    this.addNode = this.addNode.bind(this);    
    this.openModal = this.openModal.bind(this);
    ...
  }

  ...
{% endhighlight %}

The whole structure could look something like:

{% highlight javascript linenos %}
app.js
sankey.css
SankeyChart.js
utils.js
toolbars/
- AddLink.js
- AddNode.js
- TopBar.js
- FooterBar.js
{% endhighlight %}

On a side note, we'll be using [webpack](https://webpack.github.io/) for our build.

#### D3 for data calculation

We are going to rely on D3 and its Sankey plugin for the visualization calculation:

{% highlight javascript linenos %}
import d3 from 'd3';
import sankey from 'd3-plugins-sankey';

var sankey = d3.sankey()
  .size([width, height])
  .nodeWidth(15)
  .nodePadding(10);

var path = sankey.link();

var graph = {
  nodes: _.cloneDeep(this.state.nodes),
  links: _.cloneDeep(this.state.links)
};

sankey.nodes(graph.nodes)
  .links(graph.links)
  .layout(32);
{% endhighlight %}


#### D3/Faux-DOM for React DOM elements construction

Now, as we saw earlier, we can just rely on D3 + faux-DOM to build our REACT elements


{% highlight javascript linenos %}
// Initialize and append the svg canvas to faux-DOM
var svgNode = ReactFauxDOM.createElement('div');

var svg = d3.select(svgNode).append("svg")
  .attr("width", width + margin.left + margin.right)
  .attr("height", height + margin.top + margin.bottom)
  .append("g")
  .attr("transform", "translate(" + margin.left + "," + margin.top + ")");


// Add links
var link = svg.append("g").selectAll(".link")
  .data(graph.links)
  .enter().append("path")
  .attr("class", "link")
  .on('click', this.props.openModal) // register eventListener
  .attr("d", path)
  .style("stroke-width", (d) => Math.max(1, d.dy))

// add link titles
link.append("title")
  .text((d) => d.source.name + " → " + d.target.name + "\n Weight: " + format(d.value));


// Add nodes
var node = svg.append("g").selectAll(".node")
  .data(graph.nodes)
  .enter().append("g")
  .attr("class", "node")
  .on('click', this.props.openModal) // register eventListener
  .attr("transform", (d) => "translate(" + d.x + "," + d.y + ")")

// add nodes rect
node.append("rect")
  .attr("height", (d) => d.dy)
  .attr("width", sankey.nodeWidth())
  .append("title")
  .text((d) => d.name + "\n" + format(d.value));

// add nodes text
node.append("text")
  .attr("x", -6)
  .attr("y", (d) => d.dy / 2)
  .attr("dy", ".35em")
  .attr("text-anchor", "end")
  .text((d) => d.name)
  .filter((d) => d.x < width / 2)
  .attr("x", 6 + sankey.nodeWidth())
  .attr("text-anchor", "start");
{% endhighlight %}

One interesting thing is that we can register our eventListener directly within the D3 chain, i.e.:
{% highlight javascript linenos %}
  .on('click', this.props.openModal) 
{% endhighlight %}

#### React for rendering 

We can now return our faux-dom element by converting back to a react element so that can safely render it

{% highlight javascript linenos %}
return svgNode.toReact();
{% endhighlight %}

***

### Thoughts and resources

This approach could work pretty well if you want to port quickly some D3 code to react and if you are familiar with D3 more then JSX. Otherwise, a JSX only approach where we rely on D3 only for data calculation and not for DOM manipulation might be clearer, and the code produced perhaps shorter.

In the code comments you can find the [JSX alternative](https://github.com/nickbalestra/sankey/blob/master/app/SankeyChart.js#L111-L151) to create the react elements, I leave it to you pick your poison.

- [Shirley Wu](http://slides.com/shirleywu/deck#/) Deck on D3 + React
- [Github Repo](https://github.com/nickbalestra/sankey) for the demo Sankey App
- [Live demo](http://nick.balestra.ch/sankey/) of the app.
- [react-faux-dom](https://github.com/Olical/react-faux-dom) project

