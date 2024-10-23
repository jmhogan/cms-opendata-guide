# Tag and Probe

The **Tag and Probe** method is an experimental procedure commonly used in particle physics to measure the efficiency of a given selection process directly from data. The procedure provides an unbiased sample of "probe" objects that can be then used to measure the efficiency of a particular selection criteria.

## Tag and Probe method

This method is a data-driven technique that relies on SM processes that produce a correlated pair of physics objects. It is primarily used in CMS to calculate the efficiencies of **lepton** selection criteria. The method is based on identifying two leptons:

- **Tag lepton**: a lepton that passes high-purity identification and isolation criteria, typically a "tight" working point for both identification and particle-flow isolation. In a tag-and-probe study, selected events typically pass a trigger related to the tag lepton.
- **Probe lepton**: a lepton that passes only the selection criteria desired as an unbiased baseline for the particular criterion you wish to study. For example, to measure the efficiency of the "global muon" criterion, the probe muons should pass only the "tracker muon" criterion; to measure the efficiency of the `HLT_Mu50` trigger path, the probe muons should pass whatever identification and isolation criteria are desired for a physics analysis.

### Resonances

The most common and robust tag-and-probe scenario uses leptons from $J/\psi$ meson or Z boson decays. Events are selected that contain opposite-sign, same-flavor lepton pairs with a dilepton invariant mass within a narrow window around the desired resonance. The $J/\psi$ resonance is appropriate for studying lower-momentum leptons, and the Z resonance is better suited for studying higher-momentum leptons. If one of the leptons in these events passes the desired **tag** criteria, then there is a very high probability that both leptons are genuine and emerged from a resonance decay. This gives confidence that the probe leptons represent a pool of genuine leptons that can be used to measure efficiencies of various criteria.

### Top quark pairs

