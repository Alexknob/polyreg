
![](/Figures/Concept_figures/Poly_reg_tutorial_intro.png)

---------------------

# **Polynomial Regression**

## **The extended family of linear regression**

## **How to extend linear models to fit normally distributed non-linear data**

##### *a data science tutorial by Alexandra Knoblauch*
--------------------------

### Tutorial Aims & Steps

#### <a href="#Setting-yourself-up">1. Setting yourself up</a>

#### <a href="#Your-Research-Question-and-Hypothesis"> 2. Your Research Question and Hypothesis</a>

#### <a href="#Explore-your-Data"> 3. Explore your Data</a>

#### <a href="#Apply-a-Linear-Model"> 4. Apply a Linear Model</a>

#### <a href="#A-bit-of-Theory-on-Polynomial-Regression"> 5. A bit of Theory on Polynomial Regression</a>

#### <a href="#Develop-a-Polynomial-Model-of-Degree-2"> 6. Develop a Polynomial Model of Degree 2</a>

#### <a href="#Adding-more-Terms-Polynomial-Models-of-higher-Degrees"> 7. Adding more Terms: Polynomial Models of higher Degrees</a>

#### <a href="#How-do-you-Decide-which-Model-to-use?"> 8. How do you decide which model to use?</a>

#### <a href="#Check-your-Assumptions"> 9. Check your Assumptions</a>

#### <a href="#Save-and-Report-your-Results"> 10. Save and Report your Results</a>

#### <a href="#A-word-of-Caution"> 11. A word of Caution</a>

#### <a href="#Learning-Outcomes"> 12. Learning Outcomes</a>

#### <a href="#Further-Resources"> 13. Further Resources</a>

------------------------------

### Learning Objectives

By the end of this tutorial you should be able to...

* Explore your data with scatterplots and histograms
* Develop simple linear models
* Develop linear models of higher degrees
* Assess model fit visually and with statistical tests
* Check the assumptions of linear models
* Export and report your modelling results in tables and graphs

----------------------------

So... Another modelling tutorial... By this time you might have already heard about linear models, anovas, linear mixed models, Bayesian modelling or hierarchical models. Maybe you have used some of these models in the past and have got a good idea of what to watch out for and what the statistics are behind those models.

