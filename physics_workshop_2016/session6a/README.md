goto test

Session 6a: Fitting SDMEs
============

You can just `source README` to run the example from beginning to end - but you should go step by step!

The goal of this session is to generate omega->3pi events with the angular distribution presented
in Schilling, et al., Nucl. Phy. B, 15 397 (1970) using the AmpTools framework. These input data
are then be processed through sim-recon to model experimental acceptance and reconstruction
efficiency. Finally, an acceptance corrected fit is preformed on the data using AmpTools to extract the SDMEs.

We have already done most of the heavy lifting. One million omega->3pi events have been generated,
simulated, smeared and reconstructed. The results at the end of this process can be found in the
three files <insert file names>.

As an example, let's generate just 100 events and take all the steps necessary to produce AmpTools input files.

First, we will use gen_omega_3pi to determine the thrown 4-vectors. You can run without options to
print the command line options to screen. Use the following command to generate 100 events, saving the
4-vectors to both hddm format for input to hdgeant, and AmpTools root format.

    gen_omega_3pi -c $SESSION6A/gen_omega_3pi.cfg -o AmpToolsFormatThrown.root -hd HDDMFormatThrown.hddm -n 100 -r 10000

Now, we want to simulate the passage of the thrown particles through the GlueX detector. The configuration
for this step is contained in the 'control.in' file in this directory and is loaded automatically. We want to
use the ccdb variation 'mc_sim1', so we will set our environment, then launch hdgeant.

    setenv JANA_CALIB_CONTEXT "variation=mc_sim1"
    hdgeant

Next we need to smear the output of hdgeant to simulate the detector response.

    mcsmear hdgeant.hddm -PTHREAD_TIMEOUT_FIRST_EVENT=300 -PTHREAD_TIMEOUT=300

Now we can reconstruct the smeared events, and produce some generic histograms to look at results of reconstruction.

    hd_root hdgeant_smeared.hddm -PPLUGINS=danarest,monitoring_hists -PTHREAD_TIMEOUT_FIRST_EVENT=300 -PTHREAD_TIMEOUT=300

There are two output files at the end of this step: 'hd_root.root' and 'danarest.hddm'. You can look at the root
file to view various histograms related to the quality of the resonstruction. The file monitoring_hists_1Mevents.root
contains the result of this step run over the full sample that will be used in the final fit using AmpTools. To
browse the root file:

    root -l monitoring_hists_1Mevents.root
    [0] new TBrowser

The next step is to skim events from the hddm file that are consistent with the

Now that you have completed the exercise, you can always cheat if you want to do it again:

    source README

test:

# Make some utility scripts
    echo '{ \n TBrowser g; \n}' > ! tbr.C
    echo '{ \n omega_skim_Tree->Process(gSystem->ExpandPathName("$SESSION3B/DSelector_p3pi_workshop.C+"));\n}' > ! RunDSelector.C

# Run the code
    gen_omega_3pi -c $SESSION6A/gen_omega_3pi.cfg -o AmpToolsFormatThrown.root -hd HDDMFormatThrown.hddm -n 20 -r 10000
    setenv JANA_CALIB_CONTEXT "variation=mc_sim1"
    hdgeant
    mcsmear hdgeant.hddm -PTHREAD_TIMEOUT_FIRST_EVENT=300 -PTHREAD_TIMEOUT=300
    hd_root hdgeant_smeared.hddm -PPLUGINS=danarest,monitoring_hists -PTHREAD_TIMEOUT_FIRST_EVENT=300 -PTHREAD_TIMEOUT=300
    hd_root -PPLUGINS=omega_skim dana_rest.hddm
    root -l -q tree_omega.root ${ROOT_ANALYSIS_HOME}/scripts/Load_DSelector.C RunDSelector.C

# Open a browser
    root -l hd_root.root gen_omega_3pi_diagnostic.root hist_p3pi_omega.root tbr.C

# Remove the scripts
    rm tbr.C
    rm RunDSelector.C

exit