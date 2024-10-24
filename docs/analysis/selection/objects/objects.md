# Physics Objects

## Description

The CMS is a giant detector that acts like a camera that "photographs" particle collisions, allowing us to interpret their nature.

Certainly we cannot directly observe all the particles created in the collisions because some of them decay very quickly or simply do not interact with our detector.  However, we can infer their presence.  If they decay to other stable particles and interact with the apparatus, they leave signals in the CMS subdetectors. These signals are used to *reconstruct* the decay products or infer their presence; we call these **physics objects**. These objects could be electrons, muons, jets, missing energy, etc., but also lower level objects like tracks.  For the current releases of open data, we store them in ROOT files following the EDM data model in AOD format.

In the CERN Open Portal site one can find a description of these physical objects and a list of them corresponding to [2010](http://opendata.cern.ch/docs/cms-physics-objects-2010) and [2011/2012](http://opendata.cern.ch/docs/cms-physics-objects-2011) releases of open data. For Run 1 data, this guide has the most up-to-date content with the corresponding code snippets. For Run 2 data from 2015, a detailed [listing](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookMiniAOD2015#High_level_physics_objects) is available in the CMS WorkBook.

The code examples to which this guide mainly refers to are:

=== "Run 1 Data (AOD files)"

    - [Physics Objects Extractor Tool (POET)](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/tree/2012): shows how to extract physics (objects) information and gives examples of methods or tools needed for processing them. For the sake of clarity, [EDAnalyzer modules](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/tree/2012/PhysObjectExtractor/src) are provided separately for each object. Separate branches exist for 2011 and 2012 data.
    - [AOD2NanoAODOutreachTool](https://github.com/cms-opendata-analyses/AOD2NanoAODOutreachTool/tree/2012): reads events from CMS AOD files and convert them to a reduced data format. This example provides a single [EdAnalyzer module](https://github.com/cms-opendata-analyses/AOD2NanoAODOutreachTool/blob/2012/src/AOD2NanoAOD.cc) handling all types of physics objects.

=== "Run 2 Data (MiniAOD files)"

    - [Physics Objects Extractor Tool (POET)](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/tree/2015MiniAOD): shows how to extract physics (objects) information and gives examples of methods or tools needed for processing them. For the sake of clarity, [EDAnalyzer modules](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/tree/2015MiniAOD/PhysObjectExtractor/src) are provided separately for each object.
    - [MiniAOD Getting Started Guide](https://opendata.cern.ch/docs/cms-getting-started-miniaod/): this guide on the Open Data Portal provides instructions for creating EDAnalyzers in the CMSSW version required to analyze 2016 data. A POET branch with examples tailored to 2016--2018 MiniAOD files is under development for the future.

=== "Run 2 Data (NanoAOD files)"

    - NanoAOD files do not require any CMSSW software to process, and therefore they have no Physics Object Extractor Tool equivalent!
    - [The 2024 Open Data Workshop lesson on physics objects](https://cms-opendata-workshop.github.io/workshop2024-lesson-physics-objects/instructor/index.html) provides an overview of NanoAOD contents related to the most common physics objects.

## DataFormats

=== "Run 1 Data (AOD files)"

    The physical objects are usually stored in specific *collections*.  For instance, [muons](http://opendata.cern.ch/docs/cms-physics-objects-2011#muons) are most commonly obtained from the `reco::Muon` collection.  The [AOD Data Format Table](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAodDataTable) gives a good description of the different collections (or data formats) for the AOD tier.  Unfortunately, the links for the *containers* column got broken after CMSSW was moved to Github.  Those links would have pointed us to the corresponding CMSSW C++ classes associated with those containers.  This is important because one needs to know which CMSSW class matches a given collection of objects to include the headers of those classes in the header of your analyzer code.  But let that not let us down.

    ![DataFormats](../../../images/collections.png)

    Fortunately, the names of the collections *containers* actually match the name of its associated CMSSW classes.  These classes (data format classes) live under the [DataFormats](https://github.com/cms-sw/cmssw/tree/master/DataFormats) directory in CMSSW.  If we browse through, we find the `MuonReco` package.   In its `interface` area we find the [DataFormats/MuonReco/interface/Muon.h](https://github.com/cms-sw/cmssw/blob/master/DataFormats/MuonReco/interface/Muon.h) class header, which is the one we would need to incorporate in our analyzer.  This is corroborated by this [Muon Analysis Twiki section](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookMuonAnalysis#Available_information).

    !!! Note "Remember"
        When accessing a specific piece of code in the CMSSW github repository, and want to explore its methods, variables, etc., make sure you select the right git branch.  E.g., [CMSSW_5_3_X](https://github.com/cms-sw/cmssw/blob/CMSSW_5_3_X/DataFormats/MuonReco/interface/Muon.h) for 2011/2012 open data.

    In addition to this base class, sometimes it is necessary to invoke other auxiliary classes.  For instance, [DataFormats/MuonReco/interface/MuonFwd.h](https://github.com/cms-sw/cmssw/blob/master/DataFormats/MuonReco/interface/MuonFwd.h), which can be found in the same interface area.

    So, in the context of this example, in order to support muons information, at the top of your [EDAnalyzer](../../../cmssw/cmsswanalyzers.md) you should include the following lines:

    ~~~ c++
    //classes to extract Muon information
    #include "DataFormats/MuonReco/interface/Muon.h"
    #include "DataFormats/MuonReco/interface/MuonFwd.h"
    ~~~

    See an example of an EDAnalyzer accessing muon information in the [MuonAnalyzer](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2012/PhysObjectExtractor/src/MuonAnalyzer.cc) of the Physics Object Extractor Tool (POET).

=== "Run 2 Data (MiniAOD files)"

    The physical objects are usually stored in specific *collections*.  For instance,  muons are obtained from the C++ class `std::vector<pat::Muon>` where `pat` stands for "Physics analysis tools" and is a namespace for miniAOD object classes. The collection is often called `pat::MuonCollection` using its definition in [the muon class](https://cmsdoxygen.web.cern.ch/cmsdoxygen/CMSSW_7_6_7/doc/html/dc/d3b/DataFormats_2PatCandidates_2interface_2Muon_8h_source.html)). The [MINIAOD physics objects table](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookMiniAOD2015#High_level_physics_objects) gives a good description of the different collections (or data formats) for the MINIAOD tier. This is the muon entry in that table:

    ![DataFormats](../../../images/miniaodtable1.png)

    The MINIAOD data format classes live under the [DataFormats/PatCandidates/interface](https://github.com/cms-sw/cmssw/tree/CMSSW_7_6_X/DataFormats/PatCandidates/interface) directory in CMSSW. Here we find the [DataFormats/PatCandidates/interface/Muon.h](https://github.com/cms-sw/cmssw/blob/CMSSW_7_6_X/PatCandidates/interface/Muon.h) class header, which is the one we would need to incorporate in our analyzer. The other possible collection

    So, in the context of this example, in order to support muons information, at the top of your [EDAnalyzer](../../../cmssw/cmsswanalyzers.md) you should include the following line:

    ~~~ c++
    //classes to extract Muon information
    #include "DataFormats/MuonReco/interface/Muon.h"
    ~~~

    See an example of an EDAnalyzer accessing muon information in the [MuonAnalyzer](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2015MiniAOD/PhysObjectExtractor/src/MuonAnalyzer.cc) of the Physics Object Extractor Tool (POET). The definitions of object classes used in miniAOD data format can be found in the [list of all classes](https://cmsdoxygen.web.cern.ch/cmsdoxygen/CMSSW_7_6_7/doc/html/d8/d06/namespacepat.html#nested-classes) of the `pat` namespace.

=== "Run 2 Data (NanoAOD files)"

    In NanoAOD, physics object *collections* are identified within the "Events" TTree by the first segment of their name. For example, all information taken from the `slimmedMuons` collection in MiniAOD shares `Muon_` at the beginning of the branch name: `Muon_pt`, `Muon_eta`, etc. Basic information about the contents of NanoAOD files can be found in the [NanoAOD WorkBook page](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookNanoAOD), and each Open Data record contains a link to the branch list for that dataset. For the 2016 dataset called ["JetHT"](https://opendata.cern.ch/record/30558), the variable list is found under the heading "Dataset semantics": [variable list](https://opendata.cern.ch/eos/opendata/cms/dataset-semantics/NanoAOD/30558/JetHT_doc.html). The variable list contains the C++ type of each branch.

## Access methods

In the [Event methods for data access](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideEDMGetDataFromEvent#Event_methods_for_data_access) section of the *Getting Data From an Event* Twiki page, one can find a complete description of the different methods available for *Event* data access in AOD or MiniAOD files.

!!! Note "Remember"
    When accessing the CMS twiki pages we will usually point you to the most recent page.  However, historical Twiki documentation, i.e., earlier revision of the pages, may provide more accurate information for open data that is already a few years old. One can access this historical archive by going to the bottom of any Twiki page, clicking on **History** and exploring the revisions closer to the open data release year.

=== "Run 1 Data (AOD files)"

    As indicated in that page, all Event data access methods use the `edm::Handle<T>`, where `T` is the C++ type of the requested object, to hold the result of an access.  As an example, during Run 1, the recommended method was the `getByLabel` one.  This method needed an `InputTag`.  This can also be extracted from the [AOD Data Format Table](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAodDataTable).  The first column indicate the InputTag:

    ![InputTags](../../../images/inputtags.png)

    Therefore, when accessing muon information, in the [`analyze` method](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2012/PhysObjectExtractor/src/MuonAnalyzer.cc#L155) of the EDAnalyzer would include the following lines:

    ~~~ c++
    Handle<reco::MuonCollection> mymuons;
    iEvent.getByLabel("muons", mymuons);
    ~~~

    If you required *cosmic* muons, you would need instead:

    ~~~ c++
    Handle<reco::MuonCollection> mymuons;
    iEvent.getByLabel("muonsFromCosmics", mymuons);
    ~~~

    Alternatively, as done in the POET MuonAnalyzer, it is also possible to retrieve the InputTag name from [configuration](../../../cmssw/cmsswconfigure.md).  In that case, the [configuration file](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2012/PhysObjectExtractor/python/poet_cfg.py#L112) has:

    ~~~ python
    process.demo = cms.EDAnalyzer('MuonAnalyzer',
                    InputCollection = cms.InputTag("muons")
    )
    ~~~

    In this case, the appropriate input tag needs to be defined in the EDAnalyzer class as done in the [MuonAnalyzer example class](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2012/PhysObjectExtractor/src/MuonAnalyzer.cc#L53):

    ~~~ c++
    //declare the input tag for MuonCollection
        edm::InputTag muonInput;
    ~~~

    It is extracted from the ParameterSet in the [constructor](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2012/PhysObjectExtractor/src/MuonAnalyzer.cc#L96)

    ~~~ c++
    MuonAnalyzer::MuonAnalyzer(const edm::ParameterSet& iConfig)
    {
    //now do what ever initialization is needed
    muonInput = iConfig.getParameter<edm::InputTag>("InputCollection");
    }
    ~~~

    and used in the [`analyze` method](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2012/PhysObjectExtractor/src/MuonAnalyzer.cc#L160):

    ~~~ c++
    Handle<reco::MuonCollection> mymuons;
    iEvent.getByLabel(muonInput, mymuons);
    ~~~

=== "Run 2 Data (MiniAOD files)"

    As indicated in that page, all Event data access methods use the `edm::Handle<T>`, where `T` is the C++ type of the requested object, to hold the result of an access.  For Run 2 data, the method is `getByToken`.  This method needs a "token" and an `InputTag`, which will pass the name of the collection to the analyzed. This name is indicated as "Label" [the MINIAOD table](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookMiniAOD2015#High_level_physics_objects), `slimmedMuons` for muons.

    The InputTag name is defined in the [configuration](../../../cmssw/cmsswconfigure.md). In that case, the [configuration file](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2015MiniAOD/PhysObjectExtractor/python/poet_cfg.py) has:

    ~~~ python
    process.mymuons = cms.EDAnalyzer('MuonAnalyzer', 
                    muons = cms.InputTag("slimmedMuons")
    )
    ~~~

    The appropriate token in the EDAnalyzer class as done in the [MuonAnalyzer example class](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2015MiniAOD/PhysObjectExtractor/src/MuonAnalyzer.cc):

    ~~~ c++
    //declare the token for MuonCollection
      edm::EDGetTokenT<pat::MuonCollection> muonToken_;
    ~~~

    which is then passed with the Input Tag to the constructor of the EDAnalyzer class

    ~~~ c++
    MuonAnalyzer::MuonAnalyzer(const edm::ParameterSet& iConfig): 
        muonToken_(consumes<pat::MuonCollection>(iConfig.getParameter<edm::InputTag>("muons")))
    {
    //now do what ever initialization is needed
    ~~~

    and used in the `analyze` method:

    ~~~ c++
    Handle<pat::MuonCollection> muons;
    iEvent.getByToken(muonToken_, muons);
    ~~~

=== "Run 2 data (NanoAOD files)"

    Data is stored in NanoAOD files as branches of a ROOT `TTree` object, and can be accessed by any method capable of reading ROOT trees. Some examples include:

    - Event loop (C++): the [tree1r](https://root.cern/doc/master/tree1_8C.html) function in this ROOT `TTree` tutorial shows how to read tree branches by looping over events.
    - RDataFrame: the `df001_introduction.C` tutorial shows how to read and process a tree using RDataFrame, in either [C++](https://root.cern/doc/master/df001__introduction_8C.html) or [Python](https://root.cern/doc/master/df001__introduction_8py.html)
    - Uproot/Awkward: the [2024 Open Data Workshop lesson](https://cms-opendata-workshop.github.io/workshop2024-lesson-cpp-root-python/instructor/06-uproot.html) on Open Data analysis in Python shows examples of how to interact with specific tree branches using the `uproot` and `awkward` Python libraries.

## Detector information for identification

The most signicant difference between a list of certain particles from a Monte Carlo generator and a list of the corresponding physics objects from CMS is likely the inherent uncertainty in the reconstruction. Selection of “a muon” or “an electron” for analysis requires algorithms designed to separate “real” objects from “fakes”. These are called **identification** algorithms, often abbreviated as ID.

**Isolation** algorithms are designed to measure the amount of energy deposited near the object, to determine if it was likely produced near the primary interaction (typically little nearby energy), or from the decay of a longer-lived particle (typically a lot of nearby energy). Many types of isolation algorithms exist to deal with unique physics cases.

Isolation is computed in similar ways for all physics objects: search for particles in a cone around the object of interest and sum up their energies, subtracting off the energy deposited by pileup particles. This sum divided by the object of interest’s transverse momentum is called relative isolation and is the most common way to determine whether an object was produced “promptly” in or following the proton-proton collision (ex: electrons from a Z boson decay, or photons from a Higgs boson decay). Relative isolation values will tend to be large for particles that emerged from weak decays of hadrons within jets, or other similar “nonprompt” processes.

Both types of algorithms function using working points that are described on a spectrum from “loose” to “tight”. Working points that are “looser” tend to have a high efficiency for accepting real objects, but perhaps a poor rejection rate for fake objects. Working points that are “tighter” tend to have lower efficiencies for accepting real objects, but much better rejection rates for fake objects. The choice of working point is highly analysis dependent. Some analyses value efficiency over background rejection, and some analyses are the opposite.

The standard identification and isolation algorithm results can be accessed from the physics object collections.

## Additional information

The next pages in this guide provide further details on how to access and identify CMS physics objects.

In addition, in Chapter 7 of the [CMS Workbook](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookIntroHiLevelReco) one can find *Analysis* pages that provide additional information, which can be useful to check on top of the general strategy for accessing objects in AOD or MiniAOD files that was discussed above.
