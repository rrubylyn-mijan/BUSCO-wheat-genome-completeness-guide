# BUSCO 5.7.1 Genome Completeness Guide

This is a beginner-friendly guide to run BUSCO v5.7.1 (Benchmarking Universal Single-Copy Orthologs)
for genome completeness analysis and visualization.

Created by: Ruby Mijan


## STEP 1: Load Conda on HPC
> **Conda** is a tool that helps you install software and manage environments without causing version conflicts. It lets you keep your tools organized for each project.

```bash
ml miniconda3/24.3.0
```


## STEP 2: Create and Activate a Mamba Environment
> **Mamba** is a faster version of Conda. It works the same way but installs packages much quicker, which is very helpful when setting up tools like BUSCO that have many dependencies.

```bash
conda create --prefix /home/YOUR_USERNAME/miniconda_envs/mamba_env -c conda-forge mamba
conda activate /home/YOUR_USERNAME/miniconda_envs/mamba_env
```


## STEP 3: Install BUSCO Environment Using Mamba
```bash
mamba create --prefix /home/YOUR_USERNAME/miniconda_envs/busco_env -c conda-forge -c bioconda busco=5.7.1
```

## STEP 4: Initialize and Activate BUSCO
```bash
eval "$(mamba shell hook --shell bash)"
mamba activate /home/YOUR_USERNAME/miniconda_envs/busco_env
mamba shell init --shell bash --root-prefix=~/.local/share/mamba
source ~/.bashrc
mamba activate /home/YOUR_USERNAME/miniconda_envs/busco_env
```

## STEP 5: Run BUSCO on a Genome
```bash
busco -i /path/to/genome.fasta \
      -l poales_odb10 \
      -o busco_output_genome \
      -m genome
```

## STEP 6: Visualize Results for a Single Genome
```bash
mkdir /your_path/BUSCO_summaries_ggplot_genome

scp /your_path/busco_output_genome/run_poales_odb10/short_summary.txt \
    /your_path/BUSCO_summaries_ggplot_genome/

cd /your_path/BUSCO_summaries_ggplot_genome
mv short_summary.txt short_summary.generic.poales_odb10.YourGenome.txt

python3 /home/YOUR_USERNAME/miniconda_envs/busco_env/bin/generate_plot.py \
        -wd /your_path/BUSCO_summaries_ggplot_genome/
```

### Optional: Transfer to local computer (Windows example)
```bash
scp first.last@atlas-login.hpc.msstate.edu:/your_path/BUSCO_summaries_ggplot_genome/busco_figure.png \
    C:\Users\first.last\Documents\BUSCO_figures
```

## STEP 7: Combine and Visualize Multiple Genomes
```bash
mkdir /your_path/BUSCO_combined_genomes

# Copy all short summary files
scp /your_path/short_summary.generic.poales_odb10.Genome1.txt \
    /your_path/BUSCO_combined_genomes/

scp /your_path/short_summary.generic.poales_odb10.Genome2.txt \
    /your_path/BUSCO_combined_genomes/

scp /your_path/short_summary.generic.poales_odb10.Genome3.txt \
    /your_path/BUSCO_combined_genomes/
```

## Generate combined plot using python
```bash
python3 /home/YOUR_USERNAME/miniconda_envs/busco_env/bin/generate_plot.py \
        -wd /your_path/BUSCO_combined_genomes/
## Download to local PC
```bash
scp first.last@atlas-login.hpc.msstate.edu:/your_path/BUSCO_combined_genomes/busco_figure.png \
    C:\Users\first.last\Documents\BUSCO_figure
```

# Generate a BUSCO genome completeness figure using R
This R script generates BUSCO summary figures for wheat genome assemblies. The plot shows the percentage of complete single-copy, complete duplicated, fragmented, and missing BUSCO genes.

The script includes two BUSCO figures:

1. BUSCO comparison of Sumai 3, wheat-r, and wheat-g
2. BUSCO comparison of wheat-g and wheat-r only
## R script for figure 1
```bash
######################################
# BUSCO summary figure
# Wheat genome assemblies
#
# Generates BUSCO completeness plots for:
# 1. Sumai 3, wheat-r, and wheat-g
# 2. wheat-g and wheat-r only
######################################

library(ggplot2)
library(grid)

######################################
# Figure 1: Sumai 3, wheat-r, and wheat-g
######################################

my_output <- paste("D:/1atlas_HOME/BUSCO_combined_genomes", "busco_figure.png", sep = "/")
my_width <- 20
my_height <- 15
my_unit <- "cm"

my_colors <- c("#56B4E9", "#3492C7", "#F0E442", "#F04442")
my_bar_height <- 0.75
my_title <- "BUSCO Assessment Results"
my_family <- "sans"
my_size_ratio <- 1