Efficiencies of lepton selection criteria tend to show a dependence on the jet activity in the event. For analyses that include both high-momentum leptons and jets it may be difficult to find enough Z resonance events with jet properties similar to the final analysis. In this case, selecting top quark pair events in which both the top quark and antiquark decayed to a lepton can be helpful (though if your physics analysis is actually dileptonic top pairs, this method won't be appropriate!). Events are selected that contain opposite-sign, **opposite-flavor** lepton pairs as well as at least two moderate-momentum small-radius jets that are well separated from the leptons. The jets can be required to pass b-tagging criteria for increased purity. There are multiple SM processes that can produce this final state, but top quark pair production has by far the largest cross section at the LHC. If one of the leptons passes the desired **tag** criteria, then there is a high probability that both leptons emerged from a W boson in a top quark decay. This gives confidence that the probe leptons represent a pool of genuine leptons that can be used to measure efficiencies of various criteria for analyses with higher levels of jet activity.

## Lepton efficiencies and scale factors.

In general, the efficiency for a specific selection criterion will be given by the fraction of probe muons that pass the criterion, as shown in the equation below. Knowing the efficiency is useful for planning an analysis strategy.

(1) $\epsilon = \frac{\mathrm{N probes passing baseline selection + criterion under study}}(\mathrm{N probes passing baseline selection})

When simulation is used to model various physics processes in an analysis, it is important to compare selection efficiencies between data and simulation. The ratio of effiencies is called a "scale factor" and can be used to weight simulated events if efficiencies differ. The weight for simulation is calculated as:

(2) $w = \frac{\epsilon_{\mathrm{Data}}}{\epsilon_{\mathrm{simulation}}}$

When studying electrons and muons, CMS has developed a tradition of using the following efficiencies in analyses:

**Muons**:

 - $\epsilon_{\mathrm{identification}}$: baseline selection includes all general tracks and the identification criterion is tested. In Run 2 the baseline selection typically required all probes to be a ["tracker muon"](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideTrackerMuons).
 - $\epsilon_{\mathrm{isolation}}$: baseline selection includes all muons passing the desired identification criterion and the isolation criterion is tested.
 - $\epsilon_{\mathrm{trigger}}$: baseline selection includes all muons passing both the desired identification and isolation criteria and the trigger path criterion is tested.

**Electrons**:

 - $\epsilon_{\mathrm{reconstruction}}$: baseline selection includes electromagnetic calorimeter superclusters with $p_{T} > 10$ GeV and the [Gaussian Sum Filter electron reconstruction](https://twiki.cern.ch/twiki/bin/view/CMS/ElectronRecoPrinciples) is tested. 
 - $\epsilon_{\mathrm{identification}}$: baseline selection includes all reconstructed electrons and the identification criterion is tested.
 - $\epsilon_{\mathrm{isolation}}$: baseline selection includes all muons passing the desired identification criterion and the isolation criterion is tested.
 - $\epsilon_{\mathrm{trigger}}$: baseline selection includes all muons passing both the desired identification and isolation criteria and the trigger path criterion is tested.
 - Note: the electromagnetic calorimeter has a "gap" region where the barrel and endcap segments meet, $1.444 < |\eta| < 1.566$. This region is typically given its own bin in efficiency calculations to avoid biasing the measurement in regions with typical performance.

All of the efficiencies (and the resulting scale factors) in these chains are multiplied together to determine the overall efficiency or overall scale factor for simulation.

## Computing tag-and-probe efficiencies

The most basic calculation that can be performed is to literally construct the ratio shown in the equation above, typically in various bins of probe lepton momentum and pseudorapidity. This is most appropriate when a top quark tag-and-probe method has been used and the mass of the top quarks is not directly reconstructed. This type of calculation should be given larger uncertainties since the purity of the sample is not as clear. When using pure dilepton resonances for tag-and-probe, either sideband subtraction or fitting can be performed in the resonance mass distribution to calculate the efficiency using a high-purity sample of resonance decays.

### Sideband subtraction method

The sideband subtraction method involves choosing sideband and signal regions in invariant mass distribution for each tag-and-probe pair. The signal region is selected by finding the resonance position and defining a region around it. While the signal region contains both signal and background, the sideband region is chosen such as to have only background, with a gap between the sideband region and the signal region. A example of those region definitions can be seen below for the $J/\psi$ ressonance.

![Sideband regions](../../../images/analysis/selection/idefficiencystudy/signalextraction/InvariantMass_Tracker_region.svg)

For each event category (i.e. "passing probes" and "all probes"), and for a given variable of interest (e.g., the probe's transverse momentum), two distributions are obtained, one for each region (Signal and Sideband). In order to obtain the variable distribution for the signal only, we proceed by subtracting the Background distribution (Sideband region) from the Signal+Background one (Signal region):

(3) $N_{\mathrm{signal}} = N_{\mathrm{signal\,region}} - \alpha\times N_{\mathrm{sideband\,region}}$.

The normalization $\alpha$ factor compares the expected quantity of background present in the signal region to the sideband region:

(4) $\alpha = \frac{\mathrm{N\,background\,in\,signal\,region}}{\mathrm{N\,background\,in\,sideband\,region}}.

The number of background events in the signal region can be computed in various ways. For very flat background spectra like those in the figure above, $\alpha$ could be a ratio of the width, in GeV, of the signal region compared to the combined sideband regions. A fit could also be performed to only data in the sideband regions, and propagated across the peak region. The integral of the function in the signal region would then provide the number of background events for computing $\alpha$. The uncertainty in $\alpha$ is the quadrature sum of the statistical uncertainties on the number of background events in the signal and sideband regions.

After subtracting background events from the signal region, the efficiencies and scale factors are calculated by constructing the ratios in equations 1 and 2. 

### Fitting method

In this method, the signal is extracted not by histogram manipulation but by likelihood fitting. The procedure is applied after splitting the data in sub-samples, corresponding to bins of the kinematic variable of interest of the probe objects. As such, the efficiency will be measured as a function of that variable. Each sub-sample contains signal and background events; the signal is accessed by fitting the invariant mass spectra.

The fit for each bin can discriminate between signal and background, and will provide the number of signal events in the sample. This approach is illustrated below.

![Fitting method](../../../images/analysis/selection/idefficiencystudy/signalextraction/fitting_method_large.png)

Various functional forms can be used to fit the signal and background components. Signal is typically fitted with a Crystal Ball function, and background with a variety of polynomial or exponential shapes.

## Tutorials

Tag-and-probe studies can be performed using information from any CMS file format. For AOD or MiniAOD files, building an EDAnalyzer to extract the desired lepton information into a standard ROOT tree is the typical first step. Two resource are available:

 - CMSSW TagAngProbe module: this [EDAnalyzer package](https://github.com/cms-sw/cmssw/tree/CMSSW_5_3_X/PhysicsTools/TagAndProbe/test) has example configuration files in which tag and probe lepton criteria can be set and fits performed. The package has been maintained in the CMSSW versions used for 2015 and later Run 2 data.
 - The 2020 Open Data Workshop featured a [tag-and-probe lesson](https://cms-opendata-workshop.github.io/workshop-lesson-tagandprobe/) that demonstrates the resonance calulation techniques described here for muons in Run 1 data.
