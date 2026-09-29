1. Start the table with each metro's average days on market
```{r}
metro_dom_volatility <- postCOVID_housing_without_QF %>%
  group_by(cbsa_code) %>%
  summarise(cbsa_title          = first(cbsa_title),
            household_size_tier = first(household_size_tier),
            avg_days_on_market  = mean(median_days_on_market, na.rm = TRUE))
```
2. Add price volatility
```{r}
volatility <- postCOVID_housing_without_QF %>%
  group_by(cbsa_code) %>%
  summarise(price_volatility = sd(median_listing_price_mm, na.rm = TRUE))

metro_dom_volatility <- metro_dom_volatility %>%
  left_join(volatility, by = "cbsa_code")
```

3. Drop metros missing either value
```{r}
metro_dom_volatility <- metro_dom_volatility %>%
  filter(!is.na(avg_days_on_market), !is.na(price_volatility))
```

4. Keep only two tiers (Largest & Smallest)
```{r}
two_tiers <- metro_dom_volatility %>%
  filter(household_size_tier %in% c("Tier 1 (Largest)", "Tier 9 (Smallest)")) %>%
  droplevels()

table(two_tiers$household_size_tier)
```

5. The Scatterplot
```{r}
ggplot(two_tiers, aes(x = avg_days_on_market, y = price_volatility,
                      color = household_size_tier)) +
  geom_point(alpha = 0.6, size = 2) +
  geom_smooth(aes(fill = household_size_tier), method = "lm", formula = y ~ x, alpha = 0.15) +
  scale_color_manual(values = c("Tier 1 (Largest)" = "#2a78d6", "Tier 9 (Smallest)" = "#eb6834")) +
  scale_fill_manual(values  = c("Tier 1 (Largest)" = "#2a78d6", "Tier 9 (Smallest)" = "#eb6834")) +
  labs(title = "Days on market vs. price volatility: largest vs. smallest metros",
       subtitle = "Jun 2023 to Aug 2026. Each dot is one metro.",
       x = "Average median days on market",
       y = "Price volatility (SD of monthly price change)",
       color = NULL, fill = NULL) +
  theme_minimal()
```
