# vmware-concourse-deploy

This is a project intended to document and automate the process required for a
complex multi-machine deployment via Concourse (For example, deploying PKS with
NSX-T) on vSphere.

## Get the code

***Do not clone this repository.***
Instead, [install Google Repo](https://source.android.com/source/downloading#installing-repo).

Here's a quick google repo install for the impatient.

```bash
# Validate python
python2.7 -c "print 'Python OK'" || echo 'Need python 2.7!'
python --version | grep "Python 2" || echo 'Warning: python 3 is default!'
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
# If you get a warning that about python 3, you might run this:
# After repo is installed:
sed -ri "1s:/usr/bin/env python:/usr/bin/python2.7:" ~/bin/repo
```

Once you've installed Google Repo, you will use it to download and assemble all the component git repositories.

This process is as follows:

``` bash
mkdir concourse-deploy-testing
cd concourse-deploy-testing
repo init -u git@gitlab.eng.vmware.com:vmworld2018/skyway.git

# you will need to enter gitlab credentials here
repo sync
```

After pulling down all the code as described above, go into `concourse-deploy-testing`
and you'll see there are several directories.  These are each a git repository.

We'll focus on the `concourse-deploy` repository.

## Bootstrapping

Go into `concourse-deploy/bootstrap`.
This directory contains code that will create a VM in vCenter, install Concourse, ansible, and other tools into that VM.

You can use an existing OVA captured after doing this process once, or you can go into the [bootstrap directory](bootstrap/)
and follow the readme there to create the VM directly in vCenter.

This should take about 15 minutes.

## Ssh into the jumpbox

Get the ip of the vm created in the bootstrap step above.
If you set up ssh keys, you can ssh right now, otherwise use:

* Username: `vmware`
* Password: `VMware1!`

On the jumpbox, there is also a copy of the source you used to bootstrap at `/home/vmware/deployroot`.

### Download VMware bits

Go into the jumpbox directory `/home/vmware/deployroot/concourse-deploy/downloads`, and follow the readme there to pull needed bits from http://my.vmware.com.
You can see an online version here: [downloads](downloads)

These files will be hosted via nginx after they download at `http://jumpbox-ip`.

**TODO** still need to pull down pivotal files

### Apply various pipelines

On the jumpbox, the pipelines exist at `/home/vmware/deployroot` and concourse is running on `http://jumpbox-ip:8080` with the same credentials as ssh to log in.
You can use fly from the jumpbox to apply the pipelines. To log in try `fly --target main login -c http://localhost:8080` and `fly pipelines --target main`

#### Install NSX-T

cd `/home/vmware/deployroot/nsx-t-gen` and follow the guide from [sparameswaran/nsx-t-gen](https://github.com/sparameswaran/nsx-t-gen).

Anther good guide is [from Sabha](http://allthingsmdw.blogspot.com/2018/05/introducing-nsx-t-gen-automating-nsx-t.html)

A sample config file is at `/home/vmware/deployroot/deploy-params/one-cloud-param.yaml` on the jumpbox, or [live here](https://github.com/NiranEC77/NSX-T-Concourse-Pipeline-Onecloud-param/blob/master/one-cloud-param.yaml).

There is also good coverage of the config file needed in Niran's guide from above starting in section 4.b.

Once you have the config file correct:

``` bash
cd /home/vmware/deployroot/nsx-t-gen
fly --target main login -c http://localhost:8080 -u vmware -p 'VMware1!'
fly -t main set-pipeline -p deploy-nsx -c pipelines/nsx-t-install.yml -l ../concourse-deploy/one-cloud-nsxt-param.yaml
fly -t main unpause-pipeline -p deploy-nsx
```

#### Install PAS and/or PKS

cd `/home/vmware/deployroot/nsx-t-ci-pipeline` and follow the guide from [sparameswaran/nsx-t-ci-pipeline](https://github.com/sparameswaran/nsx-t-ci-pipeline)

In particular, [this is the pipeline](https://github.com/sparameswaran/nsx-t-ci-pipeline/blob/master/pipelines/install-pks-pipeline.yml) and here is [a sample param file](https://github.com/sparameswaran/nsx-t-ci-pipeline/blob/master/pipelines/pks-params.sample.yml).

``` bash
cd /home/vmware/deployroot/nsx-t-ci-pipeline
fly --target main login -c http://localhost:8080 -u vmware -p 'VMware1!'
fly -t main set-pipeline -p deploy-pks -c pipelines/install-pks-pipeline.yml -l ../pks-deploy/pks-params.sample.yml
fly -t main unpause-pipeline -p deploy-pks
```


## Contributing

The vmware-concourse-deploy project team welcomes contributions from the community. Before you start working with vmware-concourse-deploy, please read our [Developer Certificate of Origin](https://cla.vmware.com/dco). All contributions to this repository must be signed as described on that page. Your signature certifies that you wrote the patch or have the right to pass it on as an open-source patch. For more detailed information, refer to [CONTRIBUTING.md](CONTRIBUTING.md).

### Development

For development, you will clone this repository and submit PRs back to upstream.
This is intended to be used as a sub project pulled together by a meta-project called [vmware-pks-deploy-meta](https://github.com/vmware/vmware-pks-deploy-meta).
You can get the full set of repositories by follow the prep section above.

## License

Copyright © 2018 VMware, Inc. All Rights Reserved.
SPDX-License-Identifier: MIT
