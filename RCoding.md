```{r}
metro_dom_volatility <- postCOVID_housing_without_QF %>%
  group_by(cbsa_code) %>%
  summarise(cbsa_title          = first(cbsa_title),
            household_size_tier = first(household_size_tier),
            avg_days_on_market  = mean(median_days_on_market, na.rm = TRUE))
```
