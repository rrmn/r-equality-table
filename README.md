Equality in R is mostly straightforward, but sometimes it's not. Take care.

### Thanks
Thanks to [dorey](https://github.com/dorey) for the [idea](https://github.com/dorey/JavaScript-Equality-Table).

### Result
![image](https://github.com/rrmn/r-equality-table/assets/14080347/ebecd57a-587e-401c-a353-0e8045bc24c2)


### Code
```
# sorry for the hacky code :-)
x_labels <- c(
    "FALSE", "F", "0", "\"0\"", "0L", "\"0L\"", "0.0", "\"0.0\"", "0.0L", "\"0.0L\"", "list(0)", "c(0)",
    "TRUE", "T", "1", "\"1\"", "1L", "\"1L\"", "1.0", "\"1.0\"", "1.0L", "\"1.0L\"", "list(1)", "c(1)",
    "NA", "NULL", "NaN",
    "Inf", "-Inf"
)

res_v <- vector(mode = "character", length = length(x_labels) ^ 2)
res_comp <- vector(mode = "character", length = length(x_labels) ^ 2)
for (i in 1:length(x_labels)) {
    for (j in 1:length(x_labels)) {
        r <- tryCatch(
            expr = {
                as.character(eval(rlang::parse_expr(x_labels[i])) == eval(rlang::parse_expr(x_labels[j])))
            },
            error = function(e) {
                "Not implemented"
            }
        )
        if (length(r) == 0) {
            r <- "logical(0)"
        }
        if (is.na(r)) {
            r <- "NA"
        }
        res_v[((i - 1) * length(x_labels)) + j] <- r
    } 
}

res_df <- tibble(x = x_labels) %>%
    crossing(tibble(y = x_labels)) %>%
    ungroup() %>%
    # arrange(match(x, x_labels), match(y, x_labels)) %>%
    mutate(
        x = factor(x),
        y = factor(y)
    ) %>%
    group_by(x) %>%
    mutate(
        xmax = cur_group_id(),
        xmin = xmax - 1
    ) %>%
    ungroup() %>%
    group_by(y) %>%
    mutate(
        ymax = cur_group_id(),
        ymin = ymax - 1
    ) %>%
    ungroup() %>%
    mutate(
        c = factor(res_v, levels = c("TRUE", "FALSE", "NA", "logical(0)", "Not implemented"))
    )

ggplot(res_df) + 
    geom_rect(
        aes(xmin = xmin, xmax = xmax, ymin = ymin, ymax = ymax, fill = c),
        color = "#111111",
        alpha = 0.7,
        stat = "identity"
    ) +
    theme(legend.title = element_blank()) +
    scale_fill_manual(values = c("#06d6a0", "#ef476f", "#8d99ae", "#118ab2", "#073b4c")) +
    scale_y_continuous(expand=c(0,0), breaks = 1:length(x_labels) - 0.5, labels = x_labels) +
    scale_x_continuous(expand=c(0,0), breaks = 1:length(x_labels) - 0.5, labels = x_labels) +
    labs(title = "R Equality Table", subtitle = "Formula: X == Y", caption = "Code available under github.com/rrmn/r-equality-table")
```
