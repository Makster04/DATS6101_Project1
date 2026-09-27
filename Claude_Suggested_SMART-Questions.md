
The template:

> *Over [time window], is the relationship between [column X] and [column Y] across metros different in the bottom 100 metros than in the top 100?*

## Three ways to test it** (pick one per question)

- **Compare correlations (Fisher's z-test).** Compute the correlation between X and Y separately for each group, then test whether the two correlations differ. Show one scatterplot with the two groups in different colors and a fitted line for each.
- **Two-way ANOVA with an interaction.** Split X into high and low at its median, then run `aov(Y ~ X_level * group)`. The interaction p-value answers "does X play a different role in small metros?"
- **Regression with an interaction.** Fit `lm(Y ~ X * group)`. The interaction coefficient tells you how much the slope differs between groups. This is the most direct option, if your course has covered `lm`.

**Column pairs that work** (Jun 2023–Aug 2026, one value per metro):

**1. Supply vs. price**
Is the link between percent change in `active_listing_count` and percent change in `median_listing_price` (Aug 2023 → Aug 2026) weaker in the bottom 100? In small markets, a few extra listings may not move prices the way they do in big ones.

**2. Slow sales vs. price cuts**
Is `median_days_on_market` more strongly tied to `price_reduced_share` in the bottom 100 than in the top 100 (Sep 2025–Aug 2026 averages)? In other words, do sellers in small markets respond differently when homes sit longer?

**3. Price cuts vs. actual price change**
Does a higher `price_reduced_share` go along with larger price declines to the same degree in both groups?

**4. Home size vs. price**
Does `median_square_feet` explain `median_listing_price` more strongly in the bottom 100? In big metros, location may matter more than size.

**5. Demand vs. price growth**
Is `pending_ratio` (pending listings per active listing, a demand signal) more strongly linked to price change in the top 100 than in the bottom 100?

**Tips:**

- **One point per metro.** Average or compute the change for each metro first, so each group has 100 independent points.
- **Expect limited power.** With 100 metros per group, two correlations generally need to differ by about 0.3 before the test comes out significant. A non-significant result is still a valid finding, as long as you report it that way.
- **Transform skewed columns.** Listing counts and their percent changes will be skewed, especially in small metros. Log-transform them, or use Spearman's correlation and say that the Fisher test is approximate for Spearman.
- **The strongest pick:** question 1 uses the core supply-and-demand idea, is easy to explain in a presentation and has a clear visual in the two-line scatterplot.Yes, all of them work. One clarification: ggplot isn't a separate kind of chart. **ggplot2** is the R package you'd use to make every chart below, so plan to build all your graphs with it.

## IF WE WANTED DISTRIBUTION GRAPHS TOO:
Here is how each question you already have maps to a chart:

| Chart | What it shows | Questions it fits | ggplot2 code |
|---|---|---|---|
| **Histogram / density** | Shape of one column: skew, outliers, whether it looks normal | Inventory growth (shows the skew in small metros and why you'd log-transform or use Wilcoxon); price growth, top vs. bottom | `geom_histogram()` or `geom_density()` + `facet_wrap(~group)` |
| **Box plot / violin** | Two or more groups side by side, including medians, spread and outliers | Days on market top vs. bottom; price cuts by region (ANOVA); days on market by season; price volatility (shows which group spreads out more) | `geom_boxplot()`, `geom_violin()` |
| **Q-Q plot** | Normality check before a t-test or Pearson correlation | Any t-test or correlation question | `stat_qq()` + `stat_qq_line()` |
| **Scatterplot, colored by group, one fitted line per group** | Whether X relates to Y differently in each group | All the "different role" questions, especially supply vs. price | `geom_point(aes(color=group))` + `geom_smooth(method="lm")` |
| **Interaction plot** | Group means at low and high X, one line per group; lines that aren't parallel mean an interaction | Two-way ANOVA version of the "different role" questions | `stat_summary()` + `geom_line()` |
| **Stacked / proportion bar** | Yes/no share by group | Falling prices, top vs. bottom (chi-square) | `geom_bar(position="fill")` |
| **Line chart over time** | Monthly trend for top 100 vs. bottom 100 | Opening slide showing the whole Jun 2023–Aug 2026 story; seasonality | `geom_line()` on the monthly median per group |

**The questions with the most visual payoff:**

- **Supply vs. price, top vs. bottom.** Use a histogram of each variable to show the skew, then the two-line scatterplot to show the interaction.
- **Price volatility.** Box plots of each metro's standard deviation make the "small metros swing more" idea easy to see before you run the variance test.
- **Days on market.** A monthly line chart shows trend and seasonality, and a box plot sets up the t-test.

**Tips:**

- **Put a chart in front of every test.** Show the box plot, then the t-test, or the scatterplot, then the correlation. This covers both "graphical representations" and "statistical tests" in the rubric, and it makes the slides easy to follow.
- **Use the same axis scales across groups.** Keep `facet_wrap()` scales fixed so the top 100 and bottom 100 panels can be compared directly.
- **Use log scales for counts and prices.** `scale_x_log10()` keeps a few huge metros from squashing everything else into a corner.
- **Aim for about 8–10 charts in the write-up.** A well-chosen chart for each question works better than showing every variable.
