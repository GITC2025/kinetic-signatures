**From Postmortem to Clinical: Kinetic Signatures of CAG Somatic Expansion in Huntington's Disease via snRNA-seq Analysis**

* Reanalysis of Handsaker et al., 2025 PacBio CCS dataset + adaptation of Kacher Expansion Index formula (2021)
* Origins: Personal project to learn R studio for genomic analysis
* Timeline of project: Dec 2025 to Jan 2026 (one month+)

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

<img src="https://blogger.googleusercontent.com/img/a/AVvXsEj_rpKe0C7jOfTHpo0KWr8I70o6x1UpZ9yDL32L-YU7RnmEz1awZbFrmEX5ZHYNrBiGZ_3Ztz85erdiWW1yCHfkZpd4yevWgEX3UFEkqg-BU6bIzkv7VtW6Rgz9n9_0ivaSyczMb2dmjM2jTn2-FgHB8vJPnzC-1Yb_A_O3XWbnyV6iykgtJePWcla407Xn" width="500">

**QC before renalysis**
*   pacbio\_deepdives\_cells\_unfiltered : contains CAGLENGTH data for 9 donors
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
calculate_EI_weighted <- cmpfun(function(data, indices) {
    resampled_cag <- data$CAGLENGTH[indices]
    inherited <- data$HTTCAG[1]
    
    unique_cags <- unique(resampled_cag)
    ei <- 0
    total_n <- length(resampled_cag)
    
    for (cag in unique_cags) {
        count <- sum(resampled_cag == cag)
        frequency <- count / total_n
        distance <- pmax(0, cag - inherited)
        ei <- ei + (frequency * distance)
    }
    
    return(ei)
})
```

**Low cell counts → inflates weightage of sampled long CAGLENGTHs → inflates actual EI!**

![](https://blogger.googleusercontent.com/img/a/AVvXsEi5AmDRItUxFOUFSfHTwZNqxIA6y-F0LkpMSmgMZph4sfa_-St01Q-dGetAB0m_bRDGPp6IRm2ird1WWDddmVjvpdjct3kbHAKMK_0BjQZS_FNUeLnBmP21kVUdFFatWjCVjHDIdjipTGasMKcUb7mzWS0G3s-WcVZh4HoaSlrjSo6N9PL_D_dcz1NH6zY1=s16000)
  
**How do we accurately quantify this range of EI uncertainty for each donor?**
**We need to bootstrap for an EI range** 
**First let's find out if there's any undersampling of CAGLENGTH after accounting for known factors**   

**Evidence of undersampling**
*   HTTCAG, Age, disease stage should correlate with increased range – but weak correlation in this dataset
*   All 3 factors should correlate with SPN loss – reduced total reads – weak correlation
*   SPN loss possibly slightly reduced range – weak correlation
*   **But some correlation of total reads with 1. range and 2. continuity of data (mean gap)**
*   Sufficient CAGLENGTH sampling would close the mean gap towards 1, assuming continuous CAGLENGTH distribution in each donor

[![Sampling Evidence](https://blogger.googleusercontent.com/img/a/AVvXsEhZ5I574NVv0Yo3GTwujlFlYbsCJs4XNnnkeJLpw866ER8JNMPEbomhF_XCKqfPDFlnVtVQ-ddETPfdRtwX7-OlhK2rrQq5LVAO7pVtIu8n4agdqN53dZCIe4wzErhS2OLggqQC3ja2OSoshJBEpT176Am32cH7dOhLT9Xp952-MULGTXdpPoXueLh41VM7=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEhZ5I574NVv0Yo3GTwujlFlYbsCJs4XNnnkeJLpw866ER8JNMPEbomhF_XCKqfPDFlnVtVQ-ddETPfdRtwX7-OlhK2rrQq5LVAO7pVtIu8n4agdqN53dZCIe4wzErhS2OLggqQC3ja2OSoshJBEpT176Am32cH7dOhLT9Xp952-MULGTXdpPoXueLh41VM7=s16000)

**How to bootstrap**
[![Bootstrapping Convergence](https://blogger.googleusercontent.com/img/a/AVvXsEjyCznKY0LSPgxNxT2YfohxXXWwbuaqdTFuziSsNGVWNnDI0arqYyGj-zWqNPCnENifSxnu2RKTbX9wR5RQyJhsTh74towrYyfRtxC04ti2vfoLGxYfQfs9aNvH__hnUFEU9UiO0ZkzKQntn2hZM7BJ9-CqTOlIocmvmthxJ0D0TdlrGe7z3GLqmQjoa7Ta)](https://blogger.googleusercontent.com/img/a/AVvXsEjyCznKY0LSPgxNxT2YfohxXXWwbuaqdTFuziSsNGVWNnDI0arqYyGj-zWqNPCnENifSxnu2RKTbX9wR5RQyJhsTh74towrYyfRtxC04ti2vfoLGxYfQfs9aNvH__hnUFEU9UiO0ZkzKQntn2hZM7BJ9-CqTOlIocmvmthxJ0D0TdlrGe7z3GLqmQjoa7Ta)

**Let's do B=1K and B=10K first with 8 threads**

[![](https://blogger.googleusercontent.com/img/a/AVvXsEh5TbUWfm2KNOdKQZr_5O4dnsDqsxrUYz78MRb6oF2nL7kvSIMa5yIdqHxO69578AJnVB7axGsTnZHClyIf7XNwOlY73QjVkDh3QWe5wS71HTohjJ2RNsRfsvcBfiD3Gemw8SYDutPhsffpLwA74Mr8DLqyR5ijSm0XuChjrBHAynvWoix3MOgqH2N7HIBe=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEh5TbUWfm2KNOdKQZr_5O4dnsDqsxrUYz78MRb6oF2nL7kvSIMa5yIdqHxO69578AJnVB7axGsTnZHClyIf7XNwOlY73QjVkDh3QWe5wS71HTohjJ2RNsRfsvcBfiD3Gemw8SYDutPhsffpLwA74Mr8DLqyR5ijSm0XuChjrBHAynvWoix3MOgqH2N7HIBe)

**Fantastic CI convergence. Let's see if we can get away with lower B for computational efficiency! And also check convergence of random variability for twin runs within each B value.** 

[![](https://blogger.googleusercontent.com/img/a/AVvXsEiBRIUtHYL_e6BgbYPvhRsDsglYs6IRfdCY1W2DwC95svLTC7GH8YV97gxFrncDWXgmYeUKzHuA5U8x1H1olYm3gG2O6tJmS-U1m1oTZ4EF09uMs0Y_wVXxriCNjagA7iWPQzh7HFSkQNpyFqxMdLfyD1cSg3LmZxWkTOiPwzY-hfPdKiRhf9e9jR_Gw8_1=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEiBRIUtHYL_e6BgbYPvhRsDsglYs6IRfdCY1W2DwC95svLTC7GH8YV97gxFrncDWXgmYeUKzHuA5U8x1H1olYm3gG2O6tJmS-U1m1oTZ4EF09uMs0Y_wVXxriCNjagA7iWPQzh7HFSkQNpyFqxMdLfyD1cSg3LmZxWkTOiPwzY-hfPdKiRhf9e9jR_Gw8_1)

* **B=1K is the minimum we have to aim for.**
* MAD is a non standard measure we're just trying out here for clinical translation purposes. 
* Just because there's strong CI convergence doesn't mean that it is clinically precise enough.   

**Find some clinical correlations.**  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhNdirOLxzGxcdo_dqNaFg4MyFRtolWmHfqkZp-vcD5Z1YDdXo22fI3D0EweI6Ie6_TZ6t67RVNuohudjctXI464fZnixFVvBgLTeFYEfUH0i37txA4GzROUFitcqLV8WIxzUNCOtUrEySkkRAqoriNz4mbbNg-1NETw7ZFV3vEnAKHzWRxc30B-_5gnUuT=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEhNdirOLxzGxcdo_dqNaFg4MyFRtolWmHfqkZp-vcD5Z1YDdXo22fI3D0EweI6Ie6_TZ6t67RVNuohudjctXI464fZnixFVvBgLTeFYEfUH0i37txA4GzROUFitcqLV8WIxzUNCOtUrEySkkRAqoriNz4mbbNg-1NETw7ZFV3vEnAKHzWRxc30B-_5gnUuT)  

**Generate heatmap of EI range vs inherited CAG, age, disease stage** (in progress)

  

**References**

Handsaker, R. E., Kashin, S., Reed, N. M., Tan, S., Lee, W. S., McDonald, T. M., Morris, K., Kamitaki, N., Mullally, C. D., Morakabati, N. R., Goldman, M., Lind, G., Kohli, R., Lawton, E., Hogan, M., Ichihara, K., Berretta, S., & McCarroll, S. A. (2025). Long somatic DNA-repeat expansion drives neurodegeneration in Huntington's disease. _Cell_, _188_(3), 623–639.e19. [https://doi.org/10.1016/j.cell.2024.11.038](https://doi.org/10.1016/j.cell.2024.11.038 )





