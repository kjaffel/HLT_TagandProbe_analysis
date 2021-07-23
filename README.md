# ULegacy Tag and Probe HLT Analysis : 

Package of the EGamma and Muon groups to produce ULegacy Tag-and-Probe trees.
This repositry is meant only to produce dilepton and single trigger scale factors for full run2 Ulegcay compaign.
- [EgammaAnalysis-TnPTreeProducer ](https://github.com/kjaffel/EgammaAnalysis-TnPTreeProducer) WIP ...
    * [HLT Efficencies](https://github.com/kjaffel/egm_tnp_analysis)
- [MuonAnalysis-TagAndProbe](https://github.com/kjaffel/MuonAnalysis-TagAndProbe) WIP ... 

## EGamma HLT Scale Factor Measurements :
### To produce TnP ntuples :
``CMSSW_10_6`` can only be used for ultra-legacy samples, and ``CMSSW_10_2`` should be used for the rereco samples.
1. Install for ultra-legacy :
```bash
cmsrel CMSSW_10_6_13
cd CMSSW_10_6_13/src
cmsenv
git clone -b master git@github.com:kjaffel/EgammaAnalysis-TnPTreeProducer.git EgammaAnalysis/TnPTreeProducer
scram b -j8
```
2. Try-out:
You can find the cmsRun executable in ``EgammaAnalysis/TnPTreeProducer/python``:
```bash
cd EgammaAnalysis/TnPTreeProducer/
cmsenv
cmsRun python/TnPTreeProducer_cfg.py isMC=True doTrigger=True era=UL2018 maxEvents=5000
```
### To Submit Jobs using Crab: 
Check in ``EgammaAnalysis/TnPTreeProducer/crab`` the ``tnpCrabSubmit.py`` script.
1. submit: 
```bash
python tnpCrabSubmit.py
```
2. hadd: 
```bash
python tnpCrabJobsFinalStep.py crabDir
```
More options : 
- ``-s``: deactivate crab status.
- ``-r``: deactivate crab report.
- ``--hadd``: create output tree.
- ``--addAll``: hadd file in any status (default false: only finished).
- ``--dry_run``: do not hadd, just test.

### To Pass Event and PU Weight (for 106X):
Event weights are handled automatically by the package and you don't need anymore to use additional scripts. Few simple steps are needed instead to include data PU distribution before you start running the ntuplization step. Following the list to command to be issue:
1. Create Pileup Histograms based on the Golden JSON: Example for 2017 ULegacy:
```bash
pileupCalc.py -i /afs/cern.ch/cms/CAF/CMSCOMM/COMM_DQM/certification/Collisions17/13TeV/Legacy_2017/Cert_294927-306462_13TeV_UL2017_Collisions17_GoldenJSON.txt --inputLumiJSON /afs/cern.ch/cms/CAF/CMSCOMM/COMM_DQM/certification/Collisions17/13TeV/PileUp/UltraLegacy/pileup_latest.txt --calcMode true --minBiasXsec 69200 --maxPileupBin 90 --numPileupBins 90 ./2016UltraLegacyPUHist_nominal_99bins.root
```
2. Then do : 
```python
python Utilities/dumpPileup.py  PileupHistogram-goldenJSON-13tev-2017-69200ub-99bins.root
```
Copy the lines printed out by the last script [python/pileupConfiguration_cff.py](https://github.com/kjaffel/EgammaAnalysis-TnPTreeProducer/blob/master/python/pileupConfiguration_cff.py#L12) by adding a new line to the dictionary choosing a different key and then changing accordingly the call to the proper dictionary item.

Another option is to use the root files available for 2016, 2017 and 2018 Ulegacy collisions with Nominal / Up and Down variations (99 bins) so you don't have to run step1. of ``pileupCalc.py``.
```
/afs/cern.ch/cms/CAF/CMSCOMM/COMM_DQM/certification/Collisions1*/13TeV/PileUp/UltraLegacy/PileupHistogram-goldenJSON-13tev***/*.root
```
### To Make HLT Efficencies Maps:
1. Install stable branch 
```bash
cmsrel CMSSW_10_6_8
cd CMSSW_10_6_8/src
cmsenv
git clone -b master git@github.com:kjaffel/egm_tnp_analysis.git
cd egm_tnp_analysis
make 
```
**Note:** if you modify anything in histUtils.pyx then you need to run make cython-build before make in the previous instructions.

**Note:** This package does not have any CMSSW dependenies. However, we are using this package inside the CMSSW release just to ensure that its getting appropriate version of gcc, ROOT, RooFit, etc.

2. Steup ULegcay enviromenent:
```bash
source etc/scritps/setup.sh
```
3. Setting, Fitting and Plotting:

After setting up the ``settings.py``, do the following: 
```bash
python tnpEGM_fitter.py etc/config/HLTsettings/settings_eta_HLT.py --flag myWP --checkBins --doHLT
python tnpEGM_fitter.py etc/config/HLTsettings/settings_eta_HLT.py --flag myWP --createBins --doHLT
python tnpEGM_fitter.py etc/config/HLTsettings/settings_eta_HLT.py --flag myWP --createHists --doHLT
python tnpEGM_fitter.py etc/config/HLTsettings/settings_eta_HLT.py --flag myWP --doCutCount --onlyDoPlot --plotX eta --doHLT
```
Or simply use the bash script ``egm_tnp_analysis/runhlt_fitter.sh`` for all working points at differents steps :
```bash
bash runhlt_fitter
```
### Trouble shooting:
1. There was an update in scram to prevent executing arbitrary code in makefile fragments, [see details](https://indico.cern.ch/event/1058840/contributions/4450329/attachments/2281171/3875942/CMSSDT-CoreSW-210713.pdf). 

- If you face such an error: 
```bash
$ scram b -j8
Reading cached build data
Parse Error in src/EgammaAnalysis/TnPTreeProducer/plugins/BuildFile.xml, line 
Invalid value 'CMSSW_MAJOR_VERSION=$(echo ${CMSSW_VERSION} | cut -d'_' -f2)' found.
```
- Try-out: 
```xml 
- <Flags CPPDEFINES="CMSSW_MAJOR_VERSION=$(shell echo ${CMSSW_VERSION} | cut -d'_' -f2)"/>
- <Flags CPPDEFINES="CMSSW_MINOR_VERSION=$(shell echo ${CMSSW_VERSION} | cut -d'_' -f3)"/>
+ <Flags CPPDEFINES="CMSSW_MAJOR_VERSION=10"/>
+ <Flags CPPDEFINES="CMSSW_MINOR_VERSION=6"/>
```
## Muon HLT Scale Factor Measurements: WIP ... 
