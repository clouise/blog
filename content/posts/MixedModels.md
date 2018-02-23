---
title: "Mixed Effects Models"
date: 2018-02-22
draft: False
#featuredImage : "img/dndprob.jpg"
---

Introduction
------------

This all started when I was doing work for a company that wanted to
understand the impact of some metric on total their revenue. To
determine this, I first built a linear regression that modeled the
metric against total revenue. However, during this analysis I realized
this seemingly simply problem had a bit more depth to it. While
completing this analysis, I ended up learning some cool stuff about
mixed effects models.

What follows is an imaginary problem with data that I created myself to
demonstrate some of these learnings.

Problem
-------

Let’s say we worked for a software company. The company tasked us with
finding out if the number of sick days the sales people took impacted
their total revenue. We could solve that problem with a linear model.

    ggplot(sales, aes(x = SickDaysTaken, y = Revenue)) +
      geom_point() +
      geom_smooth(method = "lm", se = FALSE) + scale_y_continuous(labels = comma) + ggtitle("Figure A")

![](/MixedModels_files/figure-markdown_strict/simple_lm_plot-1.png)

    lm <- lm(Revenue ~ SickDaysTaken, data = sales)
    summary(lm)

    ##
    ## Call:
    ## lm(formula = Revenue ~ SickDaysTaken, data = sales)
    ##
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max
    ## -454896 -148865     249  153016  506604
    ##
    ## Coefficients:
    ##               Estimate Std. Error t value            Pr(>|t|)
    ## (Intercept)     754259      24688   30.55 <0.0000000000000002 ***
    ## SickDaysTaken   -19830       1746  -11.36 <0.0000000000000002 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ##
    ## Residual standard error: 197300 on 406 degrees of freedom
    ## Multiple R-squared:  0.2412, Adjusted R-squared:  0.2393
    ## F-statistic:   129 on 1 and 406 DF,  p-value: < 0.00000000000000022

I've included the R output above, but feel free to skip over it. Figure
A above is the plot of the simple linear model we built. It shows that
for each additional sick day a sales person takes, the company loses
$19,830.07. (Don’t worry, I checked my assumptions for linear models
outside of this blog post). But what if the company we worked for had
multiple headquarters each in different regions around the world. Lets
plot the data again, this time using color to identify the different
regions.

    ggplot(sales, aes(x = SickDaysTaken, y = Revenue, colour = Region)) +
      geom_point(size = 3) + scale_y_continuous(labels = comma) + ggtitle("Figure B")

![](/MixedModels_files/figure-markdown_strict/regions-1.png)

Figure B tells a different story than Figure A. While it might be hard
to see, each region is clustered relatively tightly on the graph. For
example, the South region (blue) is only seen on the top left of the
graph. This implies that these regions behave differently from one
other. We can can see this more clearly if we look at a box plot of the
data.

    ggplot(data=sales,(aes(x=Region, y=SickDaysTaken, fill=Region))) + geom_boxplot() + ggtitle("Figure C")

![](/MixedModels_files/figure-markdown_strict/boxplot-1.png)

Figure C also shows that the number of sick days taken vary with region.
While we could plot revenue by region, I can assure you that this also
varies. This variance shows that it isn't fair to compare different
regions if they behave quite differently. What we really want to know is
the relationship between sick days taken and revenue while accounting
for the region. A naive way of doing this is to include the region
within our linear model, shown below.

    lm2 <- lm(Revenue ~ SickDaysTaken + Region, data = sales)
    summary(lm2)

    ##
    ## Call:
    ## lm(formula = Revenue ~ SickDaysTaken + Region, data = sales)
    ##
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max
    ## -433104 -100628   -1153   89688  527201
    ##
    ## Coefficients:
    ##                 Estimate Std. Error t value             Pr(>|t|)
    ## (Intercept)       458253      44284  10.348 < 0.0000000000000002 ***
    ## SickDaysTaken      -3210       2486  -1.291               0.1975
    ## RegionEast        329350      36076   9.129 < 0.0000000000000002 ***
    ## RegionEurope      199551      26448   7.545    0.000000000000308 ***
    ## RegionNorthEast   -64315      27459  -2.342               0.0197 *
    ## RegionSouth       243777      35964   6.778    0.000000000043705 ***
    ## RegionSouthEast  -113956      28444  -4.006    0.000073556336767 ***
    ## RegionWest        -67153      28209  -2.381               0.0178 *
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ##
    ## Residual standard error: 144000 on 400 degrees of freedom
    ## Multiple R-squared:  0.6015, Adjusted R-squared:  0.5945
    ## F-statistic: 86.25 on 7 and 400 DF,  p-value: < 0.00000000000000022

The number of sick days taken is no longer significantly related to
revenue. Instead, the region is most significant. Being in the East
region means you’ll make $329,350 more. This no longer answers our
question. This model tells us how much revenue varies between regions.
We want to know the impact of sick days taken on revenue, while taking
into account the variance of each region.

Mixed Effects Models
--------------------

Here’s where mixed effects models come in. This type of model allows us
to control for the effects of region on our analysis. The code specified
below, builds a model that allows the intercept of our linear regression
to vary by region, but ensures that the slope of the lines are the same
between all regions.

    mixed.lmer <- lmer(Revenue ~ SickDaysTaken + (1|Region), data = sales)

    ggplot(sales, aes(x = SickDaysTaken, y = Revenue, colour = Region)) +
      geom_point(size=1) +
      geom_line(aes(y = predict(mixed.lmer)),size=1) + scale_y_continuous(labels = comma) + ggtitle("Figure D")

![](/MixedModels_files/figure-markdown_strict/mixed_model-1.png)

Figure D shows that for each sick day taken the company loses $3,904.39
when accounting for the variability of the regions. If we didn't run the
analysis this way, we might be tempted to assume that we are losing
close to $20,000 per sick day.

Why Mixed Effects Models
------------------------

We could have run this analysis another way, by fitting a linear
regression for each region. In this scenario we have seven regions, so
we would build seven linear models. This is shown in Figure E, below.

    ggplot(sales, aes(x = SickDaysTaken, y = Revenue, colour = Region, group = Region)) +
      geom_point(size = 1) + geom_smooth(method="lm", se = FALSE)  + scale_y_continuous(labels = comma) + ggtitle("Figure E")

![](/MixedModels_files/figure-markdown_strict/lm_region-1.png)

Notice that Figure E differs from the mixed model in that both the
intercepts and slopes vary. With this methodology, sick days decrease
revenue in the Eastern region, but slightly increase revenue in the
Western region. If we build a linear model for each region, we are also
limited by the number of regions and the sample size. Say each region
had five divisions within it. We would need to build 35 linear models
(seven regions \* five divisions). This isn't computationally difficult,
but with each segmentation, we are lowering our sample size. With a
mixed effects model we can use the entire sample size while accounting
for the variability of different groups.