my_species <- c(
  "wheat-s.poales_odb10", "wheat-s.poales_odb10", "wheat-s.poales_odb10", "wheat-s.poales_odb10",
  "wheat-r.poales_odb10", "wheat-r.poales_odb10", "wheat-r.poales_odb10", "wheat-r.poales_odb10",
  "wheat-g.poales_odb10", "wheat-g.poales_odb10", "wheat-g.poales_odb10", "wheat-g.poales_odb10"
)

my_percentage <- c(
  1.8, 97.9, 0.2, 0.1,
  2.0, 97.7, 0.2, 0.1,
  1.9, 97.7, 0.2, 0.2
)

my_values <- c(
  86, 4795, 9, 6,
  97, 4783, 10, 6,
  95, 4785, 10, 6
)

category <- rep(c("D", "S", "F", "M"), 3)

df <- data.frame(
  species = my_species,
  percentage = my_percentage,
  values = my_values,
  category = category
)

df$species <- factor(df$species, levels = c(
  "wheat-g.poales_odb10",
  "wheat-r.poales_odb10",
  "wheat-s.poales_odb10"
))

df$category <- factor(df$category, levels = c("S", "D", "F", "M"))

labsize <- 1
if (length(levels(df$species)) > 10) {
  labsize <- 0.66
}

figure <- ggplot(df, aes(y = percentage, x = species, fill = category)) +
  geom_bar(
    position = position_stack(reverse = TRUE),
    stat = "identity",
    width = my_bar_height
  ) +
  coord_flip() +
  theme_gray(base_size = 8) +
  scale_y_continuous(
    labels = c("0", "20", "40", "60", "80", "100"),
    breaks = c(0, 20, 40, 60, 80, 100)
  ) +
  scale_fill_manual(
    values = my_colors,
    labels = c(
      "Complete (C) and single-copy (S)",
      "Complete (C) and duplicated (D)",
      "Fragmented (F)",
      "Missing (M)"
    )
  ) +
  ggtitle(my_title) +
  xlab("") +
  ylab("\n%BUSCOs") +
  theme(
    plot.title = element_text(
      family = my_family,
      hjust = 0.5,
      colour = "black",
      size = rel(2.2) * my_size_ratio,
      face = "bold"
    ),
    legend.position = "top",
    legend.title = element_blank(),
    legend.text = element_text(
      family = my_family,
      size = rel(1.2) * my_size_ratio
    ),
    panel.background = element_rect(color = "#FFFFFF", fill = "white"),
    panel.grid.minor = element_blank(),
    panel.grid.major = element_blank(),
    axis.text.y = element_text(
      family = my_family,
      colour = "black",
      size = rel(1.66) * my_size_ratio
    ),
    axis.text.x = element_text(
      family = my_family,
      colour = "black",
      size = rel(1.66) * my_size_ratio
    ),
    axis.line = element_line(size = 1 * my_size_ratio, colour = "black"),
    axis.ticks.length = unit(0.4, "cm"),
    axis.ticks.y = element_line(colour = "white", size = 0),
    axis.ticks.x = element_line(colour = "#222222"),
    axis.title.x = element_text(
      family = my_family,
      size = rel(1.2) * my_size_ratio
    )
  ) +
  guides(fill = guide_legend(nrow = 2, byrow = TRUE))

for (i in seq_along(levels(df$species))) {
  current_species <- levels(df$species)[i]
  detailed_values <- df$values[df$species == current_species]
  total_buscos <- sum(detailed_values)

  figure <- figure +
    annotate(
      "text",
      label = paste(
        "C:", detailed_values[df$category[df$species == current_species] == "S"] +
          detailed_values[df$category[df$species == current_species] == "D"],
        " [S:", detailed_values[df$category[df$species == current_species] == "S"],
        ", D:", detailed_values[df$category[df$species == current_species] == "D"],
        "], F:", detailed_values[df$category[df$species == current_species] == "F"],
        ", M:", detailed_values[df$category[df$species == current_species] == "M"],
        ", n:", total_buscos,
        sep = ""
      ),
      y = 3,
      x = i,
      size = labsize * 4 * my_size_ratio,
      colour = "black",
      hjust = 0,
      family = my_family
    )
}

ggsave(
  figure,
  file = my_output,
  width = my_width,
  height = my_height,
  unit = my_unit
)

print("Figure 1 done")
```
## R script for figure 2
```bash
######################################
# Figure 2: wheat-g and wheat-r only
######################################

my_output <- paste("D:/1atlas_HOME/BUSCO_combined_genomes", "busco_figure_wheat-g_wheat-r.png", sep = "/")
my_title <- "BUSCO Assessment Results (wheat-g and wheat-r)"

