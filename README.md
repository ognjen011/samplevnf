# samplevnf
Mirror of the OPNFV Sample Virtual Network Function Project

INSTALL

sudo bash git clone https://github.com/opnfv/samplevnf.git 

cd samplevnf/VNFs/DPPD-PROX/helper-scripts/rapid 

./dockerimage.sh build

######################

./dockerimage.sh push curl http://localhost:5000/v2/_catalog | grep prox_slim

Create pods 

./createrapidk8s.py

This controller script will connect to the running PROX pods, run tests defined in the basicrapid.test file (or another .test file if it is specified), 
report results in the text screen, and send them into Pushgateway. The following example shows that using the default test setup (including how many cores are used for PROX functionality), 
we observed ~13.3Mpps (of 64B size) sustained without exceeding the defined packet drop rate or latency percentile. Note: 
If runrapid.py doesn’t start the PROX engines with the message
“Creating mempool…” in your environment, then try increasing the huge pages memory allocation from 512Mi to 1Gi in pod-rapid.yaml

