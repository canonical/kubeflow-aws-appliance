# Steps to test the CloudFormation template
1. Login to your [aws console](https://aws.amazon.com/console/) with admin priviledges account
3. Set the region to `us-east-1` go to `CloudFormation service`
4. Choose the option to `create stack` (choose sub-option `with new resources`)
5. In the **Create stack** page
    1. Choose `Template is ready` and `Upload teamplate file` options
    2. Navigate to `cf_kubeflow_single_instance.yaml` and submit.
8. Next in **Specify stack details** page
    1. choose a name for your stack e.g `kubeflow`
    2. choose key-pair from dropdown to allow SSH (key must be created before template. You can create one in EC2 service > key pairs > create)
9. Next in **Configure stack options** you don't need to configure something
10. Make sure all your settings are correct in the last page and deploy the stack
11. Wait until the stack is in `CREATE_COMPLETE` state

To Understand other parameters of template please refer to [technical documentation](#)

Stack will create one EC2 instance with public vpc subnet and almost empty security group (per AWS appliance quidelines SSH access should be denied by default). To allow the ssh access to the instance go to `EC2` service click on the instance (choose the one with the tag `aws:cloudformation:stack-name: <name you speciffied>`) click on the `Security` tab click on the security group. In the `Inbounds rules` tab click on `Edit inbound rules` add new rule for SSH (you can type ssh in the dropdown and choose CIDR 0.0.0.0/0 for every IP, or you can be specific with your IP). 

SSH to the instance with provided keypair (make sure to proxy ports). You can get the ip in the `EC2` service instances tab, by clicking on desired instance

```
$ ssh -i <key-pair>.pem -D 9999 ubuntu@<ec2-ip> 
```

Now you need to wait until the kubeflow is installed (approx. 40 mins). You can always check the progress of the script with 
```
tail -f /var/log/cloud-init-output.log 
```

or you can check juju components at (following command will autorefresh every 5s).

```
juju status --watch 5s
```

After the juju components are in Active state, setup your browser Proxy (Firefox: Settings > Network  Settings > Socks proxy set to 127.0.0.1 and port 9999, or the one you tunnel). Now you can wisit `http://10.64.140.43.nip.io` and use the default credentials (user123@email.com/user123).

# Further example testing
If you want to make sure that all the components of the Kubeflow are ready, you can try to run [these examples](https://github.com/canonical/kubeflow-examples). Tu run the exaples you can Login to Dashboard -> go to notebooks tab -> create notebook server (with at least 8GB RAM) and then you can clone the repository directly to your notebook.

## Cost estimates
The only charged aws resource within the template is the ec2 instance `t2.2xlarge` with `gp2` volume of size `100GB`. You can find the cost caluculation at [this link](https://calculator.aws/#/estimate?id=2c3ee088d98101f4c1bb1e41cc86704e7a52ddd2)

## Architecture
You can find the archytectural diagram for the template in the file `template1-designer.png`.
