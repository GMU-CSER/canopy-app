# Biogenic Emissions Workflow

This document describes the workflow for calculating biogenic emissions in Canopy-App.

```mermaid
graph TD
    subgraph Initialization
        A[Read Namelist Settings] --> B[Initialize Parameters]
    end

    subgraph Main_Calculation_Loop [canopy_calcs.F90]
        C[Check ifcanbio == .TRUE.] --> D{Iterate Species}
        D --> E[Isoprene, Myrcene, Sabinene, etc.]
    end

    subgraph Biogenic_Module [canopy_bioemi_mod.F90]
        E --> F[Retrieve Species Parameters]
        F --> G[Calculate Activity Factors]

        subgraph Activity_Factors
            G --> G1[TLeaf Activity - Sun/Shade]
            G --> G2[PPFD Activity - Sun/Shade]
            G1 --> G3[Combined Activity Factor]
            G2 --> G3
        end

        G3 --> H[Calculate Stress/Inhibition Factors]

        subgraph Gamma_Factors
            H --> H1[CO2 Inhibition - Isoprene only]
            H --> H2[Soil Moisture Stress]
            H --> H3[Leaf Age Response]
            H --> H4[Air Quality Stress]
            H --> H5[Temperature Stress - High/Low]
            H --> H6[Wind Speed Stress]
        end

        H1 & H2 & H3 & H4 & H5 & H6 --> I[Vertical Integration / Summing]

        subgraph Vertical_Options [biovert_opt]
            I --> I0[Option 0: Full 3D Leaf-Level]
            I --> I1[Option 1: MEGANv3-like Sum]
            I --> I2[Option 2: Gaussian Sum]
            I --> I3[Option 3: Even Sum]
        end

        I0 & I1 & I2 & I3 --> J[Apply Canopy Loss Factor]
        J --> K[Unit Conversion to kg m-3 s-1 or kg m-2 s-1]
    end

    K --> L[Output to 1D Text or 2D NetCDF]
```

## Detailed Steps

1.  **Parameter Retrieval**: The `canopy_biop` subroutine in `canopy_bioparm_mod.F90` provides emission factors (EF) and algorithm coefficients (LDF, Beta, etc.) based on MEGAN2.1.
2.  **Activity Factors**: Leaf-level activity depends on leaf temperature (TLeaf) and photosynthetically active radiation (PPFD) for both sunlit and shaded fractions.
3.  **Stress Factors**: Gamma factors account for environmental stresses including soil moisture, leaf age, ozone exposure (W126), and extreme meteorological conditions.
4.  **Vertical Integration**: Depending on `biovert_opt`, emissions are either kept at the leaf level throughout the canopy profile or integrated into a total flux.
5.  **Loss Factor**: A canopy loss factor is applied when summing emissions to account for in-canopy processing.
6.  **Conversion**: Final emissions are converted from $\mu g/m^2/hr$ to model units ($kg/m^3/s$ or $kg/m^2/s$).
