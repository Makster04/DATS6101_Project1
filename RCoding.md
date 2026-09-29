Here's the full sequence in order. Steps 1–3 and 5 are unchanged. Step 4 now labels every metro instead of dropping Tiers 2–8, and Step 6 draws Tiers 2–8 as light grey dots behind Tier 1 and Tier 9.

**1. Start the table with each metro's average days on market**

```r
metro_dom_volatility <- postCOVID_housing_without_QF %>%
  group_by(cbsa_code) %>%
  summarise(cbsa_title          = first(cbsa_title),
            household_size_tier = first(household_size_tier),
            avg_days_on_market  = mean(median_days_on_market, na.rm = TRUE))
```

**2. Add price volatility**

```r
volatility <- postCOVID_housing_without_QF %>%
  group_by(cbsa_code) %>%
  summarise(price_volatility = sd(median_listing_price_mm, na.rm = TRUE))

metro_dom_volatility <- metro_dom_volatility %>%
  left_join(volatility, by = "cbsa_code")
```

**3. Drop metros missing either value**

```r
metro_dom_volatility <- metro_dom_volatility %>%
  filter(!is.na(avg_days_on_market), !is.na(price_volatility))
```

**4. Label the plot groups (Tier 1, Tier 9, and Tiers 2–8)**

```r
plot_data <- metro_dom_volatility %>%
  mutate(plot_group = case_when(
           household_size_tier == "Tier 1 (Largest)"  ~ "Tier 1 (Largest)",
           household_size_tier == "Tier 9 (Smallest)" ~ "Tier 9 (Smallest)",
           TRUE                                        ~ "Tiers 2-8"),
         plot_group = factor(plot_group,
                             levels = c("Tier 1 (Largest)", "Tier 9 (Smallest)", "Tiers 2-8"))) %>%
  arrange(desc(plot_group == "Tiers 2-8"))

two_tiers <- plot_data %>%
  filter(plot_group != "Tiers 2-8") %>%
  droplevels()

table(plot_data$plot_group)
```

**5. Set the cutoffs from all metros**

```r
dom_cut <- median(metro_dom_volatility$avg_days_on_market)
vol_cut <- median(metro_dom_volatility$price_volatility)
```

**6. The scatterplot**

```r
ggplot(plot_data, aes(x = avg_days_on_market, y = price_volatility,
                      color = plot_group)) +
  geom_point(alpha = 0.7, size = 2) +
  geom_smooth(data = two_tiers, aes(fill = plot_group),
              method = "lm", formula = y ~ x, alpha = 0.15, show.legend = FALSE) +
  geom_vline(xintercept = dom_cut, linetype = "dashed", color = "grey50") +
  geom_hline(yintercept = vol_cut, linetype = "dashed", color = "grey50") +
  annotate("text", x = dom_cut, y = Inf,
           label = paste0("Fast selling: under ", round(dom_cut), " days"),
           hjust = 1.05, vjust = 1.5, size = 3.2, color = "grey30") +
  annotate("text", x = Inf, y = vol_cut,
           label = paste0("High volatility: above ", round(vol_cut, 3)),
           hjust = 1.05, vjust = -0.6, size = 3.2, color = "grey30") +
  scale_color_manual(values = c("Tier 1 (Largest)"  = "#2a78d6",
                                "Tier 9 (Smallest)" = "#eb6834",
                                "Tiers 2-8"         = "grey80")) +
  scale_fill_manual(values = c("Tier 1 (Largest)"  = "#2a78d6",
                               "Tier 9 (Smallest)" = "#eb6834")) +
  labs(title = "Days on market vs. price volatility: largest vs. smallest metros",
       subtitle = "Jun 2023 to Aug 2026. Each dot is one metro. Dashed lines = median of all metros.",
       x = "Average median days on market",
       y = "Price volatility (SD of monthly price change)",
       color = NULL) +
  theme_minimal() +
  theme(legend.position = "top",
        legend.justification = "left")
```

**AXES:**

Y= **Price volatility** (how much a metro's home prices bounce up and down from month to month)
- Below a 0.038 volatility, low number means prices stay steady.
- Above a 0.038 volatility, high number means prices jump around a lot.

X= **Average median days on market** (about how long a typical home in a metro takes to sell) 
- Numbers below 60 days, means homes sell fast.
- Numbers above 60 days, means homes sit on the market longer than average.

**DOT PLACEMENTS ON THE SCATTERPLOT:**
- **Bottom left:** homes sell fast and prices stay steady. A healthy, busy market.
- **Top left:** homes sell fast, but prices jump around. Buyers are active, but prices are unpredictable.
- **Bottom right:** homes take longer to sell, but prices stay steady. A slow market where sellers aren't cutting prices.
- **Top right:** homes take longer to sell and prices jump around. A slow, unpredictable market.

<img width="2125" height="1170" alt="image" src="https://github.com/user-attachments/assets/45508ee4-a47d-40e6-8861-a27945729d92" />