my_species <- c(
  "wheat-r.poales_odb10", "wheat-r.poales_odb10", "wheat-r.poales_odb10", "wheat-r.poales_odb10",
  "wheat-g.poales_odb10", "wheat-g.poales_odb10", "wheat-g.poales_odb10", "wheat-g.poales_odb10"
)

my_percentage <- c(
  2.0, 97.7, 0.2, 0.1,
  1.9, 97.7, 0.2, 0.2
)

my_values <- c(
  97, 4783, 10, 6,
  95, 4785, 10, 6
)

category <- rep(c("D", "S", "F", "M"), 2)

df <- data.frame(
  species = my_species,
  percentage = my_percentage,
  values = my_values,
  category = category
)

df$species <- factor(df$species, levels = c(
  "wheat-g.poales_odb10",
  "wheat-r.poales_odb10"
))

df$category <- factor(df$category, levels = c("S", "D", "F", "M"))

figure <- ggplot(df, aes(y = percentage, x = species, fill = category)) +
  geom_bar(
    position = position_stack(reverse = TRUE),
    stat = "identity",
    width = my_bar_height
  ) +
  coord_flip() +
  theme_gray(base_size = 8) +
  scale_y_continuous(
    labels = c("0", "20", "40", "60", "80", "100"),
    breaks = c(0, 20, 40, 60, 80, 100)
  ) +
  scale_fill_manual(
    values = my_colors,
    labels = c(
      "Complete (C) and single-copy (S)",
      "Complete (C) and duplicated (D)",
      "Fragmented (F)",
      "Missing (M)"
    )
  ) +
  ggtitle(my_title) +
  xlab("") +
  ylab("\n%BUSCOs") +
  theme(
    plot.title = element_text(
      family = my_family,
      hjust = 0.5,
      colour = "black",
      size = rel(2.2) * my_size_ratio,
      face = "bold"
    ),
    legend.position = "top",
    legend.title = element_blank(),
    legend.text = element_text(
      family = my_family,
      size = rel(1.2) * my_size_ratio
    ),
    panel.background = element_rect(color = "#FFFFFF", fill = "white"),
    panel.grid.minor = element_blank(),
    panel.grid.major = element_blank(),
    axis.text.y = element_text(
      family = my_family,
      colour = "black",
      size = rel(1.66) * my_size_ratio
    ),
    axis.text.x = element_text(
      family = my_family,
      colour = "black",
      size = rel(1.66) * my_size_ratio
    ),
    axis.line = element_line(size = 1 * my_size_ratio, colour = "black"),
    axis.ticks.length = unit(0.4, "cm"),
    axis.ticks.y = element_line(colour = "white", size = 0),
    axis.ticks.x = element_line(colour = "#222222"),
    axis.title.x = element_text(
      family = my_family,
      size = rel(1.2) * my_size_ratio
    )
  ) +
  guides(fill = guide_legend(nrow = 2, byrow = TRUE))

for (i in seq_along(levels(df$species))) {
  current_species <- levels(df$species)[i]
  detailed_values <- df$values[df$species == current_species]
  total_buscos <- sum(detailed_values)

  figure <- figure +
    annotate(
      "text",
      label = paste(
        "C:", detailed_values[df$category[df$species == current_species] == "S"] +
          detailed_values[df$category[df$species == current_species] == "D"],
        " [S:", detailed_values[df$category[df$species == current_species] == "S"],
        ", D:", detailed_values[df$category[df$species == current_species] == "D"],
        "], F:", detailed_values[df$category[df$species == current_species] == "F"],
        ", M:", detailed_values[df$category[df$species == current_species] == "M"],
        ", n:", total_buscos,
        sep = ""
      ),
      y = 3,
      x = i,
      size = labsize * 4 * my_size_ratio,
      colour = "black",
      hjust = 0,
      family = my_family
    )
}

ggsave(
  figure,
  file = my_output,
  width = my_width,
  height = my_height,
  unit = my_unit
)

print("Figure 2 done")
```

## Output Files
The script generates:
```text
busco_figure.png
busco_figure_glenn_rollag.png
```
## Notes

Change this line if running the script in a different directory:
```r
my_output <- paste("D:/1atlas_HOME/BUSCO_combined_genomes", "busco_figure.png", sep = "/")
```
# Example Directory Structure
```bash
BUSCO_project/
├── Genome FASTA files
├── BUSCO_combined_genomes/
│   ├── short_summary.generic.poales_odb10.Genome1.txt
│   ├── short_summary.generic.poales_odb10.Genome2.txt
│   ├── short_summary.generic.poales_odb10.Genome3.txt
│   └── busco_figure.png
├── BUSCO_summaries_ggplot_genome/
│   └── short_summary.generic.poales_odb10.YourGenome.txt
└── busco_env/
```

Maintainer: Ruby Mijan (rrubylyn.mijan@gmail.com)

