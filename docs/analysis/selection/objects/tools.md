# Common tools for physics objects

Many of the most important kinematic quantities defining a physics object are accessed in a common way across all the objects. All objects have associated energy-momentum vectors, typically constructed using **transverse momentum, pseudorapdity, azimuthal angle, and mass or energy**.

## 4-vector access functions

=== "Run 1 Data (AOD files)"

    In [MuonAnalyzer.cc](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2012/PhysObjectExtractor/src/MuonAnalyzer.cc), the muon four-vector elements are accessed as shown below. In this example, the values for each muon are stored into an array, which will become a branch in a ROOT TTree.

    ```cpp
    Handle<reco::MuonCollection> mymuons;
    iEvent.getByLabel(muonInput, mymuons);

    [...]

    for (reco::MuonCollection::const_iterator itmuon=mymuons->begin(); itmuon!=mymuons->end(); ++itmuon){
      if (itmuon->pt() > mu_min_pt) {

        muon_e.push_back(itmuon->energy());
        muon_pt.push_back(itmuon->pt());
        muon_eta.push_back(itmuon->eta());
        muon_phi.push_back(itmuon->phi());

        muon_px.push_back(itmuon->px());
        muon_py.push_back(itmuon->py());
        muon_pz.push_back(itmuon->pz());

        muon_mass.push_back(itmuon->mass());

    }
    ```

    The same type of kinematic member functions are used in all the different analyzers in the [src/ directory of the POET example code](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/tree/2012/PhysObjectExtractor/src). These and other basic kinematic methods are defined in the [LeafCandidate class](https://github.com/cms-sw/cmssw/blob/CMSSW_5_3_X/DataFormats/Candidate/interface/LeafCandidate.h) of the CMSSW DataFormats package (rendered for maybe easier readability in the [CMSSW software documentation](https://cmsdoxygen.web.cern.ch/cmsdoxygen/CMSSW_5_3_30/doc/html/dc/d78/classreco_1_1LeafCandidate.html)).

=== "Run 2 Data (MiniAOD files)"

    In [MuonAnalyzer.cc](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2015MiniAOD/PhysObjectExtractor/src/MuonAnalyzer.cc), the muon four-vector elements are accessed as shown below. In this example, the values for each muon are stored into an array, which will become a branch in a ROOT TTree.

    ```cpp
    Handle<pat::MuonCollection> muons;
    iEvent.getByToken(muonToken_, muons);

    [...]

    for (const pat::Muon &mu : *muons)
    {
      muon_e.push_back(mu.energy());
      muon_pt.push_back(mu.pt());
      muon_px.push_back(mu.px());
      muon_py.push_back(mu.py());
      muon_pz.push_back(mu.pz());
      muon_eta.push_back(mu.eta());
      muon_phi.push_back(mu.phi());

      [...]

    }
    ```

    The same type of kinematic member functions are used in all the different analyzers in the [src/ directory of the POET example code](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/tree/2015MiniAOD/PhysObjectExtractor/src). These and other basic kinematic methods are defined in the [LeafCandidate class](https://github.com/cms-sw/cmssw/blob/CMSSW_7_6_7/DataFormats/Candidate/interface/LeafCandidate.h) of the CMSSW DataFormats package (rendered for maybe easier readability in the [CMSSW software documentation](https://cmsdoxygen.web.cern.ch/cmsdoxygen/CMSSW_7_6_7/doc/html/dc/d78/classreco_1_1LeafCandidate.html)).

=== "Run 2 Data (NanoAOD files)"

    In NanoAOD, branches representing an object's four-vector are:

    - `OBJECT_pt`
    - `OBJECT_eta`
    - `OBJECT_phi`
    - `OBJECT_mass`

    Here, `OBJECT` can be `Muon`, `Electron`, `Jet`, etc.

## Track access functions

Many objects are also connected to tracks from the CMS tracking detectors. Information from
tracks provides other kinematic quantities that are common to multiple types of objects.

=== "Run 1 Data (AOD files)"

    From a muon object, we can access the associated track while looping over muons via the `globalTrack` method:

    ```cpp
    auto trk = mu->globalTrack(); // muon track
    ```

    Often, the most pertinent information about an object (such as a muon) to access from its
    associated track is its **impact parameter** with respect to the primary interaction vertex.
    Since muons can also be tracked through the muon detectors, we first check if the track is
    well-defined, and then access impact parameters in the xy-plane (`dxy` or `d0`) and along
    the beam axis (`dz`), as well as their respective uncertainties. They can be accessed as shown
    in this code snippet from [MuonAnalyzer](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2012/PhysObjectExtractor/src/MuonAnalyzer.cc):

    ``` cpp
        auto trk = itmuon->globalTrack();
        if (trk.isNonnull()) {
          muon_dxy.push_back(trk->dxy(pv));
          muon_dz.push_back(trk->dz(pv));
          muon_dxyErr.push_back(trk->d0Error());
          muon_dzErr.push_back(trk->dzError());
        }
    ```

    These functions need the position of the interaction vertex (`pv`) as an input and this is provided through the first elemement of the vertex collection `vertices` which gives the best estimate of the interaction point. The vertex collection is accessed with:

    ``` cpp
      Handle<reco::VertexCollection> vertices;
      iEvent.getByLabel(InputTag("offlinePrimaryVertices"), vertices);

      [...]

      math::XYZPoint pv(vertices->begin()->position());
    ```

    Note that the tracking method depends on the object, the electron tracks are found using the Gaussian-sum filter method `gsfTrack`:

    ``` cpp
    auto trk = it->gsfTrack(); // electron track
    ```

=== "Run 2 Data (MiniAOD files)"

    From a muon object, we can access the associated track while looping over muons via the `muonBestTrack` method.

    Often, the most pertinent information about an object (such as a muon) to access from its
    associated track is its **impact parameter** with respect to the primary interaction vertex.
    Since muons can also be tracked through the muon detectors, we first check if the track is
    well-defined, and then access impact parameters in the xy-plane (`dxy` or `d0`) and along
    the beam axis (`dz`), as well as their respective uncertainties. They can be accessed as shown
    in this code snippet from [MuonAnalyzer](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/2015MiniAOD/PhysObjectExtractor/src/MuonAnalyzer.cc):

    ``` cpp
      muon_dxy.push_back(mu.muonBestTrack()->dxy(PV.position()));
      muon_dz.push_back(mu.muonBestTrack()->dz(PV.position()));
      muon_dxyError.push_back(mu.muonBestTrack()->d0Error());
      muon_dzError.push_back(mu.muonBestTrack()->dzError());
    ```

    These functions need the position of the interaction vertex (`PV`) as an input and this is provided through the first elemement of the vertex collection `vertices` which gives the best estimate of the interaction point. The vertex collection is accessed with:

    ``` cpp
      Handle<reco::VertexCollection> vertices;
      iEvent.getByToken(vtxToken_, vertices);
      const reco::Vertex &PV = vertices->front();
    ```

    Note that the tracking method depends on the object, the electron tracks are found using the Gaussian-sum filter method `gsfTrack`.

=== "Run 2 Data (NanoAOD files)"

    In NanoAOD, the track object cannot be access directly. But often, the most pertinent information about an object (such as a muon) to access from its associated track is its **impact parameter** with respect to the primary interaction vertex. Since muons can also be tracked through the muon detectors, we first check if the track is well-defined, and then access impact parameters in the xy-plane (`dxy` or `d0`) and along the beam axis (`dz`), as well as their respective uncertainties. In NanoAOD files, branches representing track impact parameters are:

    - `OBJECT_dxy`
    - `OBJECT_dxyErr`
    - `OBJECT_dz`
    - `OBJECT_dzErr`
    - `OBJECT_ip3d`
    - `OBJECT_sip3d`

    Here, `OBJECT` can be `Muon`, `Electon`, `Tau`, etc, though only `Muon` and `Electron` collections have 3D impact parameter information. 

## Matching to generated particles

Simulated files also contain information about the generator-level particles that
were propagated into the showering and detector simulations. Physics objects can
be matched to these generated particles spatially.

=== "Run 1 Data (AOD files)"

    The [AOD2NanoAOD tool](https://github.com/cms-opendata-analyses/AOD2NanoAODOutreachTool/tree/2012) is an example code extracting objects from AOD file and storing them in an output file. In its [analyzer code](https://github.com/cms-opendata-analyses/AOD2NanoAODOutreachTool/blob/2012/src/AOD2NanoAOD.cc), it sets up several utility functions for matching: `findBestMatch`,
    `findBestVisibleMatch`, and `subtractInvisible`. The `findBestMatch` function takes
    generated particles (with an automated type `T`) and the 4-vector of a physics
    object. It uses angular separation to find the closest generated particle to the
    reconstructed particle:

    ``` cpp
    template <typename T>
    int findBestMatch(T& gens, reco::Candidate::LorentzVector& p4) {

      # initial definition of "closest" is really bad
      float minDeltaR = 999.0;
      int idx = -1;

      # loop over the generated particles
      for (auto g = gens.begin(); g != gens.end(); g++) {
        const auto tmp = deltaR(g->p4(), p4);

        # if it is closer, overwrite the definition of closest
        if (tmp < minDeltaR) {
          minDeltaR = tmp;
          idx = g - gens.begin();
        }
      }
      return idx; # return the index of the match
    }
    ```

    The other utility functions are similar, but correct for generated particles that
    decay to neutrinos, which would affect the "visible" 4-vector.

    In the AOD2NanoAOD tool, muons are matched only to "interesting" generated particles, which
    are all the leptons and photons (PDG ID 11, 13, 15, 22). Their generator status must be 1,
    indicating a final-state particle after any radiation chain.

    ``` cpp
    if (!isData){
      value_gen_n = 0;

      for (auto p = selectedMuons.begin(); p != selectedMuons.end(); p++) {

          // get the muon's 4-vector
          auto p4 = p->p4();

          // perform the matching with a utility function
          auto idx = findBestVisibleMatch(interestingGenParticles, p4);

          // if a match was found, save the generated particle's information
          if (idx != -1) {
            auto g = interestingGenParticles.begin() + idx;

      // another example of common 4-vector access functions!
            value_gen_pt[value_gen_n] = g->pt();
            value_gen_eta[value_gen_n] = g->eta();
            value_gen_phi[value_gen_n] = g->phi();
            value_gen_mass[value_gen_n] = g->mass();

      // gen particles also have ID and status from the generator
            value_gen_pdgid[value_gen_n] = g->pdgId();
            value_gen_status[value_gen_n] = g->status();

      // save the index of the matched gen particle
            value_mu_genpartidx[p - selectedMuons.begin()] = value_gen_n;
            value_gen_n++;
          }
      }
    }
    ```

=== "Run 2 Data (MiniAOD files)"

    In MiniAOD files, physics objects can be matched to generated particles following the same type of code example provided for Run 1 Data. In the configuration file for the `EDAnalyzer` that will perform matching, the relevant MiniAOD collection names should be provided, and the `EDAnalyzer` should use MiniAOD C++ types for the collections. But from that point the Run 1 example of performing spatial matching can also apply to MiniAOD. Examples of correct MiniAOD analyzer setup for generated particles and the reconstructed physics objects can be found in the [POET 2015 repository](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/tree/2015MiniAOD/PhysObjectExtractor/src).

=== "Run 2 Data (NanoAOD files)"

    In NanoAOD simulations, objects can be matched spatially to the contents of the `GenPart_*` branch collection that provides four-vector and particle ID information for all generated particles in the event. See the "Run 1 Data" tab for an example of performing spatial matching by comparing four-vectors. Some objects also have NanoAOD branches that provide the index of the matched generated particle in the `GenPart` arrays. For example, the `Muon` branch collection contains:

    - `Muon_genPartFlav`, a flag that shows how each muon was matched (to a prompt muon, a muon from a tau lepton, b quark, or c quark, etc.)
    - `Muon_genPartIdx`, an integer that locates each muon's matched generated particle within the `GenPart` arrays. A value of -1 would indicate that a muon was unmatched.

    All of the NanoAOD lepton collections have branches connecting the lepton to the generated particle list.
