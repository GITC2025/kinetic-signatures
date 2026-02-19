**From Postmortem to Clinical: Kinetic Signatures of CAG Somatic Expansion in Huntington's Disease via snRNA-seq Analysis**

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.18626875.svg)](https://doi.org/10.5281/zenodo.18626875)

<img width="1406" height="643" alt="kineticworkflowSMALLcropped" src="https://github.com/user-attachments/assets/78796c7e-f3a1-48bb-8ef9-d98342122296" /><br>

* Reanalysis of Handsaker et al., 2025 PacBio CCS dataset + adaptation of Kacher Expansion Index formula (2021)
* Origins: Personal project to learn R studio for genomic analysis
* Timeline of project: Dec 2025 to Jan 2026 (one month+)
* Accepted for Women in Neuroscience Conference 2026, Queen's Uni, Kingston
  
**TLDR Methodology**
*   Reanalyse Handsaker et al., 2025 postmortem genomic dataset to quantify the differences in CAGLENGTH distribution between slow expanders (asymptomatic) vs fast expanders (symptomatic)
*   Include full PacBio dataset for CAGLENGTH data : 8 donors (2 asymptomatic + subclinical CAGLENGTH) versus original Handsaker et al., analysis: 6 symptomatic + clinical CAGLENGTH donors only
*   Adapt Expansion Index (Kacher et al., 2021) to Handsaker et al. dataset
*   From fluorescence signal ratios to digital count ratios
*   BCa bootstrapping to account for differences in cell counts across donors
*   Quantify an EI confidence interval for each donor
*   Correlate each donor's EI to Handsaker's expansion phase model (slow → fast → runaway expansion)
*   Generate heatmap of EI with risk factors: inherited CAG, age etc. to correlate with disease stage

**TLDR Clinical Translation**
*   Translate postmortem EI metrics to monitoring of living individuals via single and multi time point blood sampling
*   Stratify clinical risk and inform cost-benefit analysis of emerging treatments to slow somatic expansion

**Context**
*   CAG paternal anticipation → increases inherited length beyond paternal CAG length
*   CAG somatic expansion → lifetime expansion depending on inherited CAG length and other modifiers → significant mosaicism
*   somatic expansion poorly characterised
*   previous sequencing tech misses out long reads
*   PacBio CCS and other long reads captured up to 800+ CAG repeats for the first time
*   Provide comprehensive distribution of CAG lengths per donor 

[![](https://blogger.googleusercontent.com/img/a/AVvXsEg0GAhslVWDSdPNJiF2AtaOQ10x_wu8wOkWbVJ3m151YL6g8Eqd3iuSWGUvyT7ou-oShtX01LXrwSj8fGL5ifyn7oJ4BrJXTTi4dS5Ri98-4DYp3liQrxZ4cudBvKCzJuwbqLMrNvO5QFrMI2SZrlyhkbc_8EMci-KnX1L6pwKzaBw0atrcqsZ-o1EEbo3o=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEg0GAhslVWDSdPNJiF2AtaOQ10x_wu8wOkWbVJ3m151YL6g8Eqd3iuSWGUvyT7ou-oShtX01LXrwSj8fGL5ifyn7oJ4BrJXTTi4dS5Ri98-4DYp3liQrxZ4cudBvKCzJuwbqLMrNvO5QFrMI2SZrlyhkbc_8EMci-KnX1L6pwKzaBw0atrcqsZ-o1EEbo3o)

**Handsaker et al., 2025 Methods** 
*   DropSeq snRNA + PacBio CCS + Local alignment

**Handsaker et al., 2025 Findings**
*   5 phase model used to reconstruct lifetime somatic expansion rates per donor
*   Pacbio CCS seq for 8 affected + 1 healthy donor
*   Only analysed 6 affected in main paper as the other 2 were asymptomatic at point of death
*   Missed opportunity to analyse the expansion index of slow expanders to compare with faster expanders

<img src="https://blogger.googleusercontent.com/img/a/AVvXsEj_rpKe0C7jOfTHpo0KWr8I70o6x1UpZ9yDL32L-YU7RnmEz1awZbFrmEX5ZHYNrBiGZ_3Ztz85erdiWW1yCHfkZpd4yevWgEX3UFEkqg-BU6bIzkv7VtW6Rgz9n9_0ivaSyczMb2dmjM2jTn2-FgHB8vJPnzC-1Yb_A_O3XWbnyV6iykgtJePWcla407Xn" width="450">

**Data sources**
*   **pacbio\_deepdives\_cells\_unfiltered : contains CAGLENGTH data for 9 donors**
*   https://assets.nemoarchive.org/collection/nemo:dat-ue17119
*   **Demographic data: spreadsheet 1**
*   https://www.cell.com/cms/10.1016/j.cell.2024.11.038/attachment/f29b354c-e7b0-477e-99e0-65bcbf74696c/mmc1.xlsx

**QC before renalysis**
*   Merge pacbio\_deepdives\_cells\_unfiltered with demographic data 
*   Verify QC done (Handsaker et al., 2025 methods)
*    Low nreads (<10), faulty UMI already removed, UMI deduplication done, etc. 
*   Only one unique transcript (UMI) selected per cell (CELL\_BARCODE) in original dataset
*   pacbio\_deepdives\_cells\_unfiltered : already pre-processed and QC-ed

**Reanalysis Prep**
*   2 subclinical cases + 6 clinical cases + 1 healthy control
*   8 affected individuals are all heterozygotes 
*   Include subclinical cases as slow expanders to contrast with clinical cases 
*   **Filter to CELLTYPE = SPN only**
*   Verify that healthy control shows one peak at 17 CAG (Inherited: 17/17) → then remove
*   **Created new column for pathogenic length HTTCAG**
*   **Filter out all CAG lengths below HTTCAG for all 8 cases** 
*   Contraction is possible for pathogenic allele, but very rare (% of dataset?) → assume biological effects minimal
*   Leaving 54.6% of original SPN CAGLENGTH reads from 8 donors 
*   Verified that PacBio sequencing is unbiased towards either allele

**Filtered dataframe - selected columns only**

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhq1Y167PyKcshbhMcDc-3d-2U6fy1-Xe5Rq06iOocNKj_smMY4tBJ75E-WqaOfl3-_4_wBlrluD0hUaglqD--rj1iHiJBBKa8E19HTHHO202ZwEWgYDNEwkslRIJvWCv4nbgUndAJ61skaZ9FW514W7GnDIi2LNW50obc3M-pERA06MLpaQYavJMgieMOV=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEhq1Y167PyKcshbhMcDc-3d-2U6fy1-Xe5Rq06iOocNKj_smMY4tBJ75E-WqaOfl3-_4_wBlrluD0hUaglqD--rj1iHiJBBKa8E19HTHHO202ZwEWgYDNEwkslRIJvWCv4nbgUndAJ61skaZ9FW514W7GnDIi2LNW50obc3M-pERA06MLpaQYavJMgieMOV)


*   One row = one CAGLENGTH consensus read for one unique transcript in one unique cell nuclei
*   PMI : post-mortem interval
*   VS\_Grade : Vonsattel Grade – assesses severity of brain degeneration

**Visualise CAGLENGTH distribution per donor**

[![](https://blogger.googleusercontent.com/img/a/AVvXsEiKMLXgOI5GR3cmnLxN_KlrN5E7vWk3L7D1TAk3-kScysM5i7kjaOY6D9J6FS1l34dtXqfMwTDTcPXgfjnKsn1puADuIVxPDzevslLInmEGrCXgtydO7Ksm9CpA_34sYUQs44Q9d4z6g7EBXpVJmLYALyhQCvf1_oOO3GrzUQXRUTmWtl3TV5ldaZ_0G6M3=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEiKMLXgOI5GR3cmnLxN_KlrN5E7vWk3L7D1TAk3-kScysM5i7kjaOY6D9J6FS1l34dtXqfMwTDTcPXgfjnKsn1puADuIVxPDzevslLInmEGrCXgtydO7Ksm9CpA_34sYUQs44Q9d4z6g7EBXpVJmLYALyhQCvf1_oOO3GrzUQXRUTmWtl3TV5ldaZ_0G6M3)

**Find correlations**

[![](https://blogger.googleusercontent.com/img/a/AVvXsEg7KKaOIlCl0NzE7RrKOAU6dSNLnd7W_LZM2ZjgH2QbM47fECdfCon8lLTCEPZ4wuEUciIZ4G0gNg-nWdEwdCPzKIEdnsvi6Y36jMt02Lg66ArkpE7rHZk4vBCsKH-JDmou9QSMdrG2RCuTgjWjh4SF3zPNaqFjRJ8axBEspslkMdMsCHQ_AxPnB86t0hSI=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEg7KKaOIlCl0NzE7RrKOAU6dSNLnd7W_LZM2ZjgH2QbM47fECdfCon8lLTCEPZ4wuEUciIZ4G0gNg-nWdEwdCPzKIEdnsvi6Y36jMt02Lg66ArkpE7rHZk4vBCsKH-JDmou9QSMdrG2RCuTgjWjh4SF3zPNaqFjRJ8axBEspslkMdMsCHQ_AxPnB86t0hSI)  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEipV9I2rx6kTgm0TPvA4-t_k73v7QXmRYnhuZP6CmZkY44LnkIeBKE89kKhzfOVdoY9oVytSSXsFzYxPHK4T75IGULRzjvRTAyoR17HhAYKX5ybe1REOeXac1hkUYrkCitBh-Xt_WcyR-Fgz_b_LTeyoEMAIzPHnZroI0CJTmUtLOKLbDN0svW4wUCQoYc1=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEipV9I2rx6kTgm0TPvA4-t_k73v7QXmRYnhuZP6CmZkY44LnkIeBKE89kKhzfOVdoY9oVytSSXsFzYxPHK4T75IGULRzjvRTAyoR17HhAYKX5ybe1REOeXac1hkUYrkCitBh-Xt_WcyR-Fgz_b_LTeyoEMAIzPHnZroI0CJTmUtLOKLbDN0svW4wUCQoYc1)

<img width="1280" height="405" alt="bestcoverage" src="https://github.com/user-attachments/assets/9f8e84db-4489-4c99-9335-694893d252c9" />

  
**Adapt Kacher EI index to CAGLENGTH distribution per donor**
* Kacher et al., 2021 EI derived from Lee et al., 2010 instability index
* **EI = Sum of (frequency x distance from inherited HTTCAG) for all CAG lengths**

[![Expansion Index Formula](https://blogger.googleusercontent.com/img/a/AVvXsEhXMx0TYYSgjme8ShUyg8rzF6_eWApwRCRr5mwQUyrLHyI7p-tMgb4lz1puSUOB9Brx4jkxgl2lYvc9wG_HfXkGUsnGkCykqx2vv_kj22cHpL78RE1ppRRAIrUxc6ATaWVnrlJJIlUFwSzywk3vwEncHC2hxzYqE5bVRcYFJzho1jALaJjkdhf7pIjezGam)](https://blogger.googleusercontent.com/img/a/AVvXsEhXMx0TYYSgjme8ShUyg8rzF6_eWApwRCRr5mwQUyrLHyI7p-tMgb4lz1puSUOB9Brx4jkxgl2lYvc9wG_HfXkGUsnGkCykqx2vv_kj22cHpL78RE1ppRRAIrUxc6ATaWVnrlJJIlUFwSzywk3vwEncHC2hxzYqE5bVRcYFJzho1jALaJjkdhf7pIjezGam)

* Original Lee et al, 2010: fluorescence signal lower accuracy than digital counts
* filter by 20% threshold
* **No minimum threshold filtering needed for PacBio dataset (<10 nreads already filtered)**
* EI increases with increasing frequency of small expansions, and/or a few large expansions
* Create R function calculate\_EI\_weighted → output all calculations to csv

```r
# Kacher EI function for each donor
library(compiler)
calculate_baseline_EI <- cmpfun(function(data) {
    # Extract data for the current donor
    resampled_cag <- data$CAGLENGTH
    inherited <- data$HTTCAG[1]
    donor_id <- data$DONOR[1]
    
    unique_cags <- unique(resampled_cag)
    ei <- 0
    total_n <- length(resampled_cag)
    
    # Calculate EI: Sum of (frequency * distance)
    for (cag in unique_cags) {
        count <- sum(resampled_cag == cag)
        frequency <- count / total_n
        distance <- pmax(0, cag - inherited)
        ei <- ei + (frequency * distance)
    }
    
    print(paste("Donor:", donor_id, "| Baseline EI:", round(ei, 4)))
    
    return(ei)
})

# apply the function
baseline_results <- lapply(split(df_30dec_clean, df_30dec_clean$DONOR), calculate_baseline_EI)
```

```r
# can also be simplified like this
# original kacher EI formula
df$frequency <- df$count / sum(df$count) 
EI <- sum(df$frequency * df$distance)

# optimised weighted mean formula w/o intermediate division steps, yields same results
# saves a lot of time for large datasets and iterations
# [sum of count x distance] / total reads
EI <- sum(df$count * df$distance) / sum(df$count)
```

```r
[1] "Donor: S09619 | Baseline EI: 9.6595"
[1] "Donor: S05368 | Baseline EI: 12.0642"
[1] "Donor: S12365 | Baseline EI: 43.2086"
[1] "Donor: S02205 | Baseline EI: 58.07"
[1] "Donor: S04002 | Baseline EI: 54.9917"
[1] "Donor: S05202 | Baseline EI: 74.3518"
[1] "Donor: S07681 | Baseline EI: 35.2658"
[1] "Donor: S06758 | Baseline EI: 48.1925"
```

**Low cell counts → inflates weightage of sampled long CAGLENGTHs → inflates actual EI!**

![](https://blogger.googleusercontent.com/img/a/AVvXsEi5AmDRItUxFOUFSfHTwZNqxIA6y-F0LkpMSmgMZph4sfa_-St01Q-dGetAB0m_bRDGPp6IRm2ird1WWDddmVjvpdjct3kbHAKMK_0BjQZS_FNUeLnBmP21kVUdFFatWjCVjHDIdjipTGasMKcUb7mzWS0G3s-WcVZh4HoaSlrjSo6N9PL_D_dcz1NH6zY1=s16000)
  
**How do we accurately quantify this range of EI uncertainty for each donor?**
**We need to bootstrap for an EI range** 

**First let's find out if there's any undersampling of CAGLENGTH after accounting for known factors**   
*  Let's try out different correlation models.
```r
library(dplyr)
library(ggplot2)
library(gridExtra) 

donor_stats <- df_30dec_clean %>%
    group_by(DONOR) %>%
    summarise(
        total_rows = n(),
        mean_gap = if(n_distinct(CAGLENGTH) > 1) mean(diff(sort(unique(CAGLENGTH)))) else NA_real_
    ) %>%
    filter(!is.na(mean_gap))

# fit models
m_lin <- lm(mean_gap ~ total_rows, data = donor_stats)
m_log <- lm(mean_gap ~ log(total_rows), data = donor_stats)
m_inv <- lm(mean_gap ~ I(1/total_rows), data = donor_stats)
m_pow <- lm(log(mean_gap) ~ log(total_rows), data = donor_stats)

# create plots
create_plot <- function(model, title, color, is_power=FALSE) {
    r2 <- round(summary(model)$r.squared, 3)
    
    # Generate predictions for smooth line
    x_seq <- seq(min(donor_stats$total_rows), max(donor_stats$total_rows), length.out = 100)
    
    if (is_power) {
        preds <- exp(predict(model, newdata = data.frame(total_rows = x_seq)))
    } else {
        preds <- predict(model, newdata = data.frame(total_rows = x_seq))
    }
    
    pred_df <- data.frame(total_rows = x_seq, mean_gap = preds)
    
    ggplot() +
        geom_point(data = donor_stats, aes(x = total_rows, y = mean_gap), size = 2) +
        geom_line(data = pred_df, aes(x = total_rows, y = mean_gap), color = color, linewidth = 1) +
        labs(title = paste0(title, " (R² = ", r2, ")"), 
             x = "Total Reads", y = "Mean Gap (bp)") +
        theme_minimal() +
        theme(plot.title = element_text(size = 10, face = "bold"))
}

# generate 4 plots in one output
p1 <- create_plot(m_lin, "Linear: y = a + bx", "#E41A1C")
p2 <- create_plot(m_log, "Logarithmic: y = a + b*ln(x)", "#377EB8")
p3 <- create_plot(m_inv, "Inverse: y = a + b/x", "#4DAF4A")
p4 <- create_plot(m_pow, "Power: y = ax^b", "#984EA3", is_power=TRUE)

g <- arrangeGrob(p1, p2, p3, p4, nrow = 2)
ggsave("meangapcorrelations.png", plot = g, width = 10, height = 8, dpi = 300)
grid.arrange(p1, p2, p3, p4, nrow = 2)
```
<img width="900" height="700" alt="10c9ce4e-8c55-431a-84a6-bfec8ec15097" src="https://github.com/user-attachments/assets/bb55fbd3-b450-4c11-a907-aebcc5fd09c2" />

**Evidence of undersampling**
*   HTTCAG, Age, disease stage should correlate with increased range – but weak correlation in this dataset
*   All 3 factors should correlate with SPN loss – reduced total reads – weak correlation
*   SPN loss possibly slightly reduced range – weak correlation
*   **But some correlation of total reads with 1. range and 2. continuity of data (mean gap)**
*   Log linear fit makes the most sense 
*   Sufficient CAGLENGTH sampling would close the mean gap towards 1, assuming continuous CAGLENGTH distribution in each donor

```r
library(dplyr)
  library(ggplot2)
  
  # calc mean gap per donor
  # Logic: Sort unique CAGs -> measure distance between neighbors -> average them
  donor_stats <- df_30dec_clean %>%
    group_by(DONOR) %>%
    summarise(
      total_rows = n(),
      mean_gap = mean(diff(sort(unique(CAGLENGTH))), na.rm = TRUE)
    )
  
  # fit log linaer model
  model <- lm(mean_gap ~ log(total_rows), data = donor_stats)
  s <- summary(model)

# print all stats  
print(summary(model))
  
  ggplot(donor_stats, aes(x = total_rows, y = mean_gap)) +
    geom_point(color = "#3d5c26", size = 3) + 
    geom_smooth(method = "lm", formula = y ~ log(x), color = "black", fill = "grey90") +
    theme_minimal() +
    labs(title = "Somatic Mosaicism Density",
         subtitle = "Convergence to 1 bp gap with increased read depth",
         x = "Total Reads (Depth)",
         y = "Mean Gap between Unique CAGs (bp)")
```
<img width="600" height="480" alt="image" src="https://github.com/user-attachments/assets/bf0bcc8a-0030-4503-b856-fdb0a5fef4b5" />

```r
# regression stats
Equation:    mean_gap = 16.873 -1.985 * log(total_rows)
R²:          0.497
p-value:     0.0509
F-statistic: 5.923
```

[![Sampling Evidence](https://blogger.googleusercontent.com/img/a/AVvXsEhZ5I574NVv0Yo3GTwujlFlYbsCJs4XNnnkeJLpw866ER8JNMPEbomhF_XCKqfPDFlnVtVQ-ddETPfdRtwX7-OlhK2rrQq5LVAO7pVtIu8n4agdqN53dZCIe4wzErhS2OLggqQC3ja2OSoshJBEpT176Am32cH7dOhLT9Xp952-MULGTXdpPoXueLh41VM7=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEhZ5I574NVv0Yo3GTwujlFlYbsCJs4XNnnkeJLpw866ER8JNMPEbomhF_XCKqfPDFlnVtVQ-ddETPfdRtwX7-OlhK2rrQq5LVAO7pVtIu8n4agdqN53dZCIe4wzErhS2OLggqQC3ja2OSoshJBEpT176Am32cH7dOhLT9Xp952-MULGTXdpPoXueLh41VM7=s16000)

**How to bootstrap**
[![Bootstrapping Convergence](https://blogger.googleusercontent.com/img/a/AVvXsEjyCznKY0LSPgxNxT2YfohxXXWwbuaqdTFuziSsNGVWNnDI0arqYyGj-zWqNPCnENifSxnu2RKTbX9wR5RQyJhsTh74towrYyfRtxC04ti2vfoLGxYfQfs9aNvH__hnUFEU9UiO0ZkzKQntn2hZM7BJ9-CqTOlIocmvmthxJ0D0TdlrGe7z3GLqmQjoa7Ta)](https://blogger.googleusercontent.com/img/a/AVvXsEjyCznKY0LSPgxNxT2YfohxXXWwbuaqdTFuziSsNGVWNnDI0arqYyGj-zWqNPCnENifSxnu2RKTbX9wR5RQyJhsTh74towrYyfRtxC04ti2vfoLGxYfQfs9aNvH__hnUFEU9UiO0ZkzKQntn2hZM7BJ9-CqTOlIocmvmthxJ0D0TdlrGe7z3GLqmQjoa7Ta)

**Let's do B=1K and B=10K first with 8 threads**
* Note: set seed for reproducibility as needed. 

```r
library(boot)
library(compiler)
library(parallel) 

# define bca bootstrap function
run_donor_bootstrap <- function(df) {
  donors <- unique(df$DONOR)
  
  for(d in donors) {
    # Subset specific donor from passed dataframe
    sub_df <- df[df$DONOR == d, ]
    
    # dat[idx, ] resamples the dataframe rows for the bootstrap
    boot_fn <- function(dat, idx) {
      capture.output(val <- calculate_baseline_EI(dat[idx, ]))
      return(val)
    }
    
    tryCatch({
      b_res <- boot(data = sub_df, 
                    statistic = boot_fn, 
                    R = 1000, 
                    parallel = "multicore", 
                    ncpus = 8)
      
      b_ci <- boot.ci(b_res, type = "bca")
      
      # extract values
      orig <- b_res$t0
      lower <- b_ci$bca[4]
      upper <- b_ci$bca[5]
      
      cat("DONOR:", d, 
          "| Orig:", round(orig, 4), 
          "| 95% CI: [", round(lower, 4), "-", round(upper, 4), "]\n", sep="")
      
    }, error = function(e) cat("Error for donor", d, ":", e$message, "\n")) 
  } 
} 

# time it
start_time <- Sys.time()
cat("Starting Bootstrap...", "\n")

# point function to df_30dec_clean
run_donor_bootstrap(df_30dec_clean)

end_time <- Sys.time()
print(end_time - start_time)
```

```r
# B=1K bca bootstrap results
# original Kacher EI formula 
Starting Bootstrap... 
> 
> point to df_30dec_clean
> run_donor_bootstrap(df_30dec_clean)
DONOR:S05202| Orig:74.3518| 95% CI: [67.996-81.2424]
DONOR:S12365| Orig:43.2086| 95% CI: [39.8332-46.9994]
DONOR:S07681| Orig:35.2658| 95% CI: [31.4473-40.2068]
DONOR:S06758| Orig:48.1925| 95% CI: [45.0215-51.5439]
DONOR:S04002| Orig:54.9917| 95% CI: [49.314-62.7339]
DONOR:S09619| Orig:9.6595| 95% CI: [8.0331-11.7146]
DONOR:S02205| Orig:58.07| 95% CI: [51.4572-65.9116]
DONOR:S05368| Orig:12.0642| 95% CI: [8.7903-19.601]
> 
> end_time <- Sys.time()
> print(end_time - start_time)
Time difference of 26.81494 secs
```

```r
# B=10K bca bootstrap results
# original EI formula
Starting Bootstrap... 
> 
> # point to df_30dec_clean
> run_donor_bootstrap(df_30dec_clean)
DONOR:S05202| Orig:74.3518| 95% CI: [68.1604-81.6007]
DONOR:S12365| Orig:43.2086| 95% CI: [39.8202-47.0981]
DONOR:S07681| Orig:35.2658| 95% CI: [31.5702-39.8297]
DONOR:S06758| Orig:48.1925| 95% CI: [45.0555-51.5711]
DONOR:S04002| Orig:54.9917| 95% CI: [49.1324-62.1261]
DONOR:S09619| Orig:9.6595| 95% CI: [8.0738-11.8339]
DONOR:S02205| Orig:58.07| 95% CI: [51.2438-65.9643]
DONOR:S05368| Orig:12.0642| 95% CI: [8.8556-18.8172]
> 
> end_time <- Sys.time()
> print(end_time - start_time)
Time difference of 3.112793 mins
```
*  Here you can see the efficiency difference with optimised EI formula, half the time. 
*  slight variability in CI due to random variability, seed was not set this time.
*  but we can see between random seeds the values are quite close, we'll validate twin runs within B values later.

```r
# run bca bootstrap with optimised EI formula
Starting Bootstrap (B=10,000)... 

> run_donor_bootstrap_simplified(df_30dec_clean)
DONOR:S05202| Orig:74.3518| 95% CI: [68.0812-81.4656]
DONOR:S12365| Orig:43.2086| 95% CI: [39.8479-47.0207]
DONOR:S07681| Orig:35.2658| 95% CI: [31.4849-39.8023]
DONOR:S06758| Orig:48.1925| 95% CI: [45.1548-51.5034]
DONOR:S04002| Orig:54.9917| 95% CI: [49.1023-62.1394]
DONOR:S09619| Orig:9.6595| 95% CI: [8.0902-11.8095]
DONOR:S02205| Orig:58.07| 95% CI: [51.6532-66.2867]
DONOR:S05368| Orig:12.0642| 95% CI: [8.8612-19.121]
> 
> end_time <- Sys.time()
> print(end_time - start_time)
Time difference of 1.438339 mins
```
**note to self: offload to GPU for efficiency for larger datasets**
* Using Positron - import CuPy to connect to GPU RTX 5070
* Convert needed columns to cupy array to transfer to VRAM, set dtype='float32'
* Convert R fxn to python fxn
* The 10k bootstrap generated in one GPU operation (probably in seconds for 7k cells)
* Set cp.percentile(replicates, [2.5, 97.5]) on the GPU for 95% CI if trf data back to R for plotting

[![](https://blogger.googleusercontent.com/img/a/AVvXsEh5TbUWfm2KNOdKQZr_5O4dnsDqsxrUYz78MRb6oF2nL7kvSIMa5yIdqHxO69578AJnVB7axGsTnZHClyIf7XNwOlY73QjVkDh3QWe5wS71HTohjJ2RNsRfsvcBfiD3Gemw8SYDutPhsffpLwA74Mr8DLqyR5ijSm0XuChjrBHAynvWoix3MOgqH2N7HIBe=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEh5TbUWfm2KNOdKQZr_5O4dnsDqsxrUYz78MRb6oF2nL7kvSIMa5yIdqHxO69578AJnVB7axGsTnZHClyIf7XNwOlY73QjVkDh3QWe5wS71HTohjJ2RNsRfsvcBfiD3Gemw8SYDutPhsffpLwA74Mr8DLqyR5ijSm0XuChjrBHAynvWoix3MOgqH2N7HIBe)

**Fantastic CI convergence. Let's see if we can get away with lower B for computational efficiency! And also check convergence of random variability for twin runs within each B value.** 

[![](https://blogger.googleusercontent.com/img/a/AVvXsEiBRIUtHYL_e6BgbYPvhRsDsglYs6IRfdCY1W2DwC95svLTC7GH8YV97gxFrncDWXgmYeUKzHuA5U8x1H1olYm3gG2O6tJmS-U1m1oTZ4EF09uMs0Y_wVXxriCNjagA7iWPQzh7HFSkQNpyFqxMdLfyD1cSg3LmZxWkTOiPwzY-hfPdKiRhf9e9jR_Gw8_1=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEiBRIUtHYL_e6BgbYPvhRsDsglYs6IRfdCY1W2DwC95svLTC7GH8YV97gxFrncDWXgmYeUKzHuA5U8x1H1olYm3gG2O6tJmS-U1m1oTZ4EF09uMs0Y_wVXxriCNjagA7iWPQzh7HFSkQNpyFqxMdLfyD1cSg3LmZxWkTOiPwzY-hfPdKiRhf9e9jR_Gw8_1)

* **B=1K is the minimum we have to aim for.**
* MAD is a non standard measure we're just trying out here for clinical translation purposes. 
* Just because there's strong CI convergence doesn't mean that it is clinically precise enough.   

**Find some clinical correlations.**  

<img width="800" height="280" alt="donormetrics" src="https://github.com/user-attachments/assets/6838db06-2bd5-40ab-a096-9ddc39aedfe7" />

Note: this is from a different run with a different random seed - hence the slight differences against the console results above. Good consistency across different seeds within same B.

**Generate heatmap of EI range vs inherited CAG, age, disease stage** 

<img width="450" height="1050" alt="ANNOTheatmapupdated" src="https://github.com/user-attachments/assets/5d787889-39d2-46e6-8bad-5fff5e232590" />

(EI range added in manually after plot generated in R, just this time. For many donors, better to code the EI range into the coloured squares.)
* If we had enough individuals to fill the entire heatmap, we could expect the gradient to go from blue to dark red from bottom left quadrant to top right quadrant
* despite the small cohort of 8 donors, this was intentionally selected from the larger original cohort with representativeness in mind 
* In general, EI positively correlates with higher inherited HTTCAG and age. 
* Current EI is a snapshot of past average expansion, and may be able to predict future expansion. 
* Genetic modifiers may change expansion rate, i.e. 'silent' CAA (also Q) within the CAG (Q) region interrupts somatic expansion.
* Clinical utility of postmortem EI: Good correlation with disease stage at death even for 8 donors
* For this sample: Asymptomatic to symptomatic EI range appear to fall between 20 to 30, though HTTCAG/Age must be accounted for when creating a clinical metric
* EI of 15 may be asymptomatic for someone with HTTCAG 40, but may be symptomatic for someone with HTTCAG 45, as the HTTCAG 45 individual has many more SPNs past the pathogenic threshold
* the same EI is more concerning for a younger individual than an older individual 

```r
library(ggplot2)
library(dplyr)

# EI heatmap
plot_data <- df_30dec_clean %>%
  mutate(
    Age = floor(as.numeric(as.character(Age))), 
    HTTCAG = as.numeric(as.character(HTTCAG)),
    
# sort VS_Grade correctly
    VS_Grade = factor(VS_Grade, 
                      levels = c("n/a", "HD-0", "HD-1", "HD-2", "HD-3"))
  )

# colour palette VS_Grade
custom_colors <- c(
  "n/a"  = "blue",
  "HD-0" = "#fdae6b",   
  "HD-1" = "#f16913",   
  "HD-2" = "#cb181d",  
  "HD-3" = "#67000d"   
)

# background grid
background_grid <- expand.grid(
  Age = 37:85,
  HTTCAG = 37:43
)

p <- ggplot() +
  geom_tile(data = background_grid, 
            aes(x = HTTCAG, y = Age),
            fill = "grey90", color = "white", lwd = 0.5,
            width = 1, height = 1) +
  
  geom_tile(data = plot_data, 
            aes(x = HTTCAG, y = Age, fill = VS_Grade),
            color = "white", lwd = 0.5,
            width = 1, height = 1) +
  
  scale_fill_manual(
    values = custom_colors, 
    na.value = "blue", 
    name = "Disease Stage"
  ) +
  
  scale_x_continuous(
    breaks = seq(37, 43, 1), 
    limits = c(36.5, 43.5), 
    expand = c(0, 0)
  ) +
  
  scale_y_continuous(
    breaks = seq(37, 84, 1), 
    limits = c(36.5, 84.5), 
    expand = c(0, 0)
  ) +
  
  theme_minimal() +
  labs(
    title = "Expansion Index Heatmap",
    x = "Inherited HTTCAG",
    y = "Age"
  ) +
  theme(
    panel.grid = element_blank(),
    legend.position = "right",
    axis.text.y = element_text(size = 10),
    axis.text.x = element_text(size = 10),
    axis.title = element_text(size = 14, face = "bold"),
    legend.text = element_text(size = 13),
    legend.title = element_text(size = 14, face = "bold"),
    plot.title = element_text(size = 16, face = "bold"),
  )

print(p)

timestamp <- format(Sys.time(), "%Y%m%d_%H%M%S")
filename <- paste0("heatmap", timestamp, ".png")
ggsave(filename, plot = p, width = 6, height = 14, dpi = 300)
```

**Compare gene expression profiles across EI ranges**
* sparsify gene xp data as needed
* switch to Positron instead

<div align="center"> · · · · · · · · · · · </div>

**OS** * ASUS Tuf A18 | 64 GB RAM 4800 Mhz CL40 | AMD Ryzen 7260 | NVIDIA GeForce RTX 5070 | Run locally

**Environment** * R version 4.5.2 (2025-10-31 ucrt) | Win11 Home

**References**


Handsaker, R. E., Kashin, S., Reed, N. M., Tan, S., Lee, W. S., McDonald, T. M., Morris, K., Kamitaki, N., Mullally, C. D., Morakabati, N. R., Goldman, M., Lind, G., Kohli, R., Lawton, E., Hogan, M., Ichihara, K., Berretta, S., & McCarroll, S. A. (2025). Long somatic DNA-repeat expansion drives neurodegeneration in Huntington's disease. _Cell_, _188_(3), 623–639.e19. [https://doi.org/10.1016/j.cell.2024.11.038](https://doi.org/10.1016/j.cell.2024.11.038 )









