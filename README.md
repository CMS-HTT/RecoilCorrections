# RecoilCorrections

## Setting up the MET recoil correction interface

```
cd ${CMSSW_BASE}/src
cmsenv
git clone https://github.com/CMS-HTT/RecoilCorrections.git  HTT-utilities/RecoilCorrections 
scram b 
```

## Applying recoil correction in your analysis
Recoil corrections should be applied to the Higgs, DY and W+Jets MC samples. Do not apply r
ecoil corrections to the ttbar, single-top and diboson MC samples.

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
(fromHardProcessFinalState && (isMuon || isElectron)) || (isDirectHardProcessTauDecayProduct && !isNeutrino)

### Computation of hadronic jet multiplicity (njets) for recoil corrector

When computing hadronic jet multiplicity passed as one of the argument to the recoil corrector, count jets that fulfill the following criteria:
* pT > 30 GeV;
* |eta|<4.7;
* loose PF Jet ID;
* dR(lepton,jet) > 0.5.

Essential point. In selected W+Jets events one of the leptons is faked by hadronic jet and this jet should be counted as a part of hadronic recoil to the W boson. Therefore, when processing W+Jets MC sample, increase the number of jets, passed to the recoil corrector, by one.

## Applying recoil systematics
