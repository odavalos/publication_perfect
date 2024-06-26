## Learning Objective
- Implement external packages to create Venn diagrams and heatmaps 

# External packages for figure creation

We have discovered that `ggplot2` has incredible functionality and versatility; however, there are some examples of visualization that are either not possible at all or require some assistance from external packages. There are a plethora of different packages that specialize in particular types of figures. The [Data to Viz resource](https://www.data-to-viz.com) provides popular packages for specialized plots along with the code to create them. 

## Visualizing overlaps of sets using Venn diagrams

Let's start by exploring how to create the Venn diagram in figure 4H. 

<p align="center">
<img src="../img/fig-H.jpeg">
</p>

  
A Venn diagram compares two or more lists, and by nature is categoric. If we use the [Data to Viz resource](https://www.data-to-viz.com), we can navigate to the `Categoric` data, and under 'Two independent lists' we find the Venn diagram. Clicking on the Venn diagram icon will open up a pop up window with a brief description of Venn diagrams. At the bottom of the pop up page, find the link to the "dedicated page"; explore the [dedicated page](https://www.data-to-viz.com/graph/venn.html).

<p align="center">
<img src="../img/dedicated_page.png" height="400">
</p>


**This page has a lot of nice information about Venn diagrams**, as well as, suggestions for when to use them (e.g. generally not recommended for comparison of more than 3 sets - use [upset plots](https://jku-vds-lab.at/tools/upset/) instead). Since we are comparing two sets of data for each visualization (e.g. Pax6 and Tbr2-expressing samples), a Venn diagram is a recommended method.

If we expand the 'Code', we see that the package used to create the Venn diagrams is `VennDiagram`. Since this is the package used to create the figure in the paper, we will provide code to demonstrate how it was done later in the lesson. 

**We will focus on the `ggvenn` package**, which provides an easy-to-use way to draw Venn diagrams using the standard ggplot2 syntax and layout. The package hence makes it possible to **match the design and style of Venn diagrams to other graphics created by the ggplot2 package**. 

First, we will need to load the library:

```r
# Load library
library(ggvenn)
```

To create the Venn diagram, we need to generate our sets. We can subset our data for the `Pax6`- and `Tbr2`-expressing samples to only include those genes that are significant with `threshold` equal to `TRUE` and that are up-regulated using the `log2FoldChange` values > 0.

```r

# Up-regulated
up1 <- results[which(results$pax6_threshold & results$pax6_log2FoldChange > 0),]
up2 <- results[which(results$tbr2_threshold & results$tbr2_log2FoldChange > 0),]
```

We can do a similar subset for the down-regulated genes, but with `log2FoldChange` < 0.

```r
# Down-regulated
down1 <- results[which(results$pax6_threshold & results$pax6_log2FoldChange < 0),]
down2 <- results[which(results$tbr2_threshold & results$tbr2_log2FoldChange < 0),]
```

Now that we have our sets of genes that are up- and down-regulated in the respective conditions, we can check how many overlap between the conditions. This is a good check to perform to make sure the packages report the correct interaction numbers.

```r
# Check overlap for each of the different sets
length(which(row.names(up1) %in% row.names(up2)))
length(which(row.names(down1) %in% row.names(down2)))
```

To create the figures using ggvenn we first take the union of genes for each direction of change and use those gene identifiers to create dataframes. The row names correspond to gene identifiers, and we add two columns of logical values for PAX6 and TBR2.

```r
# Combine results from datasets and remove duplicated values
up <- union(rownames(up1), rownames(up2))
down <- union(rownames(down1), rownames(down2))

# Create dataframe for comparison of up-regulated genes
data_up <- data.frame(value = up,
                      PAX6 = FALSE,
                      TBR2 = FALSE)
data_up$PAX6 <- data_up$value %in% rownames(up1)
data_up$TBR2 <- data_up$value %in% rownames(up2)

# Create dataframe for comparison of down-regulated genes
data_down <- data.frame(value = down,
                        PAX6 = FALSE,
                        TBR2 = FALSE)
data_down$PAX6 <- data_down$value %in% rownames(down1)
data_down$TBR2 <- data_down$value %in% rownames(down2)
```

Next, we can use the `ggvenn()` function from the ggvenn package to create a simple Venn diagram:

```r
ggvenn(data_up)
```

<p align="center">
<img src="../img/ggven_vanilla.png" height="300">
</p>


This has successfully created a Venn diagram, but **this is not exactly a publication-quality figure**. Most obviously, there are **changes required** to make it more closely resemble the original figure.  Let's explore what we can do with ggvenn.

First, note that since ggvenn is based in ggplot we can add it as a layer just like any other plot. **The function `geom_venn()` comes from the ggvenn package**. Note that for most uses, the simple ggvenn() command should suffice but since we have specific changes to replicate the image in the paper, we need to add arguments and/or additional layers. An example of using `geom_venn()` is provided below:

```r
ggplot(data_up, aes(A=PAX6, B=TBR2)) +
   geom_venn() +
   theme_void() +
   ggtitle("Overlap of up-regulated genes") +
   theme(plot.title = element_text(hjust = 0.5))
```

_NOTE: theme_void() is necessary here to remove all grids, axes, and coloring from the background._

***

**Exercise**

Below we provide a skeleton of the publication-quality `geom_venn` code. The value of the following arguments are left empty:

* `fill_color`
* `set_name_color`
* `set_name_size`
* `text_size`

Please fill in those values so that the final figure looks similar to that in the paper (see below).

```r
# Up-regulated genes
ggplot(data_up, aes(A=PAX6, B=TBR2)) +
  geom_venn(
        stroke_color="grey",
        show_percentage = FALSE,
        auto_scale = TRUE,
        fill_color= ,
        set_name_color =  ,
        set_name_size = ,
        text_size = ) +
  theme_void() + 
  ggtitle("Overlap of up-regulated genes") +
  theme(plot.title = element_text(hjust = 0.5, size = rel(1.5)))
```

<p align="center">
<img src="../img/ggvenn_up.png" height="450">
</p>

***

Similarly, we could plot the venn diagram for the down-regulated genes.

<details>
  <summary>Code</summary>

  <p><pre>
    # Down-regulated genes
    ggplot(data_down, aes(A=PAX6, B=TBR2)) +
     geom_venn(
        stroke_color="grey",
        show_percentage = FALSE,
        auto_scale = TRUE,
        fill_color = c("salmon", "lightblue"),
        set_name_color = c("salmon", "lightblue"),
        set_name_size = 5,
        text_size = 4) +
    theme_void() + 
    ggtitle("Overlap of down-regulated genes") +
    theme(plot.title = element_text(hjust = 0.5, size = rel(1.5)))
                 
  </pre></p>
</details>

<p align="center">
<img src="../img/ggvenn_down.png" height="450">
</p>


### Exporting the VennDiagram 

We will need to output our visualization as a pdf or png so that we can later incorporate it into the final figure. This is very easy using the plotting device function in R. We simply open the plotting device, either `pdf()` or `png()`, plot our figure and then close the plotting device with `dev.off()` Below is an example for our down-regulated genes with a png output. 

**Run this code and do a similar thing for the up-regulated genes (""results/venn_up.png").**

```r
png("results/venn_down.png")
ggplot(data_down, aes(A=PAX6, B=TBR2)) +
  geom_venn(
    stroke_color="grey",
    show_percentage = FALSE,
    fill_color=c("salmon", "lightblue"),
    set_name_color = c("salmon", "lightblue"),
    set_name_size = 8,
    text_size = 6) +
  theme_void() + 
  ggtitle("Overlap of down-regulated genes") +
  theme(plot.title = element_text(hjust = 0.5))
dev.off()
```

**Note that these outputs can be highly customized check `?pdf` and `?png` for details. Also note that other file types can be output.**

***

> **NOTE:** For the down-regulated genes, the overlap number is not placed within the overlapping region. Unfortunately, the placement of this number is not easy to change. **To create an identical figure using the `VennDiagram` package you can check out the link below.**

<details>
  <summary><b>Click here for code to create the figure using the VennDiagram package</b></summary>

  <p><pre>
    
    # Load library
    library(VennDiagram)
    
    # Create lists for comparison
    up <- list(
      PAX6 = rownames(up1),
      TBR2 = rownames(up2))

    # Plot figure
    venn.diagram(x = up, 
                 filename = "results/venn_up.png",
                 output = TRUE,
                 main = "Overlap of up-regulated genes",
                 main.fontfamily = "sans",
                 main.cex = 0.75,
                 height = 600 , 
                 width = 600 , 
                 resolution = 300,
                 compression = "lzw",
                 lwd = 2,
                 col= "gray",
                 fill = c("salmon", "lightblue"),
                 cex = 0.5,
                 fontfamily = "sans",
                 category.names = c("PAX6" , "TBR2"),
                 cat.cex = 0.7,
                 cat.fontface = "bold",
                 cat.default.pos = "outer",
                 cat.pos = 0,
                 cat.dist = c(0.06,0.097),
                 cat.fontfamily = "sans",
                 cat.col = c("salmon", "lightblue"))    
  </pre></p>
</details>

***

## Visualizing numeric data from different conditions or groups
  
Next, we will generate the Fig 4F heatmap as shown below. We will use specialized packages for the creation of hierarchical heatmaps. A benefit of these packages is that they incorporate statistical information, such as dendrograms and clustering of rows and/or columns. Although ggplot2 can easily create a heatmap with `geom_tile()`, it cannot easily provide the hierarchical clustering as these packages. A few examples of these packages include `pheatmap`, `d3heatmap`, and `ComplexHeatmap`. We will explore the `pheatmap` package for our hierarchical clustering figure; however, additional information can be found for `d3heatmap` from [Data to viz](https://www.data-to-viz.com/graph/heatmap.html), and we have [additional materials](supp_complexHeatmap.md) highlighting the code to create the same plot using `ComplexHeatmap`. ComplexHeatmap definitely embraces its name, and there is a [whole book](https://jokergoo.github.io/ComplexHeatmap-reference/book/) dedicated to creating custom heatmaps using this package.
  
<p align="center">
<img src="../img/fig-F.png" height="500">
</p>

Generally, heatmaps are used to compare numeric data from different conditions or groups. These comparisons range across different fields of study; for instance, a heatmap could be used to explore how temperatures across the seasons have increased over the past 100 years, or it could be used to explore the monthly earnings for all fortune 500 companies. We will use a heatmap to explore our biological data and look at the genes that have significantly different expression between mice with or without the *Prdm16* gene. The hierarchical clustering will add information to the figure about which mice are most similar to each other in the expression of these genes.

To create this figure, we first need to subset our gene expression data to the significant genes for the radial glia cells (_Pax6_-expressers). To get the names of our significant genes we can filter to only include those with significance (p-adjusted) values less than 0.05.
 
```r
# Get Pax6 sig genes
pax6_sig_genes <- results %>%
  rownames_to_column("ensembl_id") %>%
  filter(pax6_padj < 0.05) %>%
  pull(ensembl_id)
```

We are only interested in the radial glia, so we will extract only the `Pax6` samples.
  
```r
# Filter heatmap for only the radial glia (Pax6 samples)
heatmap_normCounts <- normalized_counts[ , colnames(normalized_counts)[str_detect(colnames(normalized_counts), "Pax6")]]
```
  
Then we can extract only the significant genes from our `Pax6` expression matrix. 

```r
# Filter for the sig genes
heatmap_normCounts <- heatmap_normCounts[which(rownames(heatmap_normCounts) %in% pax6_sig_genes), ]
```
  
Now we have the values to be included in the heatmap, but we also need to color it by group, `Prdm16` knockout or wildtype. To get that information, we need to use our metadata. We will extract the same mice samples from our metadata that we have the counts for, which is the `Pax6` samples.

We'll start by copying our full metadata table to a new variable to ensure we don't accidently alter our original metadata.
  
```r
# Copy metadata to new variable
heatmap_meta <- meta
```
 
Now to create the heatmap using the `pheatmap` package, the row names of our metadata need to be the sample names and match the column names of our counts (numeric) data.

```r
# Assign the sample names to be the row names
rownames(heatmap_meta) <- heatmap_meta$samples
```
  
Now we can subset the metadata to only include those samples present in the `Pax6` counts data.
  
```r
# Subset metadata to match samples in count data
heatmap_meta <- heatmap_meta[which(rownames(heatmap_meta) %in% colnames(heatmap_normCounts)), ]
```
  
We are only interested in using the genotype annotations, so we will keep only that column. All columns given to `pheatmap` for annotations will be included as separate bars.
  
```r
# Select only the genotype column for annotations
heatmap_meta <- heatmap_meta %>%
  select(genotype)
```
  
The factor levels for the genotype will determine the color automatically assigned to the group. We can manually specify the levels using the `factor()` function.
  
```r
# Re-level factors
heatmap_meta$genotype <- factor(heatmap_meta$genotype, levels = c("WT", "KO"))
```

<p align="center">
<img src="../img/meta_heatmap.png" height="200">
</p>

Now that we have the metadata with information for annotating the groups and the count data to be visually transformed into the heatmap, we only need to provide any customized colors to use in the heatmap and to denote the groups. We can specify the colors to use for the gradient in the numeric expression values:
  
```r
# Colors for heatmap
heatmap_colors <- colorRampPalette(c("blue", "white", "red"))(6)
```
 
Also, we can specify the colors to use to denote the annotations. The colors for the annotations need to be provided as a list:

```r
# Colors for denoting annotations
ann_colors <- list(genotype = c(KO = "#F8766D", WT = "#00BFC4"))
```

Now we can load the `pheatmap` library, using the `pheatmap()` function to create the heatmap.

```r
# Load library
library(pheatmap)

# Plot heatmap  
pheatmap(heatmap_normCounts, 
         color = heatmap_colors,
         annotation_col = heatmap_meta,
         annotation_colors = ann_colors,
         show_colnames = F, 
         show_rownames = F)
```

<p align="center">
<img src="../img/unscaled_heatmap.png" height="500">
</p>

We are have created a basic heatmap, but it's not very informative as it is. It looks like the expression of a lot of the genes is much lower than some of the others, so we can't see the differences very easily for the genes that don't have the highest values. There is a helpful argument called `scale` that allows us to scale the colors by the values in each row or column. Since each gene is a row, we will add this argument to be `scale = "row"`, which centers and scales the values.
  
```r
# Plot heatmap scaled by row
pheatmap(heatmap_normCounts, 
         color = heatmap_colors,
         annotation_col = heatmap_meta,
         annotation_colors = ann_colors,
         show_colnames = F, 
         show_rownames = F,
         scale="row")
```

<p align="center">
<img src="../img/scaled_heatmap.png" height="500">
</p>

  
Now that looks a lot better! We can see clustering between the different groups, which is good since these genes are supposed to be significantly different in their expression between the groups. We can add a few more arguments to make it look more like the figure, like deleting the annotation name and legend. We will also adjust the width and height of the tiles.

```r 
# Scaled heatmap with adjustments for figure
pheatmap(heatmap_normCounts, 
         color = heatmap_colors,
         annotation_col = heatmap_meta,
         annotation_colors = ann_colors,
         show_colnames = F, 
         show_rownames = F,
         scale="row",
         treeheight_col = 25,
         cellwidth = 20,
         cellheight = 0.4,
         annotation_names_col = F,
         annotation_legend = F)
```
  
<p align="center">
<img src="../img/heatmap.png" height="600">
</p>

  
Now we are only missing the text annotations, and we can add using the `cowplot` package we learned about earlier. However, the pheatmap image is stored as a list rather than a graphics object. The `ggplotify` package allows us to manipulate this list object into a ggplot object with the `as.ggplot()` function, faciliatating further modifications with `ggplot2` or `cowplot`.

```r
# Load library
library(ggplotify)
  
# Turn into a ggplot object
heatmap <- as.ggplot(pheatmap(heatmap_normCounts, 
                              color = heatmap_colors,
                              cluster_rows = T, 
                              annotation_col = heatmap_meta,
                              annotation_colors = ann_colors,
                              border_color="white", 
                              cluster_cols = T, 
                              show_colnames = F, 
                              show_rownames = F,  
                              scale="row",
                              treeheight_col = 25,
                              cellwidth = 20,
                              cellheight = 0.40,
                              fontsize = 9,
                              annotation_names_col = F,
                              annotation_legend = F))
```

To add the annotations at the top and left sides of the plot, we will use the `theme()` function from `ggplot2` to add whitespace. With the `plot.margin` argument we can specify the amount of space to add to the top, right, bottom and left margins (in that order).           

```r          
# Adding white space to add annotations to the top, right, bottom, and left margins
heatmap <- heatmap + 
  theme(plot.margin = unit(c(1,0.5,0.5,0.5), "cm"))           
```
  
Now we can use `cowplot` to add in our desired annotations to match the published figure below.
  
```r  
# Adding in annotations with cowplot 
heatmap_figure <- ggdraw(heatmap) +  
  draw_label("Radial glia", 
             x = 0.4, 
             y = 0.97,
             size = 14,
             hjust = 0,
             vjust = 0) +
  draw_label("Prdm16 cKO", 
             x = 0.24, 
             y = 0.91,
             size = 12,
             hjust = 0,
             vjust = 0,
             fontface = "italic") +
  draw_label("Prdm16 WT", 
             x = 0.58, 
             y = 0.91,
             size = 12,
             hjust = 0,
             vjust = 0,
             fontface = "italic") +
  draw_label("Up-regulated genes",
             angle = 90,
             size = 9,
             x = 0.03,
             y = 0.5,
             hjust = 0,
             vjust = 0) +
  draw_label("Down-regulated genes",
             angle = 90,
             size = 9,
             x = 0.03,
             y = 0.09,
             hjust = 0,
             vjust = 0) +
  draw_label("Genotype",
             size = 9,
             x = 0.09,
             y = 0.8,
             hjust = 0,
             vjust = 0)

heatmap_figure
```

                  
<p align="center">
<img src="../img/heatmap_figure.png" height="600">
</p>

Finally, we want to export the image so that it will display properly in our final figure.

```r                  
# Save image
ggsave(filename = "results/heatmap_figure.png",
       plot = heatmap_figure,
       units = "in", 
       width = 4, 
       height = 5.15, 
       dpi = 500)                    
```
