# Muons

## Introduction

Muons are measured in the CMS experiment combining the information from the [inner tracker](https://cms.cern/index.php/detector/identifying-tracks) and the [muon system](https://cms.cern/detector/detecting-muons). The signals from these systems are processed with CMSSW through [subsequent steps](../../../cmssw/cmsswdatamodel.md) to form muon candidates which are then available in the muon collection of the data files.

In the standard CMS reconstruction for proton-proton collisions, tracks are first reconstructed independently in the inner tracker and in the muon system. Based on these objects, three reconstruction approaches are used:

- **Tracker Muon** reconstruction:  all tracker tracks with pT > 0.5 GeV/c and total momentum p > 2.5 GeV/c are considered as possible muon candidates, and are extrapolated to the muon system taking into account the magnetic field;

- **Standalone Muon** reconstruction: all tracks of the segments reconstructed in the muon chambers (performed using segments and hits from Drift Tubes in the barrel region, Cathode Strip Chambers and Resistive Plates Chambers in the endcaps) are used to generate “seeds” consisting of position and direction vectors and an estimate of the muon transverse momentum;

- **Global Muon** reconstruction: starts from a Standalone reconstructed muon track and extrapolates its trajectory from the innermost muon station through the coil and both calorimeters to the outer tracker surface.

![Muon identification](../../../images/analysis/selection/idefficiencystudy/tagandprobe/muons_id.png)

## Access to muon information

The [Physics Objects page](./objects.md) shows how to access muon collections, and which header files should be included in the C++ code in order to access all of their class information. The [Common Tools page](./tools.md) gives instructions to access all the basic kinematic information about any physics object.

## Muon identification

=== "Run 1 Data (AOD files)"

    As explained in the [Physics Object page](./objects#detector-information-for-identification), a mandatory task in the physics analysis is to identify muons, i.e. to separate “real” objects from “fakes”. The criteria depend on the type of analysis. The muon object has member functions available which can directly be used to select muon with "loose" or "tight" selection criteria. These are the corresponding lines in [MuonAnalyzer](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2012/PhysObjectExtractor/src/MuonAnalyzer.cc):

    ``` cpp
        muon_tightid.push_back(muon::isTightMuon(*itmuon, *vertices->begin()));
        muon_softid.push_back(muon::isSoftMuon(*itmuon, *vertices->begin()));
    ```

    These functions need the interaction vertex as an input (in addition to the muon properties) and this is provided through the first elemement of the vertex collection `vertices` which gives the best estimate of the interaction point. The vertex collection is accessed with:

    ``` cpp
      Handle<reco::VertexCollection> vertices;
      iEvent.getByLabel(InputTag("offlinePrimaryVertices"), vertices);
    ```

    Analysts must also consider how "isolated" they desire muons to be. For instance, an isolated muon might be produced in the decay of a W boson, while a non-isolated muon can come from a weak decay inside a jet or from a very high-momentum physics process that is likely to produce muons near a quark. Muon isolation is calculated from a combination of factors: energy from charged hadrons, energy from neutral hadrons, and energy from photons, all in a cone of radius dR < 0.3 or 0.4 around the muon. It is done as shown in this code snippet from [MuonAnalyzer](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2012/PhysObjectExtractor/src/MuonAnalyzer.cc):

    ``` cpp
        if (itmuon->isPFMuon() && itmuon->isPFIsolationValid()) {
          auto iso03 = itmuon->pfIsolationR03();
          muon_pfreliso03all.push_back((iso03.sumChargedHadronPt + iso03.sumNeutralHadronEt + iso03.sumPhotonEt)/itmuon->pt());
          auto iso04 = itmuon->pfIsolationR04();
          muon_pfreliso04all.push_back((iso04.sumChargedHadronPt + iso04.sumNeutralHadronEt + iso04.sumPhotonEt)/itmuon->pt());
        }
    ```

    More details on the muon identification can be found in the [CMS SWGuide MuonID page](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideMuonId). The full list of member function can be found in documentation of the [muon object class](https://cmsdoxygen.web.cern.ch/cmsdoxygen/CMSSW_5_3_30/doc/html/df/de3/classreco_1_1Muon.html).

=== "Run 2 Data (MiniAOD files)"

    As explained in the [Physics Object page](./objects#detector-information-for-identification), a mandatory task in the physics analysis is to identify muons, i.e. to separate “real” objects from “fakes”. The criteria depend on the type of analysis. The muon object has member functions available which can directly be used to select muon with "loose" or "tight" selection criteria. These are the corresponding lines in [MuonAnalyzer](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2015MiniAOD/PhysObjectExtractor/src/MuonAnalyzer.cc):

    ``` cpp
      muon_isSoft.push_back(mu.isSoftMuon(PV));
      muon_isTight.push_back(mu.isTightMuon(PV));
    ```

    These functions need the interaction vertex as an input (in addition to the muon properties) and this is provided through the first elemement of the vertex collection `vertices` which gives the best estimate of the interaction point. The vertex collection is accessed with:

    ``` cpp
      Handle<reco::VertexCollection> vertices;
      iEvent.getByToken(vtxToken_, vertices);
      const reco::Vertex &PV = vertices->front();
    ```

    with `vtxToken_` defined as a member of the `MuonAnalyzer` class and its value read from the [configuration file](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2015MiniAOD/PhysObjectExtractor/python/poet_cfg.py) in a similar way as for the muon collection.

    Analysts must also consider how "isolated" they desire muons to be. For instance, an isolated muon might be produced in the decay of a W boson, while a non-isolated muon can come from a weak decay inside a jet or from a very high-momentum physics process that is likely to produce muons near a quark. Muon isolation is calculated from a combination of factors: energy from charged hadrons, energy from neutral hadrons, and energy from photons, all in a cone of radius dR < 0.3 or 0.4 around the muon. It is done as shown in this code snippet from [MuonAnalyzer](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2015MiniAOD/PhysObjectExtractor/src/MuonAnalyzer.cc):

    ``` cpp
      auto iso03 = mu.pfIsolationR03();
      muon_pfreliso03all.push_back((iso03.sumChargedHadronPt + iso03.sumNeutralHadronEt + iso03.sumPhotonEt)/mu.pt());
      auto iso04 = mu.pfIsolationR04();
      muon_pfreliso04all.push_back((iso04.sumChargedHadronPt + iso04.sumNeutralHadronEt + iso04.sumPhotonEt)/mu.pt());
    ```

    More details on the muon identification can be found in the [CMS SWGuide MuonID page](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideMuonIdRun2). The full list of member function can be found in documentation of the [PAT muon object class](https://cmsdoxygen.web.cern.ch/cmsdoxygen/CMSSW_7_6_7/doc/html/d6/d13/classpat_1_1Muon.html).

=== "Run 2 Data (NanoAOD files)"

    As explained in the [Physics Object page](./objects#detector-information-for-identification), a mandatory task in the physics analysis is to identify muons, i.e. to separate “real” objects from “fakes”. The criteria depend on the type of analysis. Muon NanoAOD branches can be used to select muons with various quality levels. Analysts must also consider how "isolated" they desire muons to be. For instance, an isolated muon might be produced in the decay of a W boson, while a non-isolated muon can come from a weak decay inside a jet or from a very high-momentum physics process that is likely to produce muons near a quark. Muon isolation is calculated from a combination of factors: energy from charged hadrons, energy from neutral hadrons, and energy from photons, and the typical energy deposits from pileup interactions. The [2024 Open Data Workshop lesson on muons](https://cms-opendata-workshop.github.io/workshop2024-lesson-physics-objects/instructor/03-muons.html) details many of the relevant NanoAOD branches for muon selection. More details on the muon identification can be found in the [CMS SWGuide MuonID page](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideMuonIdRun2).

## Further muon corrections

There are misalignments in the CMS detector that make the reconstruction of muon momentum biased. The CMS reconstruction software does not fully correct these misalignments and additional corrections are needed to remove the bias. Correcting the misalignments is important when precision measurements are done using the muon momentum, because the bias in muon momentum will affect the results. However, if the measurement is not sensitive to the exact muon momentum, applying these corrections is not necessary.

### The Muon Momentum Scale Corrections

The Muon Momentum Scale Corrections, also known as the Rochester Corrections, are available in the [MuonCorrectionsTool](https://github.com/cms-legacydata-analyses/MuonCorrectionsTool).

=== "Run 1 Data"

    This tool was prepared for Run 1 data, and can be run using ROOT `TTree` files that have been produced from AOD files, for instance by using the Physics Object Extractor Tool. The correction parameters have been extracted in a two step method. In the first step, initial corrections are obtained in bins of the charge of the muon and the η and ϕ coordinates of the muon track. The reconstruction bias in muon momentum depends on these variables. In the second step, the corrections are fine tuned using the mass of the Z boson. The corrections for data and Monte Carlo (MC) are different since the MC events start with no biases but they can be induced during the reconstruction. Corrections have been extracted for both data and MC events.

    In the MuonCorrectionsTool, the Run1 Rochester Corrections are added to two datasets as an example: a 2012 dataset and a MC dataset. A plot is created to check that the corrections were applied correctly. Creating the plot requires selections and the produced dataset contains only a part of the initial dataset. These selections can be skipped when the plot is not needed and a corrected version of the whole dataset is wanted as a result. Below you can find instructions on how to run the example code, how to apply the corrections to a different dataset and how to apply the corrections when you don't want to create the plot/make the selections. The official code for the Rochester Corrections can be found in the `RochesterCorrections` directory. The example code for applying the corrections is in the `Test` directory.

    !!! Warning
    	The following example does not need the CMSSW environment but it requires ROOT. This code was written using the ROOT version 6.22.08. If you are using an older version, you might get errors running the code. In this case, try using `rochcor2012wasym_old.h` instead of `rochcor2012wasym.h`. You can do this by changing the first line of `rochcor2012wasym.cc` to `#include "rochcor2012wasym_old.h"`.

    **Applying the corrections to data and MC**: In the `Test` directory you can find `Analysis.C`, which is the example code for adding the corrections. The main function of `Analysis.C` is simply used for calling the `applyCorrections` function which takes as a parameter the name of the ROOT file (without `.root`), the path to the ROOT file, the name of the `TTree`, a boolean value of whether the file contains data (`true`) or MC (`false`), and a boolean variable of whether you want to correct the whole dataset (`true`) or make the selections needed for the plot (`false`).

    The `applyCorrections` function creates a `TTree` from the ROOT file contents. In an event loop, corrections are computed and the resulting corrected muon kinematic information is stored. The boolean variable `correctAll` is used here to determine whether to correct all muons in the dataset or to select events with muon pairs of opposite charges so that a Z boson mass peak plot can be created. The functions used to compute the Rochester correction, given a muon's four-vector information, can be found in `rochcor2012wasym.cc`.


    **Running the code**:

    1. Open an interactive ROOT session in your terminal:

    ``` cpp
    root
    ```

    2. Compile `muresolution.cc`, `rochcor2012wasym.cc` and `Analysis.C`

    ``` cpp
    .L RochesterCorrections/muresolution.cc++
    .L RochesterCorrections/rochcor2012wasym.cc++
    .L RochesterCorrections/Test/Analysis.C+
    ```

    3. Run the main function

    ``` cpp
    Analysis pf
    pf.main()
    ```

    4. To create the plot, compile `Plot.C` and run the main function

    ``` cpp
    .L RochesterCorrections/Test/Plot.C+
    main()
    ```

    **Applying the corrections to a different dataset**: You can use the example code to apply the corrections to different datasets. In `Test/Analysis.C`, edit the parameters of `applyCorrections` to pass the appropriate ROOT file and TTree information. Check whether the names and data types of the branches in your dataset match the existing content of `Analysis.C` -- if you are using a POET output file, or an AOD2NanoAODOutreachTool example file, this should be true! Adjust the branch names and data types if needed.

=== "Run 2 Data"

The MuonCorrectionsTool repository will be updated with new instructions and correction values for Run 2 data.
