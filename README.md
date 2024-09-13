# RecoilCorrections

## Setting up the MET recoil correction interface

```
cd ${CMSSW_BASE}/src
cmsenv
git clone https://github.com/CMS-HTT/RecoilCorrections.git  HTT-utilities/RecoilCorrections 
scram b 
```

## Applying recoil correction in your analysis

Recoil corrections should be applied to the Higgs, DY and W+Jets MC samples. Do not apply recoil corrections to the ttbar, single-top and diboson MC samples.

```
// add the header file to your source file
#include "HTT-utilities/RecoilCorrections/interface/RecoilCorrector.h"

// Create instances of class RecoilCorrection and
// load recoil resolution functions before looping over events;
// The path to files is defined relative to ${CMSSW_BASE}/src directory

// use this RooT file when correcting Type I PF MET
// RecoilCorrector recoilPFMetCorrector("HTT-utilities/RecoilCorrections/data/TypeI-PFMet_Run2016BtoH.root");
// RecoilCorrector recoilPFMetCorrector("HTT-utilities/RecoilCorrections/data/TypeI-PFMET_2017.root");
RecoilCorrector recoilPFMetCorrector("HTT-utilities/RecoilCorrections/data/TypeI-PFMET_2018.root");


// apply recoil corrections on event-by-event basis
recoilPFMetCorrector.CorrectByMeanResolution(
    pfmet_ex, // uncorrected met px (float)
    pfmet_ey, // uncorrected met py (float)
    genPx, // generator Z/W/Higgs px (float)
    genPy, // generator Z/W/Higgs py (float)
    visPx, // generator visible Z/W/Higgs px (float)
    visPy, // generator visible Z/W/Higgs py (float)
    njets,  // number of jets (hadronic jet multiplicity) (int)
    pfmetcorr_ex, // corrected type I pf met px (float)
    pfmetcorr_ey  // corrected type I pf met py (float)
);
```

### Computation of generator Z/W/Higgs 4-momentum

Use generator particles and their status flags and pdgId to compute Z(W) 4-momentum at the generator level. Sum up 4-vectors of generator particles that satisfy the following criteria

```
(fromHardProcessFinalState && (isMuon || isElectron || isNeutrino)) || isDirectHardProcessTauDecayProduct 
```

### Computation of visible generator Z/W/Higgs 4-momentum

Sum up 4-vectors of generator particles that satisfy the following criteria

```
(fromHardProcessFinalState && (isMuon || isElectron)) || (isDirectHardProcessTauDecayProduct && !isNeutrino)
```


### Computation of hadronic jet multiplicity (njets) for recoil corrector

When computing hadronic jet multiplicity passed as one of the argument to the recoil corrector, count jets that fulfill the following criteria:
* pT > 30 GeV;
* |eta|<4.7;
* loose PF Jet ID;
* dR(lepton,jet) > 0.5.

Essential point. In selected W+Jets events one of the leptons is faked by hadronic jet and this jet should be counted as a part of hadronic recoil to the W boson. Therefore, when processing W+Jets MC sample, increase the number of jets, passed to the recoil corrector, by one.

## Applying recoil systematics

We consider two types of systematic uncertainties affecting MET:
* uncertainty in the response of hadronic recoil against leptonic system;
* uncertainty in the resolution of hadronic recoil against leptonic system.

A dedicated class [MEtSys](https://github.com/CMS-HTT/RecoilCorrections/blob/master/interface/MEtSys.h) has to be used to apply shifts to MET, reflecting systematic variations in the response and resolution of hadronic recoil.

```
#include "HTT-utilities/RecoilCorrections/interface/MEtSys.h"

// Create an instance of class MEtSys. The constructor loads RooT file with uncertainties. 
// The RooT file also contains histograms reflecting  the hadronic recoil response as 
// a function of the leptonic system pT for three types of processes : 
// 1) W+Jets / DY / Higgs 
// 2) dibosons and single-top (obsolete, not used any longer)
// 3) top pair events (obsolete, not used any longer)
// The path to the RooT file is defined relative to the folder
// $CMSSW_BASE/src 
// MEtSys metSys("HTT-utilities/RecoilCorrections/data/PFMEtSys_2016.root");
// MEtSys metSys("HTT-utilities/RecoilCorrections/data/PFMEtSys_2017.root"); 
MEtSys metSys("HTT-utilities/RecoilCorrections/data/PFMEtSys_2018.root");

// example below demonstrates how to apply upward systematic variation in the 
// RESPONSE of hadronic recoil to the W+Jets / DY / Higgs MC samples
metSys.ApplyMEtSys(
    met_x,met_y, // (float) mva met, use RECOIL CORRECTED value for the Higgs / DY / W+Jets MC
    lepPx,lepPy, // (float) transverse momentum of the full leptonic system
    visLepPx,visLepPy, // (float) transverse momentum of the visible leptonic system
    njets, // (int) number of jets : pT > 30 GeV, eta<4.7, loose PF JetID
    MEtSys::ProcessType::BOSON; // (int) type of process 
    MEtSys::SysType::Response, // (int) type of systematic uncertainty
    MEtSys::SysShift::Up, // (int) direction of systematic shift
    met_scaleUp_x,met_scaleUp_y // (float) shifted value of the met 
);

// for the systematic shifts in hadronic recoil RESOLUTION replace
// argument  MEtSys::SysType::Response by MEtSys::SysType::Resolution

// when applying downward variation use argument
// MEtSys::SysShift::Down 

```

Computation of the full momentum of the leptonic system and visible momentum of the leptonic system is done following the same procedure as for application of recoil corrections.

### Recoil correction systematics in the statistical inference

It is suggested to account for systematic uncertainties related to:  
* RESPONSE of hadronic recoil in Higgs / DY / W+Jets processes
* RESOLUTION of hadronic recoil in Higgs / DY / W+Jets processes

Eventually we intend to use 6 nuisance parameters. Each of the systematic sources listed above is split into three uncorrelated nuisances corresponding to three jet multiplicity bins:
* njets = 0
* njets = 1
* njets >= 2

We recommend to decorrelate these nuisance parameters across various data taking periods (2016, 2017 and 2018).

For other processes, such as ttbar, single-top and diboson production, apply standard MET uncertainties recommended by JME POG:

* JEC uncertainties split by various sources;
* unclustered energy scale uncertainty.

