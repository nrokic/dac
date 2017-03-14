## Setup Details

**DNS Records:**

	Team 1:
	`dac-lb-0`: `ucp.team1.dac.dckr.org`
	`dac-lb-1`: `dtr.team1.dac.dckr.org`
	`dac-lb-2`: `*.app.team1.dac.dckr.org` 
	Team 2:
	`dac-lb-0`: `ucp.team2.dac.dckr.org`
	`dac-lb-1`: `dtr.team2.dac.dckr.org`
	`dac-lb-2`: `*.app.team2.dac.dckr.org` 
	Team 3:
	`dac-lb-0`: `ucp.team3.dac.dckr.org`
	`dac-lb-1`: `dtr.team3.dac.dckr.org`
	`dac-lb-2`: `*.app.team3.dac.dckr.org` 
	Team 4:
	`dac-lb-0`: `ucp.team4.dac.dckr.org`
	`dac-lb-1`: `dtr.team4.dac.dckr.org`
	`dac-lb-2`: `*.app.team4.dac.dckr.org` 
	Team 5:
	`dac-lb-0`: `ucp.team5.dac.dckr.org`
	`dac-lb-1`: `dtr.team5.dac.dckr.org`
	`dac-lb-2`: `*.app.team5.dac.dckr.org` 


# Lab 1 ( 90 mins)

**Goal:** Create a Swarm Cluster configured with recommended logging and devicemapper configuration. 

Documentation Reference: https://docs.docker.com/engine/swarm/

   *  Configure recommended `devicemapper` on the manager nodes 
   	* Documentations [here](https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/#configure-direct-lvm-mode-for-production)
    * This is the best practices for all centos/rhel based systems. For sake of time you're required to do the manager nodes only.
* Verify engine config and restart engine
* Set up 3-node swarm with 3-managers and 2 workers
* Set up engine logging 
	*  Documentation [here](https://github.com/nicolaka/elk-dee)
* Finally, confirm that you have logging working, full swarm setup with device-mapper configured, nodes are labeled, 


# Lab 2 ( 90 mins)

 **Goal:**  Install and configure highly available UCP
  
**Documentation Reference:** https://docs.docker.com/datacenter/ucp/2.1/guides/admin/install/

* Install UCP version `2.1.0` on top of existing swarm cluster
  * UCP needs to listen on non-standard port 12390 (using `--controller-port 12390` )
  * Make sure you add the correct SANs. One SAN should be the  FQDN for your ucp (e.g `--san ucp.team1.dac.dckr.org`) and the other SAN should be the public IP of the node.
  * Make sure you can access UCP using the public IP of the node after installation is complete.
  * Install your license.
* Deploy NGINX on UCP LB node (`dac-lb-0`)
	* NGINX needs to listen on ports 80/443 and forward to non-standard port of UCP. Please use port *12390* as the backend port. 
	* Reconfigure `ucp-nginx.conf` config file available in this directory with your setup's info by substituting `MANAGER_IP` in the config file. The launch the nginx container using `docker run -d -p 80:80 -p 443:443 --restart=unless-stopped --name ucp-lb -v ${PWD}/ucp-nginx.conf:/etc/nginx/nginx.conf:ro nginx:stable-alpine`
* Install external certs using the provided certs.
* Set up HRM on standard ports 80 and 443.
* Set up node labels
      - [documentation](https://docs.docker.com/engine/swarm/manage-nodes/#change-node-availability)
      - Label the two worker nodes as `staging` and `production`
      - Label the two worker nodes with their availability zones. ( you can find an instance availability zone using `curl http://169.254.169.254/latest/meta-data/placement/availability-zone`) 
* Perform a UCP backup
  * [Documentation](https://docs.docker.com/datacenter/ucp/2.1/guides/admin/backups-and-disaster-recovery/#backup-command)
* Upgrade UCP to version `2.1.1`
* Finally, ensure that you have a fully functioning UCP cluster composed of three manager nodes and two worker nodes. 

# Lab 3 ( 90 mins)

**Goal:** Install and configure highly available DTR

* Deploy NGINX on LB node for DTR (`dac-lb-1`)
	* NGINX needs to listen on ports 80/443 and forward to non-standard port of UCP. Please use port *12391* for HTTP and *12392* for  HTTPS.
	*  Reconfigure `dtr-nginx.conf` config file available in this directory with your setup's info by substituting `MANAGER_IP` in the config file. The launch the nginx container using `docker run -d -p 80:80 -p 443:443 --restart=unless-stopped --name dtr-lb -v ${PWD}/dtr-nginx.conf:/etc/nginx/nginx.conf:ro nginx:stable-alpine`
* Install DTR Version `2.2.2`
	* Install three DTR replicas on the Manager nodes
	* Make sure to use correct external url. This should be your DTR's FQDN. 
	* Make sure to use non-standard ports *12391* for HTTP and *12392* for  HTTPS for **each** replica using the `--replica-http-port` and `--replica-https-port` flags.
* Configure External Certs.
* Configure Storage Backend.
	* Minio, an S3 Compatible Object storage, is configured already on the `dac-services` node. Please use the following configuration to set it up.
	* Go to $DAC_SERVICES_PUBLIC_IP>:9000
	* Create a Bucket (lower right corner) and call it `dtr`
	* Go back to DTR URL, and go to Settings > Storage.
	* Select S3 and provide the required parameters. Make sure to use your own `dac-services` private IP as the **Region Endpoint** Here's a sample configuration:
	![](images/minio.png)
	* Confirm your configuration worked:
	* Create your first repository under the `admin` namespace.
	* Perform a `docker login` from your local Docker client.
	* Perform a docker push to confirm that storage backend is configured. 
* Backup DTR
	* [Documentation](https://docs.docker.com/datacenter/dtr/2.2/guides/admin/backups-and-disaster-recovery/)
* Upgrade DTR to 2.2.3

* Finally, ensure that you have a fully functioning DTR cluster composed of three replicas, configured with proper external certs, object storage.



# Lab 4

**Goal:** Application Deployment Operations

* Deploy pets app




