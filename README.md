[![Build Status](https://travis-ci.com/jpata/hepaccelerate.svg?branch=master)](https://travis-ci.com/jpata/hepaccelerate-cms)
[![pipeline status](https://gitlab.cern.ch/jpata/hepaccelerate-cms/badges/master/pipeline.svg)](https://gitlab.cern.ch/jpata/hepaccelerate-cms/commits/master)

# hepaccelerate-cms

CMS-specific accelerated analysis code based on the [hepaccelerate](https://github.com/hepaccelerate/hepaccelerate) library.

Currently implemented analyses:
- `tests/hmm/analysis_hmumu.py`: CMS-HIG-19-006, [internal](http://cms.cern.ch/iCMS/analysisadmin/cadilines?line=HIG-19-006&tp=an&id=2254&ancode=HIG-19-006)

Variations of this code have been tested at:
- T2_US_Caltech (jpata, nlu)
- Caltech HPC (jpata)
- T2 Purdue
- T3 PSI

~~~
#Installation
pip3 install awkward uproot numba
git clone https://github.com/jpata/hepaccelerate-cms.git
cd hepaccelerate-cms
git submodule init
git submodule update

#Compile the C++ helper code (Rochester corrections and lepton sf, ROOT is needed)
cd tests/hmm/
make
cd ../..
~~~

Best results can be achieved if the CMS data is stored locally on a filesystem (few TB needed) and if you have a cache disk on the analysis machine of a few hundred GB.

# Contributing
If you use this code, we are happy to consider issues and merge improvements.
- Please make an issue on the Issues page for any bugs you find.
- To contribute changes, please use the 'Fork and Pull' model: https://reflectoring.io/github-fork-and-pull.
- For non-trivial pull requests, please ask at least one other person with push access to review the changes.

## Installation on Caltech T2 or GPU machine

On Caltech, an existing singularity image can be used to get the required python libraries.
~~~
git clone https://github.com/jpata/hepaccelerate-cms.git
cd hepaccelerate-cms
git submodule init
git submodule update

#Compile the C++ helpers
cd tests/hmm
singularity exec /storage/user/jpata/gpuservers/singularity/images/cupy.simg make -j4
cd ../..

#Run the code as a small test (small dataset by default, edit the file to change this)
#This should take approximately 5 minutes and processes 1 file from each dataset for each year
./tests/hmm/run.sh
~~~

## Running on full dataset using batch queue
We use the condor batch queue on Caltech T2 to run the analysis. It ~20 minutes for all 3 years using just the Total JEC & JER (2-3h using factorized JEC).

~~~
#Submit batch jobs after this step is successful
mkdir /storage/user/$USER/hmm
export SUBMIT_DIR=`pwd`
cd batch
./make_submit_jdl.sh
condor_submit submit.jdl

#submit merging and plotting
... (wait for completion)
condor_submit merge.jdl

cd ..

#when all was successful, delete partial results
rm -Rf /storage/user/$USER/hmm/out/partial_results
du -csh /storage/user/$USER/hmm/out
~~~


## Making plots, datacards and histograms
From the output results, one can make datacards and plots by executing this command:
~~~
./tests/hmm/plots.sh out
~~~
This creates a directory called `baseline` which has the datacards and plots. This can also be run on the batch using `merge.jdl`.


# Misc notes
Luminosity, details on how to set up on this [link](https://cms-service-lumi.web.cern.ch/cms-service-lumi/brilwsdoc.html).
~~~
export PATH=$HOME/.local/bin:/cvmfs/cms-bril.cern.ch/brilconda/bin:$PATH
brilcalc lumi -c /cvmfs/cms.cern.ch/SITECONF/local/JobConfig/site-local-config.xml \
    -b "STABLE BEAMS" --normtag=/cvmfs/cms-bril.cern.ch/cms-lumi-pog/Normtags/normtag_PHYSICS.json \
    -u /pb --byls --output-style csv -i /afs/cern.ch/cms/CAF/CMSCOMM/COMM_DQM/certification/Collisions16/13TeV/ReReco/Final/Cert_271036-284044_13TeV_23Sep2016ReReco_Collisions16_JSON.txt > lumi2016.csv

brilcalc lumi -c /cvmfs/cms.cern.ch/SITECONF/local/JobConfig/site-local-config.xml \
    -b "STABLE BEAMS" --normtag=/cvmfs/cms-bril.cern.ch/cms-lumi-pog/Normtags/normtag_PHYSICS.json \
    -u /pb --byls --output-style csv -i /afs/cern.ch/cms/CAF/CMSCOMM/COMM_DQM/certification/Collisions17/13TeV/ReReco/Cert_294927-306462_13TeV_EOY2017ReReco_Collisions17_JSON_v1.txt > lumi2017.csv

brilcalc lumi -c /cvmfs/cms.cern.ch/SITECONF/local/JobConfig/site-local-config.xml \
    -b "STABLE BEAMS" --normtag=/cvmfs/cms-bril.cern.ch/cms-lumi-pog/Normtags/normtag_PHYSICS.json \
    -u /pb --byls --output-style csv -i /afs/cern.ch/cms/CAF/CMSCOMM/COMM_DQM/certification/Collisions18/13TeV/ReReco/Cert_314472-325175_13TeV_17SeptEarlyReReco2018ABC_PromptEraD_Collisions18_JSON.txt > lumi2018.csv
~~~
