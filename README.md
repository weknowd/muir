#muir

The **muir** package allows users to explore a data.frame using the tree data structure with minimal effort by simply providing the data.frame columns to be explored. In addition, the **muir** package allows for more targeted tree data structures to be created with specific column criteria as a method for documenting and communicating the data structure and relationships within a data set. These data tree structures can be viewed within the *RStudio* console, standard browers, or saved as HTML for sharing using the **htmlwidgets** package.

The package leverages the infrastructure provided by [**DiagrammeR**](http://rich-iannone.github.io/DiagrammeR/).

## Installation

Install the development version of **muir** from GitHub using the **devtools** package.

```r
devtools::install_github('alforj/muir')
```

Or, get the v0.1.0 release from **CRAN**.

```r
install.packages('muir')
```

## Using muir

### Basic example
A basic example using **muir** to explore the **mtcars** data set showing the 'cyl' and 'carb' columns
and the relationship between them (in that order). 

Providing the "\*" qualifier (after the ":" separator) indicates that all distinct values in the column
should be returned as separate nodes. Obviously, depending on the data this may result in a LOT of nodes.
As a guard against this, especially during initial exploration, a *node.limit* parameter limits the 
number of nodes returned at each level and is defaulted to three (3). If there are more than 3 distinct
values, then only the Top 3 occuring values will be returned. This global parameter can be adjusted for all node levels or specific limits can be provided individually for each *node.level* value (for example, by providing "cyl:4" instead of "cyl:*" to indicate a 
node limit of 4 for the 'cyl' column only).

The resulting tree will be rendered starting with a level 0 node counting all rows in the data set. Each resulting level will be based on the columns provided by the user in *node.levels*. Each subsequent level will carry the filters from previous parent nodes forward. Percentages will be provided for each node (compared to the level 0 count) by default and can be turned off if not desired.

*tree.height* and *tree.width* values control how the tree is rendered and can be adjusted to best fit trees of various depths and widths.


```r
library(muir)
data(mtcars)
mtTree <- muir(data = mtcars, node.levels = c("cyl:*", "carb:*"), 
               tree.height = 1200, tree.width = 800)

mtTree
```

<!--html_preserve--><div id="htmlwidget-8292" style="width:800px;height:1200px;" class="DiagrammeR"></div>
<script type="application/json" data-for="htmlwidget-8292">{ "x": {
 "diagram": "graph LR;1(All<br/>Count: 32.00<br/>% of Total: 100.00<br/>);1-->2(cyl = 8<br/>Count: 14.00<br/>% of Total:  43.75<br/>);1-->3(cyl = 4<br/>Count: 11.00<br/>% of Total:  34.38<br/>);1-->4(cyl = 6<br/>Count: 7.00<br/>% of Total:  21.88<br/>);2-->5(carb = 2<br/>Count: 4.00<br/>% of Total:  12.50<br/>);2-->6(carb = 4<br/>Count: 6.00<br/>% of Total:  18.75<br/>);2-->7(carb = 1<br/>Count: NA<br/>% of Total:   0.00<br/>);3-->8(carb = 2<br/>Count: 6.00<br/>% of Total:  18.75<br/>);3-->9(carb = 4<br/>Count: NA<br/>% of Total:   0.00<br/>);3-->10(carb = 1<br/>Count: 5.00<br/>% of Total:  15.62<br/>);4-->11(carb = 2<br/>Count: NA<br/>% of Total:   0.00<br/>);4-->12(carb = 4<br/>Count: 4.00<br/>% of Total:  12.50<br/>);4-->13(carb = 1<br/>Count: 2.00<br/>% of Total:   6.25<br/>);linkStyle default stroke-width:2px, fill:none;classDef default fill:white,stroke:#333,stroke-width:2px;classDef invisible fill:white,stroke:white,stroke-width:0px;" 
},"evals": [  ] }</script><!--/html_preserve-->


### More complicated example
Instead of (or in combination with) just returning top counts for columns provided in *node.levels*,
you can provide custom filter criteria and custom node titles in *level.criteria*
(*level.criteria* could also be read in from a stored file (e.g., a crtieria.csv) as a data.frame)

The **criteria** data.frame below includes the column names, operators, and associated values 
(e.g., "cyl" <= 4), and a node title to accompany each node generated for that filter criteria. 

Adding a "+" suffix after the column name in the *node.levels* parameter will add an extra 
"Other" node that will aggregate all values not already provided in the *level.criteria* value 
or for all distinct values below the *node.limit* provided.

Additional label values can be provided with the *label.vals* parameter using 
[**dplyr**](https://github.com/hadley/dplyr) summary functions. Custom labels for each value 
can be provided by adding custom text after the ":" separator. 

Lastly, the direction the tree is drawn can be changed from the default left-to-right to a 
top-to-bottom ("TB") rendering by providing a new value for *tree.dir*.


```r
criteria <- data.frame(col = c("cyl", "cyl", "carb"),
                       oper = c("<=", ">", "=="),
                       val = c(4, 4, 2),
                       title = c("Up to 4 Cylinders", "More than 4 Cylinders", "2 Carburetors"))

mtTree <- muir(data = mtcars, node.levels = c("cyl", "carb:+"),
               level.criteria = criteria,
               label.vals = c("n():Count", "min(wt):Min Weight", "max(wt):Max Weight"),
               tree.dir = "TB",
               tree.height = 400, tree.width = 800)

mtTree
```

<!--html_preserve--><div id="htmlwidget-7784" style="width:800px;height:400px;" class="DiagrammeR"></div>
<script type="application/json" data-for="htmlwidget-7784">{ "x": {
 "diagram": "graph TB;1(All<br/>Count: 32.00<br/>Min Weight: 1.51<br/>Max Weight: 5.42<br/>% of Total: 100.00<br/>);1-->2(Up to 4 Cylinders<br/>Count: 11.00<br/>Min Weight: 1.51<br/>Max Weight: 3.19<br/>% of Total:  34.38<br/>);1-->3(More than 4 Cylinders<br/>Count: 21.00<br/>Min Weight: 2.62<br/>Max Weight: 5.42<br/>% of Total:  65.62<br/>);2-->4(2 Carburetors<br/>Count: 6.00<br/>Min Weight: 1.51<br/>Max Weight: 3.19<br/>% of Total:  18.75<br/>);2-->5(Other<br/>Count: 5.00<br/>Min Weight: 1.84<br/>Max Weight: 2.46<br/>% of Total:  15.62<br/>);3-->6(2 Carburetors<br/>Count: 4.00<br/>Min Weight: 3.44<br/>Max Weight: 3.85<br/>% of Total:  12.50<br/>);3-->7(Other<br/>Count: 17.00<br/>Min Weight: 2.62<br/>Max Weight: 5.42<br/>% of Total:  53.12<br/>);linkStyle default stroke-width:2px, fill:none;classDef default fill:white,stroke:#333,stroke-width:2px;classDef invisible fill:white,stroke:white,stroke-width:0px;" 
},"evals": [  ] }</script><!--/html_preserve-->


### More examples

More examples can be found in the *examples* in the **muir** function


```r
help(muir)
```
