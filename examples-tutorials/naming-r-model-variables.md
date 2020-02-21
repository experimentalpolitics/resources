Model covariate names
================
gist by [*democracyobserver*](https://github.com/democracyobserver)  
2/20/2020

Scientific writing that includes model output tables can be difficult to
read if the variables included are not well named. While clever readers
may be able to recognize what a covariate named "gdppc\_avg" might mean,
authors should provide a variable label instead of the variable name as
it appears in the data. For instance, “Average GDP per capita” is much
more obvious.

There are many ways to create more readable tables for your manuscript.
One such method (for use in `R`), which is particularly suitable for
multiple tables containing the same variables, is included here. This
approach will work with both `data.frame` and `tibble` data objects and
is intended to reduce the need to manually replace or add the variable
names each time a new table is created.

First, for the sake of the example I created some fake data.

``` r
library(tibble)
# fake data (note, you can make this a data.frame if you like)
dat <- tibble(
  id = letters,
  cash = sample(seq(100, 10000, 100), 26, replace = TRUE),
  college = sample(c(0, 1), 26, replace = TRUE),
  sex = sample(c(0, 1), 26, replace = TRUE),
  debt = sample(seq(0, 0.25, .01), 26, replace = TRUE)
)
```

We can now run a simple OLS model where "debt" is the dependent
variable. I like to use Marek Hlavac’s `stargazer` to render tables.
This package can flexibly handle most model types and has many arguments
available to create the perfect table.

``` r
# simple model
mod <- lm(debt ~ cash + college + sex, data = dat)
# output
stargazer::stargazer(mod, type = "text")
```

``` 

===============================================
                        Dependent variable:    
                    ---------------------------
                               debt            
-----------------------------------------------
cash                          0.00000          
                             (0.00000)         
                                               
college                        0.016           
                              (0.028)          
                                               
sex                            0.021           
                              (0.029)          
                                               
Constant                      0.077**          
                              (0.030)          
                                               
-----------------------------------------------
Observations                    26             
R2                             0.083           
Adjusted R2                   -0.042           
Residual Std. Error       0.069 (df = 22)      
F Statistic             0.660 (df = 3; 22)     
===============================================
Note:               *p<0.1; **p<0.05; ***p<0.01
```

Ignore the results, they are meaningless. The model output is not
entirely clear; what do "college" and "sex" mean? Even if we knew the
unit of analysis here (individuals), without a reference category these
variable labels could mean anything. One way to fix this problem is to
manually edit the table, or pass the `stargazer()` function call some
useful arguments (i.e. `covariate.labels`) to clarify the variable
labels. If you have more than one table, particularly if there are many
variables, this can be inefficient. Instead, we can use `attributes` to
help us out.

``` r
# what attributes are there?
names(attributes(dat))
```
``` 
[1] "names"     "row.names" "class"    
```

Predictably (because tibbles are special data frames) there are *names* (column names), *row.names*, and *class* attributes. If we want to add another attribute, it is easy using `attr()`:

``` r
# add one; must be same length of ncol(dat)
attr(dat, "model.varnames") <- c("Unique observation ID", "Cash on hand", "College student", "Female", "Debt load")
# check it out
names(attributes(dat))
```
```
[1] "names"          "row.names"      "class"          "model.varnames"
```


``` r
# accessing attributes by name with attr
attr(dat, "model.varnames")
```
``` 
[1] "Unique observation ID" "Cash on hand"          "College student"      
[4] "Female"                "Debt load"            
```

Now that we have a list of sensible variable labels to draw from, adding
them to the model output in `stargazer` is easy\!

``` r
# now use it in stargazer, using indexing to grab the right ones
stargazer::stargazer(mod, type = "text", 
          covariate.labels = attr(dat, "model.varnames")[2:4], 
          dep.var.labels = attr(dat, "model.varnames")[5])
```

``` 

===============================================
                        Dependent variable:    
                    ---------------------------
                             Debt load         
-----------------------------------------------
Cash on hand                  0.00000          
                             (0.00000)         
                                               
College student                0.016           
                              (0.028)          
                                               
Female                         0.021           
                              (0.029)          
                                               
Constant                      0.077**          
                              (0.030)          
                                               
-----------------------------------------------
Observations                    26             
R2                             0.083           
Adjusted R2                   -0.042           
Residual Std. Error       0.069 (df = 22)      
F Statistic             0.660 (df = 3; 22)     
===============================================
Note:               *p<0.1; **p<0.05; ***p<0.01
```

This version of the table has the dependent variable and the covariates
named in such a way that anyone reading it can likely figure out what
these mean. A win for transparency and communication\!
