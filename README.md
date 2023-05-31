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

In this process, Docker configures metrics collection and visualization, the test automation script sends metrics to Pushgateway,
Prometheus* collects the metrics, and then Grafana presents it. If you are using a different environment, then you can set up a 
similar process after Pushgateway. If you use another system to collect metrics, then modify runrapid.py for your environment. 
The steps below assume the following:  •  The user has privileges to run docker commands. •  Commands are run on the node that 
will run Pushgateway, with example IP address: PUSHGWIP  •  The working directory is: $WORK_DIR  1.  Perform the following one-time
setup procedure:
cd $WORK_DIR mkdir scripts && cd scripts cat > run_once << EOF #!/bin/bash -v 

docker run --name grafana -d --network host -e "GF_SECURITY_ADMIN_PASSWORD=password" grafana/grafana 
EOF 
cat > start_all << EOF 
#!/bin/bash -v docker run --name pushgateway -d --network host prom/pushgateway docker run --name prometheus -d --network host -v 
$WORK_DIR/scripts/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus docker start grafana EOF cat > stop_all << EOF #!/bin/bash -v 
docker stop grafana docker kill prometheus pushgateway docker rm prometheus pushgateway EOF cat > prometheus.yml << EOF global:   scrape_interval: 2s   
evaluation_interval: 15s scrape_configs:   - job_name: 'pushgateway'     static_configs:     - targets: ['$PUSHGWIP:9091'] EOF chmod 755 run_once start_all stop_all 2. 
Ensure that the file is properly formatted including spaces as per listing above, using the command: cat prometheus.yml 3.  Complete the one-time setup with the script: ./run_once 4.
After the one-time setup is completed, use the following script for later test runs: ./start_all 5.  Verify that Grafana, Prometheus, and Pushgateway are running with the command: 
docker ps | grep -e grafana -e prometheus -e pushgateway 6.  Using a browser, go to the Grafana page http://$PUSHGWIP:3000 , log in with admin/password, and change the password. 
Add the Prometheus data source with URL http://$PUSHGWIP:9090 where: a.  Replace $PUSHGWIP with your IP address. b.  Enter the desired Scrape interval, for example 15s. c. 
Press the Save & Test button which should result in the message “Data source is working”. After these steps are complete, Pushgateway is ready to receive metrics from PROX tests.
From there, Prometheus will collect them and Grafana will visualize them as graphs. If you want to protect Pushgateway, then modify the Python script runrapid.py accordingly. 
Later as needed, you can do cleanup on old metrics with the command: ./stop_all && ./start_all The cleanup step above will not delete the Grafana configuration.

