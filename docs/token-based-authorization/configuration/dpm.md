# DPM

Currently supports only very [basic configuration described in documentation](https://twiki.cern.ch/twiki/bin/view/DPM/DpmSetupManualInstallation#Note_on_OpenID_Connect_and_WLCG)

:exclamation: [DPM EOL / end of support](https://indico.cern.ch/event/813745/contributions/3766117/note/) comes in summer 2024 (CentOS7/CentOS8 EOL) and it was decided not to fix issues with current WLCG JWT token implementation. If you would like to use WLCG JWT tokens with your production DPM storage than there is a quick, easy and transparent way how to [migrate to dCache](https://twiki.cern.ch/twiki/bin/view/DPM/DpmDCache) (several production storages already successfully used this method with less than 48 hours downtime).