If you have not and would like to learn more about these models try this [link](https://ourcodingclub.github.io/tutorials.html).

In this tutorial we are going to explore another modelling type: the bigger brother (or sister...) of linear models: `polynomial regression`. Sometimes when we plot two variables against each other their relationship will not always look like a straight line but somehow curved and non-linear. We may have tried a histogram and know that the data is somewhat normally distributed but <mark>HOW</mark> do we model the non-linear relationship? This is where we can use `polynomial regression` :smile:
In this tutorial we will explore how we can build polynomial regression models, how to choose the 'right one' and how to report our modelling results :nerd_face:

To get all the resources for this tutorial clone or download the zip file of this <a href="https://github.com/Alexknob/polyreg" target="_blank">GitHub repository</a> and unzip it.


<a name='Setting-yourself-up'></a>

## 1. Setting yourself up

Now lets dive straight in. If you have not done it yet, clone or download the tutorial resources from this <a href="https://github.com/ourcodingclub/CC-EAB-tut-ideas" target="_blank">GitHub repository</a> and open a new script in `RStudio` on your computer.

It is always good to add some explanatory information at the top of your script for example add your name, date and purpose of the script. You can use the Addin `Little Boxes` ([more info here](https://rdrr.io/github/ThinkRstat/littleboxes/)) to nicely format this information which will transform it into this style:

```r
##%######################################################%##
#                                                          #
####    Tutorial Script  Tutorial Development in R      ####
####           Teaching a quantitative Skill            ####
####              to Data Science Students              ####
####                   Topic: Polynomial Regression     ####
####             Script used in the developed           ####
####     tutorial Alexandra Knoblauch   s2090259        ####
####    Data Science for Ecological and Environmental   ####
####     Sciences (EES)   Year 2022/23   28.11.2022     ####
#                                                          #
##%######################################################%##
```
It is also good practice to divide your code into sections by adding `----` after comments (starting with `#`) which will create a new section in your script. More on good coding etiquette can be found [here](https://ourcodingclub.github.io/tutorials/etiquette/).

Now lets load in our libraries. If you have not installed the required packages run the `ìnstall.packages('package_name')` command.

```r
# Libraries ----

# if you have not installed the required packages, run the command
# install.packages('nameofthepackage')

# load the required packages
library(tidyverse) # includes ggplot and dplyr for data visualisation and manipulation
library(wesanderson) # colour palette
library(ggExtra) # additional gg plot functions
library(gridExtra) # arrange plots
library(broom) # to tidy up model summaries
library(beepr) # for sounds, so sound on please!
```
We can also set up our own `ggplot2` theme function. This will save us loads of time later :sweat_smile: and will ensure that all our graphs have the same format.

```r
# Functions ----

# customised ggplot function to produce graphs with a consistent format

theme.ggplot <- function (){
  # set a plain background
  theme_bw() +
    # modify different plot elements
    theme (
     # add a plot border
     panel.border = element_rect(colour='black', fill = NA, size = 1),
     # remove gridlines
     panel.grid.major = element_blank(),
     panel.grid.minor = element_blank(),
     # change text size and font of the x and y axis
     axis.text.x = element_text (size = 12),
     axis.text.y = element_text (size = 12),
     axis.title.x = element_text (size = 14, face = 'plain'),
     axis.title.y = element_text (size = 14, face = 'plain'),
     # modify legend text
     legend.text = element_text(size = 12, face = 'plain'),
     legend.title = element_text(size = 14, face = 'plain'),
     # modify plot title
     plot.title = element_text(size = 15, face = 'bold'),
     # change plot margins
     plot.margin = unit(c(0.5, 0.5, 0.5, 0.5), units = 'cm'))
}
```

And of course what would a graph be without pretty colours? :rainbow:
We are going to use the `Darjeeling1` colour palette from the [wesanderson](https://github.com/karthik/wesanderson) package. Basically, you can use any colour scheme that you want. To find out more about colours in `ggplot2` check out [this website](https://r-graph-gallery.com/ggplot2-color.html). Lets set the colour palette up.

```r
# Create colour palette used for plots

# use Darjeeling1 colour palette from the wesanderson package
# if you want another palette from the package run the following command to access their names
# names(wes_palettes)
my.colours <- wes_palette('Darjeeling1', 5, type = c('discrete', 'continuous'))
```

Okay now we are almost set up. One thing is missing though: our data. :chart_with_upwards_trend:
We need to specify our working directory for the script so that we can correctly load in the data file.

```r
# Load Data ----

# set your working directory to the folder that you downloaded from Github
# for example
# setwd('~/Downloads/polynomial-regression')

# if you do not know your current working directory run
# getwd()

# and then set your working directory with the code above
```

-------

Once we have set this up we can load in the ``data``. We will be looking at a data set created and sent to us from a galaxy far far away :milky_way: :rocket: :star:

![](/Figures/Concept_figures/Milky_Way.jpeg)
[Source: Oliver Griebl](https://commons.wikimedia.org/wiki/File:Milky_Way_%28262681609%29.jpeg) *(This image is unchanged and licensed under the [Creative Commons Attributions 3.0 Unported License](https://creativecommons.org/licenses/by/3.0/deed.en))*

The rebels have found out that evocs :bear: like music :notes:. They have played a song several times on repeat (continuous) and have measured evoc happiness during that time. Now they want us to analyse their data since it might help them to encourage the evocs to fight with them against the imperium.

---------

Lets load in the `.csv` file called `evoc_data.csv` and store it in the name `evocs`. In `R` you can store objects in variables with the `<-` command. *(Quick side note: we are using a `.csv` file bacause `R` works quicker with this file format than with excel `.xlsx` files)*

```r
# load in the Evoc dataset evoc_data.csv from the downloaded repository

evocs <- read.csv('~/Downloads/polynomial-regression/evoc_data.csv', head = TRUE)
```

If everything worked fine, you should now see a file called `evocs` pop up under the `Environment` tab on the right hand panel on your `RStudio` window.
And that is us set up :party:

<a name="Your-Research-Question-and-Hypothesis"></a>

## 2. Your Research Question and Hypothesis

This is the most important part of your model design!
Your <mark>research question</mark> will set the underlying structure of your study, experiment and of course of the model.

Based on your understanding of the system's processes you will also set up a <mark>hypothesis</mark></a> on how <mark>variables</mark></a> relate that we can then test with our model. This reduces the risk that you don't come up with a hypothesis after you have done all your analysis (known as <mark>HARKing</mark></a>)because that would defeat the purpose of good scientific research. You can read more on this [here](https://www.psychiatrist.com/jcp/assessment/harking-cherry-picking-p-hacking-fishing-expeditions-and-data-dredging-and-mining-as-questionable-research-practices/).

You may also want to come up with a <mark>null hypothesis</mark></a> that applies should you prove your initial hypothesis wrong and variables do not relate in the way you thought they would.

So, before getting started we need to know what the variables are that we want to understand, we need to set our research question and come up with our hypothesis (and null hypothesis).

Lets look at the `evocs` data frame to see what variables we have:

```r
# Explore data ----

# explore the data

head(evocs) # get the first few rows
str(evocs) # look at data structure
```
`head` lets you see the first few rows of the data and `str` shows you the types of variables in the dataframe.
We can see that we have two variables here: `numb_listened` and `evoc_happiness` and both variables are `numeric` which means they just contain numbers.

What do these variables mean?
`numb_listened` is the number of times the song has been played on repeat and is continuous. `evoc_happiness` is the evoc happiness the rebels measured.

We want to know how do evocs respond in their happiness to the number of times they have listened to a song. So our research question could be:

:question: How does evoc happiness change with the number of times a song has been played?

Based on this our hypothesis could be something like:

:exclamation: Evoc happiness increased with the number of times they have listened to a song.

*(We phrase our hypothesis in past tense since the data has already been collected and observations have already been made)*

Our null hypothesis could then be:

:grey_exclamation: Evoc happiness decreased with the number of times listening to a song.

Now if we want to answer our research question with a model we need to define the x and y variables in the model (basically the explanatory - what goes on your x axis- and response variable - what goes on your y-axis-).
Here `numb_listened` is our explanatory variable so the `x variable` and `evoc_happiness` is the response variable so our `y variable`.

<a name="Explore-your-Data"></a>

## 3. Explore your Data

Now lets explore our data a bit further so that we know what models we can apply. We will be doing that visually since everyone probably loves graphs and it is just easier to understand.

First, let us remove the nasty `X` column that has accidentally been introduced into our data (Probably something got messed up when switching into lightspeed mode during data transport :stars:). We will do this with a `Pipe` operator form the `dplyr` package included in the `tidyverse` package which looks like this `%>%`. You can find more info on this [here](https://www.tidyverse.org/packages/).

```r
# remove X column

evocs <- evocs %>%
  select(-X)
```
You can check your data frame again by clicking on the `evocs` data frame in the `Environment` tab on the right side of your `RStudio` window. This should display the data frame.

Okay tiptop. We can now make a nice scatter plot of the data to look how our two variables `numb_listened` and `evoc_happiness` are related. If you want to know more about the cool things `ggplot2` can do check out [this tutorial](https://ourcodingclub.github.io/tutorials/datavis/).

Note that if you place `()` around the entire plot command the graph will automatically show in the `Plots` pannel on the bottom right of your `RStudio` window.

```r
# make a scatter plot to look at the relationship of evoc happiness to number of times a song is played

# the () around the plot will automatically display the plot
(evoc_scatter <- ggplot(data = evocs) +
    # create a scatter plot
    geom_point(aes(x = numb_listened, y = evoc_happiness),
               # modify point attributes
               colour = my.colours[5], alpha = 0.5, pch = 21, stroke = 1, size = 3) +
    # add customised function
    theme.ggplot() +
    # rename axis
    xlab('Listening to the Song') +
    ylab('Evoc Happiness'))
```
Okay, so the graph should look something like this:

![](/Figures/tutorial_output_figures/evoc_scatter.png)

We can clearly see that something is going on here. Evoc happiness seems to show an increasing trend with the number of times a song has been played. The plot also shows that tis relation is not really linear but more curved... We will worry about this a bit later.

Lets find out how our measured data points are distributed. This will determine what kind of model we will use. More on data distributions can be found [here](https://rstudio-pubs-static.s3.amazonaws.com/100906_8e3a32dd11c14b839468db756cee7400.html).

We will use a histogram to look at the data distribution.

```r
# add histograms to each side of the scale to see how the data is distributed
(evoc_scatter_hist <- ggMarginal(evoc_scatter,
                                  type = 'histogram',
                                  colour = 'white',
                                  alpha = 0.6,
                                  fill = my.colours[5],
                                  xparams = list(binwidth = 1),
                                  yparams = list(binwidth = 30)))
```
We have added histograms to both sides of the x and y axis and got this plot:

![](/Figures/tutorial_output_figures/evoc_scatter_hist.png)

So what does this tell us? :thinking:

* :mag: **The upper histogram** this distribution shows the data distribution of the `x-axis`. It makes sense that all the bars are the same size and evenly distributed. The number of times the song has been played is evenly distributed with the same timestep between songs.
* :mag: **The right histogram** this looks a like bell-shaped curve :bell: typical for a normal distribution of our data points. This is <mark>GREAT</mark> because we can probably use a `linear model` to explain the relationship between our variables. *If you have never heard about linear models and their assumptions, check this great tutotrial out [here](https://ourcodingclub.github.io/tutorials/modelling/)*.


<a name="Apply-a-Linear-Model"></a>

## 4. Apply a Linear Model

Okay cool. We know now that we can possibly fit a `linear model` to our data. Lets try with the most simple version that we all know.

(*Again check this [tutorial](https://ourcodingclub.github.io/tutorials/modelling/) out if you need a wee refresher*)

```r
# Linear model ----

# make a simple linear model of listening to the song and evoc happiness

model1 <- lm(evoc_happiness ~ numb_listened, data = evocs)

# retrieve the model summary
summary(model1)
```
You should get something like this:

![](/Data/tutorial_output_data/evocs_lm_model1_output.png)

We can see that our `p-value` is very low for both model coefficients `Intercept` and `numb_listened`. Remember that the `Intercept` is the value where our <mark>straight line</mark> crosses the y-axis - so basically our value at 0. `numb_listened` is the <mark>slope</mark> of the line - so how much our response variable changes over time.

The `numb_listened` coefficient is :arrow_upper_right: positive indicating that `evoc_happiness` is increasing with the number of times they listen to a song.

Lets make sense of this visually.

```r
# plot the linear model onto the raw data scatterplot
(evoc_model1 <- evoc_scatter +
    # add the line from the created linear model
    # fortify fixes the model coefficients
    geom_line(data = fortify(model1),
              aes(x = numb_listened,
                  y = .fitted, colour = my.colours[4]),
              size = 2) +
    # add legend with renamed labels
  scale_colour_manual(name = 'Model Fit', label = 'n = 1', values = my.colours[4])
)
```
![](/Figures/tutorial_output_figures/evoc_model1.png)

Okayyyyyy... this does not look right :raised_eyebrow:

A <mark>straight line</mark> does not seem to fit our <mark>curved</mark> data.

What should we do now? :shrug:

There are several ways of how to deal with <mark>non-linear data</mark>.

A. We can make it :sparkles:**linear**:sparkles:.
	 This is called **data transformation**. *Check out [this tutorial](https://ourcodingclub.github.io/tutorials/data-scaling/) for more info.*

B. We could try to apply a - drumroll - :sparkles:**polynomial regression**:sparkles:.
	Now what is that????? *(visible confusion)*
	Lets check it out!

<a name="A-bit-of-Theory-on-Polynomial-Regression"></a>

## 5. A bit of Theory on Polynomial Regression

---------
Lets have a quick look at the theory behind `polynomial regression`:

* **what does it mean?**
* **when should I use it?**
* **what is the difference to `linear regression`?**

---------

**What does it mean?**

`Polynomial Regression` works similar to a linear regression but we could call polynomial regression the "bigger brother" (or sister...) of linear regression.

This is because we are adding some more terms to the regression model so that we can fit non-linear data. Depending on how many more terms we add we can model different <mark>degrees</mark> of curved data. The higher the degree, the more flexible will the data distribution and model become.

What do we mean with this <mark>degree</mark> term?

Lets look at the maths behind the model a bit.

From normal linear regression we know that the formula includes an `intercept` $\beta_0$ and a `slope` $\beta_1$ and we can write:

 > $y = \beta_0 + \beta_1 x$

 This is theoretically a polynomial model of <mark>degree 1</mark>. This is because technically the $x$ variable has an exponent of $1$. So we can say the degree is $n = 1$.

 How would a <mark>degree 2</mark> model look?

> $y = \beta_0 + \beta_1 x + \beta_2 x^2$

 You can see that the first part is exactly the same as the linear model but we have one more term $\beta_2 x^2$. The exponent of this term is $2$. So we can say the degree is $n = 2$.

 Note that we also have one more <mark>coefficient</mark> $\beta_2$ compared to the linear model. This introduces more flexibility into our model and allows us to toggle the model more to become more curve shaped.

 So if there is a <mark>degree 2</mark> model, there are probably a <mark>degree 3</mark>, <mark>degree 4</mark>, <mark>degree 5</mark> etc. models waiting around the corner.

 For every higher degree we just add one more term and increase the exponent. If you have figured it out by now, you could probably tell me how a <mark>degree 3</mark> model would look like :nerd_face:. - Or you could have a look at this table summarising the different degrees:

 |Degree|Formula|
 |--------|---------|
 | n = 1 | $y = \beta_0 + \beta_1 x$ |
 | n = 2 | $y = \beta_0 + \beta_1 x + \beta_2 x^2$ |
 | n = 3 | $y = \beta_0 + \beta_1 x + \beta_2 x^2 + \beta_3 x^3$|
 | n = 4 | $y = \beta_0 + \beta_1 x + \beta_2 x^2 + \beta_3 x^3 + \beta_4 x^4$  |
 | n = 5 | $y = \beta_0 + \beta_1 x + \beta_2 x^2 + \beta_3 x^3 + \beta_4 x^4 + \beta_5 x^5$  |
 | ... | ...|
 | n = n | $y = \beta_0 + \beta_1 x + \beta_2 x^2 + ... + \beta_n x^n$|

You can see that with each degree we are adding one more term  extending the formula further and further (*maybe until the galaxy far far away*).
The models are becoming more and more complex with each term until some term where it just gets too complex. That's why we usually only use polynomial models up to <mark>degree 5</mark> but not further.
Generally, the higher the degree the better will our model fit the data but it could at some point `overfit` the data but more on this later.

Lets see how these curves look visually:

![](/Figures/Concept_figures/polynomial_curves.jpeg)

[published by Aden Haussmann, 2020](https://towardsdatascience.com/polynomial-regression-the-only-introduction-youll-need-49a6fb2b86de)

You can see that with each degree the graphs become even more flexible.

During the modelling process we want to find the simplest model that describes the relationship of `x` and `y` to a suitable level. We want to find the most adequate number of  <mark>degrees</mark> $n$ and determine the $\beta$ coefficients.

--------

**When should I use it?**

So how do we find out if we should use `polynomial regression`?

We just did that earlier :tada:

The easiest way is to make a `Scatterplot` of the variables that you are interested in to see if the relationship between `x` and `y` is non-linear. Just as we did earlier.

Then there is one more **important** thing:  <mark>Check your assumptions</mark>.

We said that `polynomial regression` is just an  <mark>extension</mark> of `linear models`. This means that the <mark>same assumptions</mark> for linear models also apply to polynomial regression models. Remember these are:

* The `residuals` (errors) must be `normally distributed`, so in a bell-shaped histogram :bell:
* The `variation` of `residuals` must be `constant`, so our data must be `homoscedastic`
* `residuals` must be `independent` from each other, so that there is no systematic relationship between them
* `residuals` must be `independent` from the `x` variable and not affect the `y` variable

If you want to know more about these assumptions look at this [webiste](http://r-statistics.co/Assumptions-of-Linear-Regression.html).

Once we have developed our final model, we will also check these assumptions to see if we can actually use our model for the data but more later.

----------

 **What is the difference to `linear regression`?**

 You may already have some idea how `polynomial regression` is different or similar to `linear regression` but it is always good to have some overview of the differences in this table:

 | Simple Linear Regression                    |Polynomial Regression|
 |---------------------------------------------|------------------|
 | One predictor `x` and one response variable `y` | One predictor `x` and one response variable `y` |
 | assumes linear relationship between ``x`` and ``y`` | can account for non-linear relationships between ``x`` and ``y``|
 | $y = \beta_0 + \beta_1 x$ | $y = \beta_0 + \beta_1 x + \beta_2 x^2 + ... + \beta_h x^h$ $h$ is the degree of the polynomial|
 |![](https://user-images.githubusercontent.com/114161001/204134987-1c0b9323-f095-44ad-b29d-261056ad45fc.png) [published by Michy Alice, 2015](https://datascienceplus.com/fitting-polynomial-regression-r/)|![](https://user-images.githubusercontent.com/114161001/204135324-4a7fc757-e9b8-4249-a673-518eed506d04.png)[published by Michy Alice, 2015](https://datascienceplus.com/fitting-polynomial-regression-r/)|
 |<ul><li>linear relationships</li><li>assumptions must be bet</li><li>assuming a linear relationship can result in poor model fit</li></ul>| <ul><li>nonlinear data</li><li>**assumptions of linear models** must be met</li><li>extension of linear regression by adding polynomial terms of `x` ($x, x^2, x^3$) with increasing degree $h$</li></ul>|

--------

<a name="Develop-a-Polynomial-Model-of-Degree-2"></a>

## 6. Develop a Polynomial Model of Degree 2

Enough theory now :sweat_smile: Lets get dive right in and do some :sparkles:polynomial modelling:sparkles: in `R`.

As always there are different versions of how you can develop `polynomial regression` models in `R`.

The Figure below gives you an <mark>overview</mark> of the options you have:

![](/Figures/Concept_figures/Poly_reg_overview_r.png)

We will go through all of them just for good practice and that you know what you can do :smile:

#### **Version 1: Add terms with wrap function `I()`**

We can add polynomial terms by specifying them in our `lm` with a wrap function which is `I()`. This function wraps in additional terms and allows us to include them into the model.

:warning: You can easily forget the `I()` in your model and just enter the polynomial terms without it. Then `R` will not include the terms in the model and you would just get a linear model. Make sure you add the `I()` around your terms.

Lets check out the wrap function. We will also plot the "wrong" way by forgetting the `I()` just for illustration.

```r
# Polynomial Model of degree 2 ----

# Version 1: add terms with wrap function I()

# add a quadratic term

# WATCH OUT:
# you can easily forget the wrap function I() and accidantily write this code:
model2 <- lm(evoc_happiness ~ numb_listened + numb_listened^2, data = evocs)
# if you enter the code this way R will not include the quadratic term

# retrieve the model summary
summary(model2)

# RIGHT WAY:
# use I() function
model2a <- lm(evoc_happiness ~ numb_listened + I(numb_listened^2), data = evocs)

# retrieve the model summary
summary(model2a)
```
You can see that the output of `summary(model2)` without the `I()` is the same as a simple linear model because `R` excluded the quadratic term.
The output of `summary(model2a)` shows an additional term `I(numb_listened^2)` which is the <mark>quadratic</mark> term from the model.

**What does this  <mark>OUTPUT</mark> mean?**

![](/Data/tutorial_output_data/evocs_poly_model2_output.png)

* :one: **Call**: this gives you a summary of the model structure that you ran
* :two: **Residuals**: this is a summary of the minimum and maximum values and the quartiles with the median, so basically a [Boxplot](https://www.khanacademy.org/math/statistics-probability/summarizing-quantitative-data/box-whisker-plots/a/box-plot-review) of the dataset
* :three: **Coefficients**: now this is the interesting part - below all the parameters estimated by the model will be listed.
* :four: **Estimate**: model estimates of
	* the `Intercept` (*where you cross the y-axis, so you $\beta_0$*) and
	* the different ``slopes`` (*so your $)\beta_1, \beta_2...$*)
	 	* if they are *positive*, the `y` variable is increasing with `x`
		* if they are *negative*, the `y` variable is decreasing with `x`
* :five: **Std. Error**: standard error of the statistical estimates
* :six: **t Value**: tests the significance of the independent variables
* :seven: **p-value**: if p-value < 0.05 the model parameters are statistically significant
* :eight: **residual standard error**: estimates the average prediction error (standard deviation of the residuals)
* :nine: **Multiple R-Squared**: goodness of fit, how much variation in `y` is explained by our model
* :keycap_ten:**Adjusted R-Squared**: adjusts the R-squared for the number of independent variables
* :one::one: **F-statistic**: statistical measure comparing how much variation is explained by the model vs. how much variation remains unexplained so it gives information about the explanatory quality of our model
	* if ``F`` is *small*, than the explained variation is small compared to the unexplained variation;
	* if ``F`` is *large*, then the explained variation is large compared to the unexplained variation;
	* the **p-Value** of the **F-statistic** must be small if the model describes variation well
* :one::two: **degrees of freedom (DF)**: number of observations minus the number of estimated regression coefficients

#### **Version 2: Add terms with `poly()` function**

To save space and time we can specify the polynomial degree that we want to model in a function called `poly()`. Here we are using <mark>2</mark> because we are doing a <mark>quadratic</mark> model.

```r
# Version 2: add terms with the poly function

# specify the polynomial degree within the poly() argument
# because we make a quadratic fit we add 2
model2b <- lm(evoc_happiness ~ poly(numb_listened, 2), data = evocs)

# retrieve model summary
summary(model2b)
```
![](/Data/tutorial_output_data/evocs_poly_model2_output_orthogonal_poly.png)

If we look at the `summary(model2b)` output above, you might have noticed that we get different coefficient estimates for the `poly()` version model compared to the `I()` model.

Which one is the "right" version now? - *trick question* BOTH coefficient estimates are right.

The `poly()` function uses a slightly different way to estimate coefficients called *orthogonal polynomials*. We are not going to dig deeper on this topic since it would be beyond the scope of this tutorial but if you want to read more on *orthogonal polynomials* check out this [link](https://towardsdatascience.com/why-should-we-use-orthogonal-polynomials-b42b36f158a7).

We can fix this problem of different coefficients by adding `raw = TRUE` to the `poly()` model. This just says that the model will produce coefficient estimates based on `raw data` (*and not on othogonal polynomials*).

```r
# to fix this we can add raw = TRUE to the poly() function
# then the model will make coefficient estimates based on raw data which easier to interpret
model2b <- lm(evoc_happiness ~ poly(numb_listened, 2, raw = TRUE), data = evocs)

# retrieve model summary
summary(model2b)

# now the model outputs of model2a (wrap function) and model2b (poly function) are the same
```

Now the model summaries between `model2a` (`I()`) and `model2b` (`poly()`) should be the same :tada:

#### **Version 3: Use `cbind()` to save space**

We can also use the `cbind()` command to bind our explanatory variables together into one command. This lets us avoid the `I()` function which can get quite long as we will see later.

```r
# Version 3: use cbind() to save space

model2c <- lm(evoc_happiness ~ cbind(numb_listened, numb_listened^2), data = evocs)

# retrieve model summary
summary(model2c)

# gives the same summary as for the other models
```
And voliá! We get the same ``summary`` as for the other two versions.

Now lets look at a plot of our <mark>quadratic</mark> model to see if it fits the data better than our <mark>linear</mark> model.

```r
# plot the quadratic model onto the scatter plot with the linear model
(evoc_model2 <- evoc_model1 +
    # add the line from the created linear model
    # fortify fixes the model coefficients
    geom_line(data = fortify(model2a),
              aes(x = numb_listened,
                  y = .fitted, colour = my.colours[2]),
              size = 2,
              ) +
    scale_colour_manual(name = 'Model Fit', label = c('n = 1','n = 2'), values = c(my.colours[4], my.colours[2]))
)
```
![](/Figures/tutorial_output_figures/evoc_model2.png)

Now we can see that the <mark>linear</mark> and <mark>quadratic</mark> model plots do not differ much. The model estimate of our raw data still looks quite poor.

Maybe a <mark>higher degree</mark> polynomial model could fix this and help us out to estimate evoc happiness for our galactic friends :milky_way:


<a name="Adding-more-Terms-Polynomial-Models-of-higher-Degrees"></a>

## 7. Adding more Terms: Polynomial Models of higher Degrees

----

>**Quick Recap**:
>
> * We saw that a $n=1$ (<mark>linear model</mark>) and a $n = 2$ <mark>quadratic model</mark> both did not fit our data well.
>* We know that we will probably need a $n > 2$ ()<mark>higher degree model</mark>) to fit our data adequately.
>* BUT: we also know that if we add <mark>too many degrees</mark>, the model will get super complex.
>
> * :arrow_right: now we need to find out to <mark>what degree</mark> we need to extend our model to fit our data well but also to still be simple enough.

----

We will do this by building several polynomial models of increasing degrees from $n = 1$ to $n = 5$ (*beyond that our model would loose predictive power as it gets too complex*).

#### **Version 1: Add terms with wrap function `I()`**

Lets start with the `I()` wrap function again:

```r
# Polynomial Models of higher degrees ----

# Develop 5 models from degree n = 1 to degree n = 5

# Version 1: with wrap function

# linear model (same as above) n = 1
model1 <- lm(evoc_happiness ~ numb_listened, data = evocs)
# quadratic model (same as above) n = 2
model2 <- lm(evoc_happiness ~ numb_listened + I(numb_listened^2), data = evocs)
# cubic model n = 3
model3 <- lm(evoc_happiness ~ numb_listened + I(numb_listened^2) + I(numb_listened^3), data = evocs)
# n = 4
model4 <- lm(evoc_happiness ~ numb_listened + I(numb_listened^2) + I(numb_listened^3) + I(numb_listened^4), data = evocs)
# n = 5
model5 <- lm(evoc_happiness ~ numb_listened + I(numb_listened^2) + I(numb_listened^3) + I(numb_listened^4) + + I(numb_listened^5), data = evocs)
````

#### **Version 2: Add terms with `poly()` function**

Remember we can also use the `poly()` function with the `raw = TRUE` argument to get the same results. We will try this version too to illustrate how to model higher degree polynomials.

You will see that `poly()` is more compact and saves time. Keep in mind though that you can easily loose track of your actual model structure with this compact version. The `I()` function allows you to see what your model looks like which might be easier to interpret results later.

```r
# Version 2: with poly function

# linear model n = 1
model2b <- lm(evoc_happiness ~ poly(numb_listened, 1, raw = TRUE), data = evocs)
# quadratic model (same as above) n = 2
model2b <- lm(evoc_happiness ~ poly(numb_listened, 2, raw = TRUE), data = evocs)
# cubic model n = 3
model3b <- lm(evoc_happiness ~ poly(numb_listened, 3, raw = TRUE), data = evocs)
# n = 4
model4b <- lm(evoc_happiness ~ poly(numb_listened, 4, raw = TRUE), data = evocs)
# n = 5
model5b <- lm(evoc_happiness ~ poly(numb_listened, 5, raw = TRUE), data = evocs)

# again this is the same as with the wrap I() function
# we will use the wrap function models
```


-------------
</div>

<div style="background-color:rgba(177, 115, 241, 0.5)">

## For the very keen people...

#### **Version 3: Use a ``for`` loop and `poly()` to add terms**

We can also use a `for` loop to build all 5 models in one go and save loads of time.

If you do not know how `for` loops work, do not worry! This [tutorial](https://ourcodingclub.github.io/tutorials/funandloops/) will help you out.

This bit of code now is not really necessary for you to complete the tutorial but still may be interesting :smile:

````r
# Version 3: use a for loop and poly function

# use a for loop to make the models and save time copying and pasting

# create an empty list to store the model output in
models <- list()

# specify how many degrees we want to model
# here we develop models from degree 1 (linear) to degree 5
degree <- 5

# write the loop
# it will loop through the model 5 times because we want 5 different models
# in each loop we will apply a different degree from 1 to 5
# we will store the model outputs of each loop in the models list created earlier
for (i in 1:degree) {
  # fit model
  model <- lm(evoc_happiness ~ poly(numb_listened, i, raw = TRUE), data = evocs)
  # store model in list
  models[[i]] <- model
}

# to extract model summaries from the for loop run for example
summary(models[[2]])
# here we extract model number 2 (degree = 2)
````
This code has stored all our models in a `list` called models and has saved us loads of variable space and time. Again, this is just if you want to dig deeper on the topic.

</div>

---------------

<a name="How-do-you-Decide-which-Model-to-use?"></a>

## 8. How do you decide which model to use?

Now we have loads of different models with different <mark>degrees</mark>. How do we decide which model fits our data best while still making sense?

Again, we have several options as outlined by the figure below.

![](/Figures/Concept_figures/which_model.png)

#### **Option A: Visual Assessment**

Lets look at this visually. Similarly to the <mark>linear</mark> and <mark>quadratic</mark> model comparisons we are going to plot all the different models onto the raw data. Since that can get quite messy we will produce an extra plot for each model.

The code below seems a bit daunting since it is so big but is basically repeating the same bits of code just 5 times for each model. (*of course we could make the plots in a `for` loop but I did not want to put you through this again*)

````r
# Assess model Fit ----

# Version 1: Visually

# add the different model fits to the raw data scatter plot

# for model 1 n = 1
(evoc_model1_plot <- evoc_scatter +
   # add the line from the created linear model
   # fortify fixes the model coefficients
   geom_line(data = fortify(model1),
             aes(x = numb_listened,
                 y = .fitted), colour = my.colours[4],
             size = 2) +
   # add the model degree with text annotation
   annotate('text', x = 1, y = Inf, vjust = 2, label = 'n = 1', size = 7, fontface = 2))

# for model 2 n = 1
(evoc_model2_plot <- evoc_scatter +
    # add the line from the created linear model
    # fortify fixes the model coefficients
    geom_line(data = fortify(model2),
              aes(x = numb_listened,
                  y = .fitted), colour = my.colours[2],
              size = 2) +
    # add the model degree with text annotation
    annotate('text', x = 1, y = Inf, vjust = 2, label = 'n = 2', size = 7, fontface = 2))

# for model 3 n = 1
(evoc_model3_plot <- evoc_scatter +
    # add the line from the created linear model
    # fortify fixes the model coefficients
    geom_line(data = fortify(model3),
              aes(x = numb_listened,
                  y = .fitted), colour = my.colours[1],
              size = 2)+
    # add the model degree with text annotation
    annotate('text', x = 1, y = Inf, vjust = 2, label = 'n = 3', size = 7, fontface = 2))

# for model 4 n = 1
(evoc_model4_plot <- evoc_scatter +
    # add the line from the created linear model
    # fortify fixes the model coefficients
    geom_line(data = fortify(model4),
              aes(x = numb_listened,
                  y = .fitted), colour = my.colours[3],
              size = 2) +
    # add the model degree with text annotation
    annotate('text', x = 1, y = Inf, vjust = 2, label = 'n = 4', size = 7, fontface = 2))

# for model 5 n = 1
(evoc_model5_plot <- evoc_scatter +
    # add the line from the created linear model
    # fortify fixes the model coefficients
    geom_line(data = fortify(model5),
              aes(x = numb_listened,
                  y = .fitted), colour = my.colours[5],
              size = 2) +
    # add the model degree with text annotation
    annotate('text', x = 1, y = Inf, vjust = 2, label = 'n = 5', size = 7, fontface = 2))

		````
Now we have all the plots separately but to actually compare the models lets arrange them onto one page.

```r
# arrange the plots in a gird
evoc_model_compare <- grid.arrange(evoc_model1_plot, evoc_model2_plot, evoc_model3_plot, evoc_model4_plot, evoc_model5_plot)

# save the plot
# save the plot to the directory you want to save the plot to
ggsave(evoc_model_compare, filename = 'Figures/tutorial_output_figures/evoc_model_compare.png',
       height = 20, width = 30, unit = 'cm')
````
The plot should look something like this:

![](/Figures/tutorial_output_figures/evoc_model_compare.png)

We can see that the <mark>$n = 3, n = 4, n = 5$</mark> models fit the data way better than the <mark>linear and quadratic<mark> degree models. So our model has to be at least of <mark>degree 3<mark>. But how do we decide if it is <mark>degree 3, 4 or 5<mark>? - with an :sparkles:``ANOVA``:sparkles:

#### **Option B: Statistical Assessment: ``ANOVA``**

``ANOVA`` means <mark>Analysis of Variance</mark> and is also a subform of linear model. If you want to know more about `ANOVA` go to this [tutorial here](https://ourcodingclub.github.io/tutorials/anova/).

Lets do an `ANOVA` to compare the different model fits. We will test whether a simpler model of <mark>lower</mark> degree is sufficient enough to explain the data or if we require a <mark>higher</mark> degree model. So basically we are comparing if $n=2$ or $n=3$ are better and then if $n=3$ or $n=4$ are better and so on.
You can see that the models must be nested within each other to be comparable with an `ANOVA`. This just means that the <mark>lower degree</mark> model must be stated before the <mark>higher degree</mark> model since the higher degree model kind of includes the lower degree model but with one more term.

````r
# Version 2: Anova of the models

anova(model1, model2, model3, model4, model5)

````

This gives following output:

![](/Data/tutorial_output_data/anova_output.png)

What does this tell us? :thinking:

* :one: **Analysis of Variance Table**: here all the models that we built are listed again to give us an overview. You can see that *Model 1* is embedded in *Model 2* which is part of *Model 3* etc.
* :two: **Res.Df**: describes the `Degrees of Freedom` from each model. Remember that was the number of data points minus the number of estimated coefficients.
* :three: **RSS**: ``residual sum of squares`` or sum of squared errors is the amount of <mark>error unexplained</mark> by the models. You can see that RSS decreases with increasing <mark>degree</mark>. This is because more and more of the data is explained by the model.
* :four: **Degrees of freedom**: These are `1` here because we are only comparing models pairwise.
* :five: **Sum of Squares**: the ``sum of squares`` are in a way the <mark>opposite</mark> of the ``residual sum of squares`` and describe how much of the data is described by the model. They should be <mark>highest</mark> for the <mark>best fitting</mark> model.
* :six: **F-value**: this is the ratio of <mark>explained</mark> to <mark>unexplained</mark> variation and should be mark>highest</mark> for the <mark>best fitting</mark> model.
* :seven: **p-value**: the ``p-value`` tells us whether the model is sufficient to describe the data or if we need a higher degree model. *For example*
 	* if the p-value of *Model 2* is > 0.05 this means that *Model 2* is not sufficient to describe the data and *Model 3* may represent the data better.
   * if the p-value of *Model 2* is < 0.05 the more complex model *Model 3* is unnecessary and we can use the simpler model of lower degree
   * if p-values of *Model 2* and *Model 3* are similar this means that either of the models could provide a reasonable fit to the data

Our ``ANOVA`` shows that `Model 3` has the lowest ``p-value`` (< 2e-16 so super small). All other `Models` have ``p-values`` > 0.05. This means they are probably not the best fits to the data. For `Model 1` and ``Model 2`` this is not surprising since we already saw in our earlier plots that the <mark>linear</mark> and <mark>quadratic</mark> models did not fit well with the data. ``Model 4`` and ``Model 5`` could have been good fits according to our plot above. Remember though that we are trying to fit a <mark>simple</mark> model rather than a super <mark>complex</mark> model. That is also why `Model 3` is probably the best choice. Note that ``Model 3`` also has the highest `´Sum of Squares` and the best ``F-statistic`` confirming our model choice! :tada: :party_face:


<a name="Check-your-Assumptions"></a>

## 9. Check your Assumptions

Okay now we have our model: **Model 3**.

But remember one thing: we still need to... :drum:**Check our Assumptions**:drum:

We can assess this in several ways. If you need another refresher on model assumptions, have a look at this [tutorial](https://ourcodingclub.github.io/tutorials/modelling/).

**Check if the residuals (errors) are normally distributed**

Lets extract the <mark>residuals</mark> from ``Model 3`` and plot them in a `histogram`. We can extract the residuals with the command ``residuals()``.

````r
# extract residuals from model 3
model3_res <- data.frame(residuals(model3))

# histogram of the residuals of the model
(resid_hist <- ggplot(data = model3_res) +
    # create a histogram
    geom_histogram(aes(x = residuals.model3.),
                   # change transperancy, binwidth and colours
                   alpha = 0.4,
                   breaks = seq(-110, 100, by = 10),
                   fill = my.colours[5], colour = my.colours[5]) +
    # add customised function
    theme.ggplot() +
    # rename axis
    xlab('Model Residuals') +
    ylab('Count'))
````
![](/Figures/tutorial_output_figures/resid_hist.png)

This looks nice and bell shaped :bell: so we can check off the assumption of normally distributed residuals.

**Check for homoscedasticity and residual independence of your data**

Now lets check our other assumptions with `plot()`.
This will produce following plots:
>  * **a residual vs. fitted plot**: this will check if our data is <mark>homoscedastic</mark>
>  * **a Normal QQ Plot**: this will check if the <mark>residuals</mark> are <mark>normally distributed</mark> (so the same job as the histogram)
>  * **a scale-location plot**: this will also check if our data is <mark>homoscedastic</mark>
>  * **a residuals vs. leverage plot**: this checks whether our <mark>residuals</mark> are independent of the `Y` and `X` variables

Lets check:

```r
# assess other linear model assumptions with plot()

# create a grid with 2 rows and 2 columns
par(mfrow = c(2,2))
# this command produces
  ## a residual vs. fitted plot
  ## a Normal QQ Plot
  ## a scale-location plot
  ## a residuals vs. leverage plot
plot(model3)

# all looks good :)

beep(sound = 8, expr = NULL)
```
![](/Figures/tutorial_output_figures/model3_assumpt.png)

All plots show that the <span style='color:red'> red </span> line is somewhat straight and that all the dots fall on a straight line for the `QQ-Plot`.

This means we can confirm our assumptions of `linear models`.
`Model 3` is actually adequate to represent the relationship of `numb_listened` to ``evoc_happiness`` :notes: :smile:


<a name="Save-and-Report-your-Results"></a>

## 10. Save and Report your Results

This is great! We have our model and we have something to give our friends in the galaxy far far away :star: :moon:

But they probably do not have much time as they are probably learning how to lift spaceships out of swamps :rocket: or how to avoid getting eaten by desert plants :seedling:. So it would be great if we could present them with a plot of our results.

Lets make our plot of the final model. For that we will need:

  > * the raw data
  > * the model fit
  > * some uncertainty measure (<mark>Confidence Intervals (ci)</mark>) *(if you do not know what confidence intervals are, this [website](https://statisticsbyjim.com/hypothesis-testing/confidence-interval/) gives a good summary)*

We have the raw data and the model fit but we still need our Confidence Intervals. We can extract them from our `Model 3` with the `predict()` command.

````r
# model fit and confidence intervals
# extract model fit and confidence intervals with predict() from the model
# level specifies the width of the interval
model3_ci <- predict(model3, interval = 'confidence', level = 0.99)
````
Now lets add the extracted ``model3_ci`` Confidence Intervals to our ``evocs`` data to create the dataframe that we will use in our final plot.

````r
# create dataframe combining raw data, model fit and ci

# add fit and the extracted intervals to the the evoc raw data
evocs_final <- cbind(evocs, model3_ci)
````
Now we have all the ingredients to produce our final plot for the rebel base :star:

````r
# plot the model and confidence intervalls together with the raw data

(evoc_final <- ggplot(data = evocs_final, aes(x = numb_listened)) +
    # create a scatter plot
    geom_point(# modify point attributes
               aes(y = evoc_happiness, colour = my.colours[5]),
               size = 3, pch = 21, stroke = 1) +
    # add the line from the chosen model
    geom_line(data = fortify(model3),
              aes(y = .fitted, colour = my.colours[1]),
              size = 1) +
    # add confidence intervals from the data frame
    geom_line(aes(y = lwr, colour = my.colours[1], linetype = 'Confidence Interval'),  
              # modify line attributes
              size = 1) +
    geom_line(aes(y = upr, colour = my.colours[1], linetype = 'Confidence Interval'),
              # modify line attributes
              size = 1) +
    # add legend with renamed labels
    scale_colour_manual(name = 'Model of Evoc Happiness',
                        values = c(my.colours[5], my.colours[1]),
                        labels = c('raw data', 'n = 3')) +
    # add legend for the confidence intervals
    scale_linetype_manual(name = NULL,
                          values = c(3),
                          labels = c('Confidence Interval')) +
    # change the aesthetics of the legend
    guides(color = guide_legend(override.aes = list(linetype = c(0, 1),
                                                    shape = c(21, NA))),
           linetype = guide_legend(override.aes = list(colour = my.colours[1]))) +
    # add customised function
    theme.ggplot() +
    # rename axis
    xlab('Listening to the Song') +
    ylab('Evoc Happiness') +
    # place legend position below plot
    theme(legend.position = 'bottom'))

````

![](/Figures/tutorial_output_figures/evoc_final.png)

The <span style='color:blue'> blue </span> dots show the raw data points and the <span style='color:red'> red </span> solid line the `polynomial regression fit` of <mark>degree $n = 3$</mark>. The dotted <span style='color:red'> red </span> lines are the <mark>upper and lower confidence intervals</mark>.

We can see that the <mark>$n = 3$</mark> model represents the relationship of ``numb_listened`` to ``evoc_happiness`` better than the <mark>degree $n = 1, 2...$</mark> models.
Evoc happiness really seems to be increasing with the number of times a song is played to them :smile: :notes: :bear:

Lets save the plot to send it across the universe :rocket:

````r
# save the plot
# save the plot to the directory you want to save the plot to
ggsave(evoc_final, filename = 'Figures/tutorial_output_figures/evoc_final.png',
       height = 15, width = 20, unit = 'cm')
````

We would also like to send some more quantitative measures of our model. So lets look at the model ``summary()`` again and save the main output.


We will do this with the `broom` package which is super useful to clean up model data. You can read more on the package [here](https://cran.r-project.org/web/packages/broom/vignettes/broom.html).

````r
# look at the model summary again
summary(model3)

# save the coefficients and statistics for later use
# save the coefficients with tidy() and store in a csv file in your preferred folder
write.csv(tidy(model3), 'Data/tutorial_output_data/coefs_model3.csv')

# save additional summary statistics with glance() and store in a csv file in your preferred folder
write.csv(glance(model3), 'Data/tutorial_output_data/an_model3.csv')
````

Amazing :star_struck: If you go to the folder where you saved your ``.csv`` files to, you should be able to open them and see a summary of the model output. Something like this:

![](/Figures/Concept_figures/screenshot_csv_output.png)

We would also like to send a small summary note of our model results just to make it complete.
This will also answer our **research question**!
Also remember our initial **hypothesis**:

:exclamation: Evoc happiness increased with the number of times they have listened to a song.

----------------
>
><div style="background-color:rgba(255, 210, 0, 0.5)">
>
> ## Results on Evoc Happiness
>
>We found that the number of times Evocs have been listening to a song increased Evoc Happiness (**Polynomial Model, degree = 3, F = 1606, DF = 3, 197, p = 2.2e-16**).
>
>The relation of Evoc happiness `y` to the number of times ``x`` listened to a song could have been approximated in the following way:
>
>$y = 6.99 + 152.69x - 15.08x^2 + 0.50x^3$
>
></div>
>
-------------

And this is the :sparkles:end:sparkles: of all the hard work on `polynomial regression modelling`!

We hope we could give our galactic friends :star: some insight on Evoc happiness :bear: and how they respond to music :notes:

We have successfully mastered `polynomial regression`! Congrats! :tada:

````r
# sound on :)
beep(sound = 3, expr = NULL)
````

<a name="A-word-of-Caution"></a>

## 11. A word of Caution

We have touched upon some problems with ``polynomial regression`` during the tutorial and here are some general things to watch out for when building the model.

:warning: **The risk of Overfitting**

* <mark> Overfitting</mark> means that you include more terms in your model than your data can actually support.

* More terms will make your model more *exact* but also more *complex*. Sometimes your model can become too exact and will start picking up *noise* (you can read more about this [here](https://www.javatpoint.com/what-is-noise-in-data-mining)) and model every single fluctuation in the data.

* Your model may end up looking like this:

![](/Figures/Concept_figures/overfitting.png)

* So your model may be getting better and better in fitting the *existing* data but if you try to predict *new* data, the model may be <mark>not reliable</mark>.

* Basically: the most exact model fit is not necessarily the best fit (see this [video](https://www.youtube.com/watch?v=DmYV5cma7cQ) for an explanation)

Make sure that you are not overfitting when building a model for non-linear data. Sometimes **other modelling approaches** may be better than **polynomial regression**. Check out what alternatives are available [here](https://ourcodingclub.github.io/tutorials.html).

This was the very :drum:last:drum: comment and I hope you are less intimidated by polynomial regression now and may go out onto your own galactical modelling endeavours :rocket: :milky_way: :smile:

<a name="Learning-Outcomes"></a>

## 12. Learning Outcomes

--------------

You should now know...

* how to explore your data with scatterplots and histograms
* how to develop simple linear models
* what the theory behind polynomial regression models is
* how to build polynomial regression models with ``I()``, ``poly()``, ``cbind()`` and ``for`` loops
* how to assess model fit visually and with ``ANOVA`` tests
* how to check the assumptions of linear models with ``histograms`` of the residuals and by producing and interpreting `statistical plots``
* what the components of ``summary()`` output of polynomial models and `ANOVAS` means
* how to export and report your modelling results in tables and graphs
* what impact and risks of *overfitting* are

--------------

<a name="Further-Resources"></a>

## 13. Further Resources

<div style="background-color:rgba(0, 0, 0, 0.1)">

Here you can find further interesting resources:

| Topic | Content | Link |
|------|--------|------|
|Intro to Polynomial regression| Description of maths, concepts, drawbacks of polynomial regression vs linear regression | https://www.statology.org/polynomial-regression/|
|Polynomial Regression in R| Step-by-step description of how to do polynomial fitting in R with a foor loop and k-fold comparison | https://www.statology.org/polynomial-regression-r/|
|Getting started with polynomial regression in R|Tutorial in how to perform polynomial regression in R simple version without assessing model fit|https://www.section.io/engineering-education/polynomial-regression/|
|Fitting polynomial regression in R|simple tutorial on how to use polynomial regression in R based on economics data and already known relationships of the data|https://datascienceplus.com/fitting-polynomial-regression-r/|
|Orthogonal Polynomials|Description of what orthogonal polynomials are and how they are important in polynomial regression|https://towardsdatascience.com/why-should-we-use-orthogonal-polynomials-b42b36f158a7|
|An Introduction to Statistical Learning|Theory description of polynomial regression, k-fold cross validation and applied Lab with R codes|https://static1.squarespace.com/static/5ff2adbe3fe4fe33db902812/t/6009dd9fa7bc363aa822d2c7/1611259312432/ISLR+Seventh+Printing.pdf|
|Polynomial Regression and Step Functions|Step by step tutorial on polynomial regression in R following the Introduction to Statistical Learning book|http://www.science.smith.edu/~jcrouser/SDS293/labs/lab12-r.html|
|Building an end-to-end Polynomial Regression Model in R|detailed tutorial with good mathematical explanation of model outputs on polynomial regression|https://www.analyticsvidhya.com/blog/2021/11/building-an-end-to-end-polynomial-regression-model-in-r/|
|Transformations, polynomial fitting and interaction terms|powerpoint presentation on different statistical lm concepts|https://www.southampton.ac.uk/~mb1a10/stats/FEEG6017_lecture-Interaction_terms_etc_brendan.pdf|
|Polynomial regression in R|Video on how to build regression models in R (6 min)|https://www.youtube.com/watch?v=ZYN0YD7UfK4|
|Polynomial regression in R|Video on choosing the right regression model in R (11 min)|https://www.youtube.com/watch?v=DmYV5cma7cQ|
|Polynomial regression in Rstudio|Video buidling on polynomial regression and how to programme it in RStudio (9 min)|https://www.youtube.com/watch?v=gNkTydo6b-A|
|Coding Club Tutorials| List of different Coding Club Tutorials|https://ourcodingclub.github.io/tutorials.html|

</div>
