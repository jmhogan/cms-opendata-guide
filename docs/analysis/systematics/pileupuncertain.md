# Pileup Uncertainty

!!! Warning
    This page is under construction

When proton bunches cross in CMS for collisions, there is routinely more than one collision! The [CMS Pile-up simulation guide](https://opendata.cern.ch/docs/cms-guide-pileup-simulation) on the Open Data Portal provides information about how CMS simulates pileup and links to the public information about pileup distributions used in CMS over time. That guide also shares how to access relevant pileup variables in different CMS file formats.

The pileup distribution is based on a measurement of the total inelasic cross section, or the overall rate at which protons will have an inelastic collision given the beam conditions. This measurement carries uncertainties, which should be propagated to simulations.

=== "Run 1 data"

    The pileup uncertainty can be calculated using the following steps:

    1. Download the "by luminosity section" luminosity information from the portal, as well as the validated runs luminosity JSON files:

     - 2011 data: [2011lumibyls_pxl.csv](https://opendata.cern.ch/record/1053/files/2011lumibyls_pxl.csv) and [list of validated runs](https://opendata.cern.ch/record/1001)
     - 2012 data: [2012lumibyls_pxl.csv](https://opendata.cern.ch/record/1054/files/2012lumibyls_pxl.csv) and [list of validated runs](https://opendata.cern.ch/record/1002)

    Place the files in your CMSSW directory. 

    2. Run the `estimatePileup_makeJSON.py` script in CMSSW, providing the `.csv` file you downloaded as a command-line argument. (Note: this script is available in CMSSW_5_3_32, but not earlier versions.)

    ```bash
    estimate_pileup_makeJSON.py --csvInput 2011lumibyls.csv pileup_JSON.txt
    ```

    Note: you do not need to explicitly run this script using `python` -- it is precompiled in CMSSW.

    3. Run the `pileupCalc.py` script to prepare pileup histograms, providing your `pileup_JSON.txt` file and the relevant list of validated runs file as command-line arguments. There are two other important command-line arguments to provide:

     - the inelastic cross section, `--minBiasXsec`: This value was **68000** $\mu$b for 2011 and **69400** $\mu$b for 2012. These should both be varied by **5%** as a systematic uncertainty in the pileup simulation.
     - the maximum number of pileup interactions and number of bins for the histograms. These inputs should typically match each other, and a value of 50 is reasonable for Run 1 data.

    For example, to create the required histograms for 2011 data, you would execute:

    ```bash
    pileupCalc.py -i 2011_listOfValidatedRuns.txt --inputLumiJSON pileup_JSON.txt --calcMode true --minBiasXsec 68000 --maxPileupBin 50 --numPileupBins 50 MyDataPileupHistogram.root
    pileupCalc.py -i 2011_listOfValidatedRuns.txt --inputLumiJSON pileup_JSON.txt --calcMode true --minBiasXsec 71400 --maxPileupBin 50 --numPileupBins 50 MyDataPileupHistogram_MinBiasXsecUp.root
    pileupCalc.py -i 2011_listOfValidatedRuns.txt --inputLumiJSON pileup_JSON.txt --calcMode true --minBiasXsec 64600 --maxPileupBin 50 --numPileupBins 50 MyDataPileupHistogram_MinBiasXsecDown.root
    ```

    4. Calculate the weight needed for simulation. The ROOT files created in step 3 are histograms of the number of events per number of pileup interactions. The horizontal axis corresponds to `PileupSummaryInfo::getTrueNumInteractions()` from AOD files (see the pileup guide linked at the top of this page!).

    You can create a corresponding histogram for any Monte Carlo simulation that you are using by storing the results of `PileupSummaryInfo::getTrueNumInteractions()` from all the events (with no filtering applied) in several ROOT files (use enough to get a smooth histogram that populates the full range expected for a given year. Monte Carlo simulations that were produced with common conditions should have the same pileup distribution.

    With all four histograms in hand, compute the following three sets of weights:

     - central pileup weight = (central data histogram, normalized to unit area) / (MC histogram, normalized to unit area)
     - up-shifted pileup weight = (up-shifted data histogram, normalized to unit area) / (MC histogram, normalized to unit area)
     - down-shifted pileup weight = (down-shifted data histogram, normalized to unit area) / (MC histogram, normalized to unit area)

    5. Apply the weights to simulation: the result of step 4 is a histogram (or text list) of weights as a function of the number of pileup interactions. When processing simulation, use the number of pileup interactions to determine the correct event weights. The central weight should be multiplied with any other event weights in your analysis. The shifted weights should also be multiplied with any other event weights, but stored as a separate pair of weights used to fill separate histograms or prepare separate event counts as a systematic uncertainty.

=== "Run 2 data"

    In Run 2, the pileup weights are available via the `correctionlib` platform!

    For 2015 data, the instructions provided for Run 1 will also work to produce your own set of pileup weights. In this case the total inelastic cross section should be **69200** $\mu$b, with an uncertainty of **4.6%**. More detailed instructions are coming for using correctionlib for 2015 data.

    For 2016 data this [workshop lesson on systematic uncertainties](https://cms-opendata-workshop.github.io/workshop2024-lesson-uncertainties/instructor/index.html) shows how to extract and apply the pileup uncertainty. This example uses NanoAOD files, but the same techniques could be used for the pileup uncertainty if the `PileupSummaryInfo::getTrueNumInteractions()` value is stored when processing MiniAOD files to produce simpler ROOT trees for further analysis.
